> 项目中使用了Elasticsearch作为主要业务库，今天特此研究记录一下Elasticsearch评分推荐功能



```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
    <version>4.0.9.RELEASE</version>
</dependency>
```



### 查询上下文和过滤上下文

首先理解查询上下文和过滤上下文的概念

> 查询上下文：回答查询子句与文档的匹配度如何，会计算评分，根据评分进行排序
>
> 过滤上下文：回答查询字句与文档是否匹配，不会计算评分

```java
//过滤上下文
nativeSearchQueryBuilder.withFilter(boolQueryBuilder).build();
//查询上下文
nativeSearchQueryBuilder.withQuery(boolQueryBuilder).build();
```

只要查询子句中使用filter参数或者bool查询中使用了filter参数，过滤上下文就会生效

**Elasticsearch会缓存频繁使用的过滤上下文，提供性能**

## 相关性评分

> 使用查询上下文进行检索时，默认根据评分进行排序，如果使用自定义排序字段，则返回结果中score为null

相关性评分会在查询结果的score中体现，是一个单精度浮点数，Float类型，查询结果根据score从大到小排序

创建索引时，score默认值为1

### 自定义文档相关性

* 第一种：创建索引时设置字段的相关性

  ```sql
  PUT test_index
  {
    "mappings": {
      "properties": {
        "title":{
          "type": "text",
          "boost": 2
        }
      }
    }
  }
  ```

  **弊端**：修改boost值得唯一方式是重建索引，reindex数据

* 第二种：**查询时修改文档的相关性**

## 自定义评分

在实际应用场景中，对于默认评分排名不能满足要求，可以在检索时自定义评分影响排名

* 对某个子句查询结果自定义评分

```java
//java
MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("title",title);
matchQueryBuilder.boost(666); //自定义评分
boolQueryBuilder.must(matchQueryBuilder);
boolQueryBuilder.boost(3); //自定义评分

{
  "bool" : {
    "must" : [
      {
        "match" : {
          "title" : {
            "query" : "南",
            "operator" : "OR",
            "prefix_length" : 0,
            "max_expansions" : 50,
            "fuzzy_transpositions" : true,
            "lenient" : false,
            "zero_terms_query" : "NONE",
            "auto_generate_synonyms_phrase_query" : true,
            "boost" : 666.0
          }
        }
      }
    ],
    "adjust_pure_negative" : true,
    "boost" : 3.0
  }
}
```

* 修改negative_boost自定义相关性

```java
//java
QueryBuilder queryBuilder = QueryBuilders.termsQuery("_id","BFfQrnsBiQBPQEO76YPG");
BoostingQueryBuilder boostingQueryBuilder = QueryBuilders.boostingQuery(boolQueryBuilder,queryBuilder);
boostingQueryBuilder.negativeBoost(10f);
	return nativeSearchQueryBuilder.withQuery(boostingQueryBuilder).build();

{
  "boosting" : {
    "positive" : {
      "bool" : {
        "must" : [
          {
            "term" : {
              "_id" : {
                "value" : "BFfQrnsBiQBPQEO76YPG",
                "boost" : 1.0
              }
            }
          },
          {
            "match" : {
              "title" : {
                "query" : "南",
                "operator" : "OR",
                "prefix_length" : 0,
                "max_expansions" : 50,
                "fuzzy_transpositions" : true,
                "lenient" : false,
                "zero_terms_query" : "NONE",
                "auto_generate_synonyms_phrase_query" : true,
                "boost" : 1.0
              }
            }
          }
        ],
        "adjust_pure_negative" : true,
        "boost" : 1.0
      }
    },
    "negative" : {
      "terms" : {
        "_id" : [
          "BFfQrnsBiQBPQEO76YPG"
        ],
        "boost" : 1.0
      }
    },
    "negative_boost" : 10.0, //仅对negative部分生效
    "boost" : 1.0
  }
}
```

> negative_boost仅对negative部分生效

* 对评分进行过滤

  ```java
  nativeSearchQueryBuilder.withMinScore(3L); //设置最小评分为3，小于3的不展示
  ```


## 复杂搜索语句自定义评分

> 在实际业务场景中仅仅对索引中的某个字段进行文档相关性排序很难满足要求，需要对多个字段进行综合评估以得出文档相关性评分

### function_score

> 可以使用function_score在搜索结束之后，对每一个匹配的文档进行重新计算评分，根据新的评分在排序

**坑！！！function_score函数计算评分时，部分函数不支持text类型的字段**

