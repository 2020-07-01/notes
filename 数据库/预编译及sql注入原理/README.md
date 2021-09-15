## 预编译





服务端预编译

PREPARE ins from 'insert into t select ? , ?'

set @a=999,@b='hello';

execute ins using @a,@b;

select * from t





客户端预编译

















参考：

https://www.cnblogs.com/micrari/p/7112781.html

