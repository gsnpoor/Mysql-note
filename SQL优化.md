## 插入数据

### insert优化

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

### 主键优化

1. 尽量降低主键的长度
2. 插入数据时尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键
3. 尽量不要使用UUID做主键或者其他自然主键，如身份证，因为它们是无顺序的
4. 业务操作时，避免对主键的修改

### order by 优化

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

### group by 优化

1. 在分组操作时，可以通过索引来提高效率

   ```mysql
   #创建索引
   create index idx_user_pro_age_sta on tb_user(profession,age,status);
   #执行分组操作，根据profession字段分组
   explain select profession,count(*) from tb_user group by profession,age; 
   ```

   

2. 分组操作时，索引的使用也是满足最左前缀法则的