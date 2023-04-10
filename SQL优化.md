## 插入数据

### 一、insert优化

1. 批量插入

   ```mysql
   insert into ** values (),()...
   ```

2. 手动提交事务

   ```mysql
   start transaction
   ...
   commit
   ```

3. 主键顺序插入

4. 大批量插入数据

   如果一次性需要擦汗如大批量数据，使用insert语句插入性能较低，可以使用load指令插入。

   ```mysql
   #客户端连接服务端时，加上参数 --local-infile
   mysql --local-infile -u root -p
   #设置全局参数local infile为1，开启从本地加载文件导入数据的开关
   set global local infile = 1;
   #执行load指令将准备好的数据，加载到表结构中
   load data local infile '/root/sql1.log' into table "tb_user' fields terminated by ',' lines terminated by '\n';
   ```

### 二、主键优化

1. 尽量降低主键的长度
2. 插入数据时尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键
3. 尽量不要使用UUID做主键或者其他自然主键，如身份证，因为它们是无顺序的
4. 业务操作时，避免对主键的修改

### 三、order by 优化

```
using index: 直接通过索引返回数据，性能高

using filesort: 需要将返回的结果在排序缓冲区排序
```

优化：

1. 根据排序字段建立合适的索引，多字段排序时，遵循最左前缀法则

2. 尽量使用覆盖索引，可以防止进行**回表查询**，从而提高效率

3. 多字段排序时，一个升序一个降序

   ```mysql
   #创建索引
   create index idx_user_age_phone_ad on tb_user(age asc, phone desc);
   #根据age,phone进行一个升序，一个降序
   explain select id,age,phone from tb_user order by age asc,phone desc;
   ```

   

4. 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小sort_buffer_size(默认256k)。

### 四、group by 优化

1. 在分组操作时，可以通过索引来提高效率

   ```mysql
   #创建索引
   create index idx_user_pro_age_sta on tb_user(profession,age,status);
   #执行分组操作，根据profession字段分组
   explain select profession,count(*) from tb_user group by profession,age; 
   ```

   

2. 分组操作时，索引的使用也是满足最左前缀法则的

### 五、limit优化

一般分页查询时，通过创建覆盖索引能够比较好的提高性能，可以通过覆盖索引加子查询形式进行优化

```mysql
explain select * from tb_sku t, (select id from tb_sku order by id limit 2000000,10) a where t.id = a.id;
```

### 六、count优化

count用法的效率顺序：count(字段) < count(主键) < count(1) ≈ count(*)，所以尽量使用count(*)

```
1. count(主键)
2. count(字段)
3. count(1)
4. count(*)：InnoDB引擎并不会把全部字段取出来，而是专门做了优化，不去之，服务层直接按行进行累加，效率最高
```

### 七、update优化

InnoDB的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，否则会从行锁升级为表锁

-> 在update时要对有索引的列进行更新，否则会从行锁变为表锁，导致并发性能降低

-> 所以尽量根据主键/索引字段进行数据更新

## 视图

