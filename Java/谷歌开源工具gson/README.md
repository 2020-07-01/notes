> 工作中经常遇到json解析，对象转换等场景，通过阅读官方文档及自己工作中遇到的坑，特此记录。

[gson](https://github.com/google/gson)

gson是谷歌开源的一个java对象和JSON之间相互转换的java类库，最常用的两个方法**toJson()**和**fromJson()**，目前最版本为2.8.8

## 基本概念

* 序列化和反序列化

  > 序列化：java对象转换为JSON字符串的过程
  >
  > 反序列化：JSON字符串转换为java对象的过程

* **gson 处理对象的几个重要点**

  > 1.gson序列化时，输出中会省略空字段
  >
  > 2.**反序列化时，json中缺失的字段会导致对象中相应的字段为其默认值**
  >
  > 3.对于合成的字段，序列化和反序列化时将会被忽略

## 基本用法

### 注解

* @SerializedName

  > ```
  > 指定该字段在json中对应的名称
  > @SerializedName("n")
  > private String name;
  > ```

* @Expose

  > ```tsx
  > 指定该字段是否支持序列化和反序列化
  > @Expose(serialize = false, deserialize = false)
  > private String name;
  > ```

### 空值参与序列化

> gson 默认空值不参与序列化

```java
GsonBuilder gsonBuilder = new GsonBuilder();
Gson gson=  gsonBuilder.serializeNulls().create();
```

### 字符串null 转 ""

> gson默认情况下空值会序列化为null

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.serializeNulls();
gsonBuilder.registerTypeAdapter(String.class,new StringNullAdapter());
Gson gson=  gsonBuilder.create();

static class StringNullAdapter extends TypeAdapter<String> {
    @Override
    public void write(JsonWriter out, String value) throws IOException {
        if(value == null){
            out.value("");
            return;
        }
        out.jsonValue(value);
    }
    @Override
    public String read(JsonReader in) throws IOException {
        if(in.peek() == JsonToken.NULL){
            in.nextNull();
            return "";
        }
        return in.nextString();
    }
}
```

### 日期null转 ""

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setDateFormat("yyyy-MM-dd HH:mm:ss");//日期格式设置
Gson gson=  gsonBuilder.serializeNulls().create();

static class DateNullAdapter extends TypeAdapter<Date> {
    @Override
    public void write(JsonWriter out, Date value) throws IOException {
        if(value == null){
            out.value("");
        }else {
            //自定义日期处理
            String dateFormatAsString = DateFormat.getDateTimeInstance().format(value);
            out.value(dateFormatAsString);
        }
    }
    @Override
    public Date read(JsonReader in) {
        return null;
    }
} 
```

### 注册TypeAdapterFactory

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapterFactory(new NullToEmptyAdapterFactory());

class NullToEmptyAdapterFactory<T> implements TypeAdapterFactory{
    @Override
    public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {
        Class<T> rawType = (Class<T>) type.getRawType();
        if(rawType == String.class){
            return (TypeAdapter<T>) new StringNullAdapter();
        }
        return null;
    }
}
```

## 泛型类型的支持

>gson 饭序列化过程中，如果遇到泛型类型，则默认转换为LinkedTreeMap对象

```java
String json = "{\"demo\":\"this is demo generic name\",\"data\":{\"demoName\":\"this is demo nam\",\"date\":null}}";
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.serializeNulls();
Gson gson = gsonBuilder.create();
//重点 获取类型
Type clazz = new TypeToken<DemoGeneric<Demo>>(){}.getType();
DemoGeneric<Demo> demoDemoGeneric = gson.fromJson(json,clazz);
```

**此处涉及到Java高级类型接口Type**