```java
//语法
{
  "query": {
    "function_score": {
      "query": {
        全局匹配条件
      },
      "functions": [
        {
          "filter": {
            函数生效的匹配条件
          },
          "function": 函数
        }
      ],
      "boost": 1, 
      "min_score": 1,
      "max_boost": 10,
      "boost_mode": "multiply", //必填项，否则会报错 如何影响最终评分
      "score_mode": "multiply"
    }
  }
}
```

| 函数               | 说明                     | 是否支持text类型 |
| :----------------- | ------------------------ | ---------------- |
| weight             | 对查询出来的文档设置权重 | 支持text类型     |
| field_value_factor | 对指定字段计算函数得分   | 不支持text类型   |
| Random Score       | 生成[0,1)之间的函数分数  | 不支持text类型   |
|                    |                          |                  |

> 待研究：衰减函数，自定义脚本

### boost_mode && score_mode

> score_mode：当存在多个函数时，计算最终函数分数的方式
>
> boost_mode ：查询得分与函数的得分的综合得分方式

```java
FunctionScoreQueryBuilder functionScoreQueryBuilder = new FunctionScoreQueryBuilder(matchQueryBuilder,fieldValueFactorFunctionBuilder)
                        .boostMode(CombineFunction.SUM)
                        .scoreMode(FunctionScoreQuery.ScoreMode.SUM);
```

* multiply

  > 多个函数分数相乘，默认

* SUM

  > 多个函数分数相加

* avg

  > 取平均值

* first

  > 取匹配条件的第一个函数的分数

* max

  > 取最大值

* min

  > 取最小值

### max_boost

> 最大的函数分数, 函数分数不能超过max_boost

### min_score

> 最小综合分数，最终综合分数小于min_score的文档将会被过滤掉

### boost

> 设置函数的基础分数

#### weight

> 设置权重

```java
WeightBuilder weightBuilder = ScoreFunctionBuilders.weightFactorFunction(12);
MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("title", key);
FunctionScoreQueryBuilder functionScoreQueryBuilder = QueryBuilders.functionScoreQuery(matchQueryBuilder,weightBuilder);
boolQueryBuilder.must(functionScoreQueryBuilder);
```

```java
{
  "query": {
    "bool": {
      "must": [
        {
          "function_score": {
            "query": {
              "match": {
                "title": {
                  "query": "南",
                  "operator": "OR",
                  "prefix_length": 0,
                  "max_expansions": 50,
                  "fuzzy_transpositions": true,
                  "lenient": false,
                  "zero_terms_query": "NONE",
                  "auto_generate_synonyms_phrase_query": true,
                  "boost": 1
                }
              }
            },
            "functions": [
              {
                "filter": {
                  "match_all": {
                    "boost": 1
                  }
                },
                "weight": 12
              }
            ],
            "score_mode": "multiply",
            "max_boost": 3.4028235e+38,
            "boost": 1
          }
        }
      ],
      "adjust_pure_negative": true,
      "boost": 1
    }
  }
}
```

> 上面query查询出来的文档都会乘以权重

### random

```java
//java
TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("status", status);
RandomScoreFunctionBuilder randomScoreFunctionBuilder = new RandomScoreFunctionBuilder().setField("status").seed(100);
FunctionScoreQueryBuilder functionScoreQueryBuilder = new FunctionScoreQueryBuilder(termQueryBuilder,randomScoreFunctionBuilder)
                        .boostMode(CombineFunction.SUM)
                        .scoreMode(FunctionScoreQuery.ScoreMode.SUM);
boolQueryBuilder.must(functionScoreQueryBuilder);


{
  "bool" : {
    "must" : [
      {
        "function_score" : {
          "query" : {
            "term" : {
              "status" : {
                "value" : "offline",
                "boost" : 1.0
              }
            }
          },
          "functions" : [
            {
              "filter" : {
                "match_all" : {
                  "boost" : 1.0
                }
              },
              "random_score" : {
                "seed" : 100,
                "field" : "status"
              }
            }
          ],
          "score_mode" : "sum",
          "boost_mode" : "sum",
          "max_boost" : 3.4028235E38,
          "boost" : 1.0
        }
      }
    ],
    "adjust_pure_negative" : true,
    "boost" : 1.0
  }
}
```

## rescore(再研究)

## 踩坑点

1.  部分函数不支持text类型的字段

   > **报错**："reason":"Fielddata is disabled on text fields by default. Set fielddata=true on [title] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.

## 总结

> 1. ES提供了非常强大的评分推荐功能，上面实际操作了一些函数，没有太多难度，但是需要结合具体业务场景，灵活使用
>
> 2. 重在研究理解其实现原理



参考：

[1] https://blog.csdn.net/laoyang360/article/details/104809787

