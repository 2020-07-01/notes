> MySQL执行语句，当存在order by 排序时，Extra中可能会出现using filesort 和 using temporary



# using filesort

MYSQL会使用sort_buffer_size大小的内存进行排序，当数据大小超过sort_buffer_size时，会将结果数据转移到文件中排序（**多路归并排序**）

sort_buffer_size是一个connection级别的参数，指定连接允许分配的排序内存大小

**using filesort仅限于单表查询中出现**



# using temporary

MYSQL会使用临时表来保存临时临时结构，

**using temporary出现在多表查询排序**



# order by + limit 优化处理

MYSQL对order by + limit 的filesort做了优化处理，使用priority queue来保存top n的数据



参考文章：

[MySQL · 答疑释惑· using filesort VS using temporary](http://mysql.taobao.org/monthly/2015/03/04/)

