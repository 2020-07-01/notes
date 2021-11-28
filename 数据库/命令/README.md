> 将自己平时用到的，已经理解的一些参数进行整理，具体以官方文档为准，尤其在生产环境中执行时以官方文档为准

- **[官方文档-参数](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_buffer_pool_chunk_size)**



* InnoDB页大小

  > ```sql
  > -- 页大小，默认为16KB
  > show VARIABLES like 'innodb_page_size';
  > ```

* 缓冲池大小

  > ```sql
  > -- 默认是128M，单位字节
  > show VARIABLES like 'innodb_buffer_pool_size'
  > ```

* 缓冲池实例

  > 缓冲池可配置多个实例
  >
  > ```sql
  > show VARIABLES like 'innodb_buffer_pool_instances'
  > ```

* 缓冲池块大小

  > ```sql
  > -- 缓冲池块的大小，单位字节
  > show VARIABLES like 'innodb_buffer_pool_chunk_size';
  > ```

* 缓冲池老年代大小

  > ```sql
  > -- 老年代占整个LRU链长度的比例，默认是37
  > show VARIABLES like 'innodb_old_blocks_pct'
  > ```

* 缓冲池老年代停留时间窗口

  > ```sql
  > -- 默认1000，单位毫秒
  > show VARIABLES like 'innodb_old_blocks_time'
  > ```

* 脏页清理线程的数量

  > ```sql
  > show VARIABLES like 'innodb_page_cleaners';
  > ```

* 监控缓冲池

  > ```sql
  > show engine Innodb satus/;
  > ```
  
* 查看磁盘页大小

  > ```sql
  > SHOW GLOBAL STATUS like '%Innodb_page_size%';
  > ```

* 锁定和解锁表

  > ```sql
  > -- 加读锁
  > lock tables tableName read;
  > -- 加写锁
  > lock tables tableName write;
  > -- 解锁
  > unlock tables;
  > ```

* 锁定行

  > ```sql
  > selecr ***  for update;
  > ```

* 查看存储引擎

  > ```sql
  > show engines;
  > ```

  



