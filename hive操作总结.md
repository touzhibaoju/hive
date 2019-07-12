## 一、数据类型

```
Hive的内置数据类型可以分为两大类
    1. 基础数据类型包括：TINYINT、SMALLINT、INT、BIGINT、BOOLEAN、FLOAT、DOUBLE、STRING、BINARY、TIMESTAMP、DECIMAL、CHAR、VARCHAR、DATE。 
    下面的表格列出这些基础类型所占的字节以及从什么版本开始支持这些类型。
    2. 复杂类型包括ARRAY、MAP、STRUCT、UNION，这些复杂类型是由基础类型组成的。

  数据类型     	所占字节                                    	开始支持版本         
  TINYINT  	1byte： -128 ~ 127                       	               
  SMALLINT 	2byte：-32,768 ~ 32,767                  	               
  INT      	4byte：-2,147,483,648 ~ 2,147,483,647    	               
  BIGINT   	8byte：-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807	               
  BOOLEAN  	                                        	               
  FLOAT    	4byte单精度                                	               
  DOUBLE   	8byte双精度                                	               
  STRING   	                                        	               
  BINARY   	                                        	从Hive0.8.0开始支持 
  TIMESTAMP	                                        	从Hive0.8.0开始支持 
  DECIMAL  	                                        	从Hive0.11.0开始支持
  CHAR     	                                        	从Hive0.13.0开始支持
  VARCHAR  	                                        	从Hive0.12.0开始支持
  DATE     	                                        	从Hive0.12.0开始支持

Hive表结构中的数据类型与MySQL对应列有如下关系
    MySQL(bigint) --> Hive(bigint) 
    MySQL(tinyint) --> Hive(tinyint) 
    MySQL(int) --> Hive(int) 
    MySQL(double) --> Hive(double) 
    MySQL(bit) --> Hive(boolean) 
    MySQL(varchar) --> Hive(string) 
    MySQL(decimal) --> Hive(decimal) 
    MySQL(date/timestamp) --> Hive(date)
    MySQL(timestamp) --> Hive(timestamp)
    注：
    Mysql[decimal(12,2)]-->Hive[decimal]会导致导入后小数丢失
```

## 数据类型转化关系表

![hive数据类型转化关系表](58E079893E7B45579C8D35BEED2C1B92)

## 二、DDL操作

```
1. 存储格式：
压缩表和非压缩表
- textfile是Hive建表时默认使用的存储格式，数据不做压缩，本质上textfile就是以文本的形式将数据存放在hdfs中。
- orcfile 列式存储格式，查询更快。

2. 表类型：
2.1 分区表和非分区表，分桶表
一个表可以拥有一个或者多个分区，每个分区以文件夹的形式单独存在表文件夹的目录下
2.2 外部表和内部表
- 管理表 默认创建的表都是管理表，也被称为内部表，删除管理表时，Hive 也会删除这个表中数据
- 外部表 删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉

3. hive -f/-e操作
hive -f 后面指定的是一个文件，然后文件里面直接写sql
hive -e 后面是直接用双引号拼接hivesql，然后就可以执行命令
```

1. ### 建库

```sql
1. 创建库
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
　　[COMMENT database_comment]　　　　　　//关于数据块的描述
　　[LOCATION hdfs_path]　　　　　　　　　　//指定数据库在HDFS上的存储位置
　　[WITH DBPROPERTIES (property_name=property_value, ...)];　　　　//指定数据块属性

2. 查看有哪些数据库
dfs -ls /hive/warehouse/;
show databases;

3. 显示数据库的详细属性信息
desc database [extended] dbname;

4. 查看正在使用哪个库
select current_database();

5. 查看创建库的详细语句
show create database t3;

6. 删库（默认情况下，hive 不允许删除包含表的数据库， 使用 cascade 关键字）
drop database if exists dbname;
drop database if exists dbname cascade;

7. 切换库
use database_name;
```

2. ### 建表

```sql
1. 创建表
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
　　[(col_name data_type [COMMENT col_comment], ...)]
　　[COMMENT table_comment]
　　[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
　　[CLUSTERED BY (col_name, col_name, ...)
　　　　[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
　　[ROW FORMAT row_format]
　　[STORED AS file_format]
　　[LOCATION hdfs_path]；

•CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXIST 选项来忽略这个异常
•EXTERNAL 关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION）
•LIKE 允许用户复制现有的表结构，但是不复制数据
•COMMENT可以为表与字段增加描述
•PARTITIONED BY 指定分区
•ROW FORMAT 
　　DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char] 
　　　　MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char] 
　　　　| SERDE serde_name [WITH SERDEPROPERTIES 
　　　　(property_name=property_value, property_name=property_value, ...)] 
　　用户在建表的时候可以自定义 SerDe 或者使用自带的 SerDe。如果没有指定 ROW FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe。在建表的时候，
用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的 SerDe，Hive 通过 SerDe 确定表的具体的列的数据。 
•STORED AS 
　　SEQUENCEFILE //序列化文件
　　| TEXTFILE //普通的文本文件格式
　　| RCFILE　　//行列存储相结合的文件
　　| INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname //自定义文件格式
　　如果文件数据是纯文本，可以使用 STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCE 。
•LOCATION指定表在HDFS的存储路径

2. 创建内部表
create table student(
    id int,
    name string
) row format delimited fields terminated by "\001"
stored as textfile;

3. 创建外部表
create external table student_ext(
    id int, 
    name string
) row format delimited fields terminated by "," 
location "/hive/student";

4. 创建分区外部表
如果不指定数据库，hive会把表创建在default数据库下。分区表现为这张表的数据存储目录下的一个子目录，如果是分区表。那么数据文件一定要存储在某个分区中，而不能直接存储在表中。
create external table tmp.table_add_column_test(
  a string comment '原始数据1',
  b string comment '原始数据2'
) comment '表注释'
partitioned by (dt string)
row format delimited fields terminated by '\001'
stored as textfile;

5. 创建分桶外部表
create external table student_bck(
    id int, 
    name string
) clustered by (id) 
sorted by (id asc, name desc) into 4 buckets
row format delimited fields terminated by "\001"
location "/hive/student_bck";

6. 使用CTAS创建表（从一个查询SQL的结果来创建一个表进行存储）
create table student_ctas as select * from student where id < 95012;

7. 复制表结构（加external关键字，复制成外部表）
create external table student_copy like student;

8. 查看当前使用的数据库中有哪些表
dfs -ls /hive/warehouse/edw.db;
show tables;

9. 查看非当前使用的数据库中有哪些表
show tables in myhive;

10. 查看数据库中以xxx开头的表
show tables like 'student_c*';

11. 查看表的信息
desc student;

12. 查看表的详细信息（格式不友好）
desc extended student;

13. 查看表的详细信息（格式友好）
desc formatted student;

14. 查看分区信息
show partitions student_ptn;

15.查看表的详细建表语句
show create table student_ptn;
```

3. ### 修改表

```sql
1. 内部表和外部表转换
内部转外部：alter table tableA set TBLPROPERTIES('EXTERNAL'='true');
外部转内部：alter table tableA set TBLPROPERTIES('EXTERNAL'='false');

2. 修改表名（需先把表改为内部表然后在改名，外部表改名后相应路径不变，会导致数据问题）
alter table student rename to new_student;

3. 修改字段类型（注意hive字段类型转换规则，范围小的数据类型可以转换为范围更大的类型，但不能逆向转，比如 int可以转换为string,但string无法转为int）
alter table new_student change name new_name string;

4. 增加字段（添加列默认在列的末尾添加新列，但在分区列之前。默认模式为RESTRICT（即不修改元数据），cascade则同步修改元数据，这样才会重新刷新数据时添加的字段才会有值，不然刷新数据新添加的字段以前的数据都为null）
alter table tmp.tablename add columns (
    last_lesson_teacher_no string comment '老师编号',
    teaching_department string comment '教学部',
    assistant_job_number string comment '班主任工号'
) cascade;

5. 对字段排序（hive可以对已存在的表字段进行排序，但HDFS文件内容并没有变化,内部也不行）
alter table tmp.table_add_column_test change column added_column added_column string after a cascade;

6. 删除一个字段
不支持

7. 替换所有字段
alter table new_student replace columns (id int, name string, address string);

8. 增加分区（未分区表增加分区时，只能删除重建表结构，然后增加分区，把数据移到分区目录下，记得把表转为外部表；分区表增加分区时，直接增加分区即可）
增加一个分区：
alter table ods_hfjydb.view_user_uuid add partition(dt='20190101');
增加多个分区：
alter table student_ptn add partition(city="chongqing2") partition(city="chongqing3") partition(city="chongqing4");

9. 修改分区（修改已经指定好的分区的数据存储目录，此时原先的分区文件夹仍存在，但是在往分区添加数据时，只会添加到新的分区目录）
alter table student_ptn partition (city='beijing') set location '/student_ptn_beijing';

10. 删除分区（如果是外表,需要删除hdfs目录数据）
alter table ods.sc_career_window drop partition(dt='20190101');
```

4. ### 删除表

```sql
1. 删除表(如果是外表,需要删除hdfs目录数据 dfs -rm -r/hive/warehouse/ods.db/tablename)
drop table new_student;

2. 清空表
truncate table student_ptn;
insert overwrite table t_table1 select * from t_table1 where 1=0;

3. 删除符合条件的数据（查询出数据重新插入表或者新表，分区表也可以）
insert overwrite table t_table1 select * from t_table1 where XXXX;
```

5. ### 修复表

```sql
1. 删除表结构后在重建表，可以对表进行数据修复(如果是未分区表转分区表需先把数据转到分区中再修复)
msck repair table ods.sc_career_window;
```

## 三、DML操作

1. ### Load

```
load data [local] inpath 'filepath' [overwrite] into table tablename [partition (partcol1=val1, partcol2=val2 ...)]
load data inpath '/p/view_user_uuid/*' into table table partition (dt='2019-04-28');
注意：
1. load 命令会将 filepath 中的文件复制到目标文件系统中。目标文件系统由表的位置属性决定。被复制的数据文件移动到表的数据对应的位置。
2. 如果没有指定 LOCAL 关键字，filepath 指向的是一个完整的 URI，hive会直接使用这个 URI。
3. 如果使用了 overwrite 关键字，则目标表（或者分区）中的内容会被删除，然后再将 filepath 指向的文件/目录中的内容添加到表/分区中。
4. 如果目标表（分区）已经有一个文件，并且文件名和 filepath 中的文件名冲突，那么现有的文件会被新文件所替代。
```

2. ### Insert

```sql
Hive 中 insert 主要是结合 select 查询语句使用，将查询结果插入到表中。
注意：
1. 需要保证查询结果列的数目和需要插入数据表格的列数目一致.
2. 如果查询出来的数据类型和插入表格对应的列数据类型不一致，将会进行转换，但是不能保证转换一定成功，转换失败的数据将会为 NULL。
3. 可以将一个表查询出来的数据插入到原表中, 结果相当于自我复制了一份数据。

1. 覆盖(当select为空时，不覆盖以前数据)
insert overwrite table tmp.trial_lesson_success partition(dt='20190419')
2. 新增
insert into table tmp.trial_lesson_success partition(dt='20190419')
3. 保存到本地和HDFS
insert overwrite local directory '/root/123456' select * from t_p;
insert overwrite directory '/aaa/test' select * from t_p;
```

3. ### Select

```sql
select [all | distinct] select_expr, select_expr, ...
from table_reference
join table_other on expr
[where where_condition]
[group by col_list [having condition]]
[cluster by col_list
| [distribute by col_list] [sort by| order by col_list]
]
[limit number]

1. order by 会对输入做全局排序，因此只有一个 reducer，会导致当输入规模较大时，需要较长的计算时间。
2. sort by 不是全局排序，其在数据进入 reducer 前完成排序。因此，如果用 sort by 进行排序，并且设置 mapred.reduce.tasks>1，则 sort by 只保证每个 reducer 的输出有序，不保证全局有序。
3. distribute by(字段)根据指定字段将数据分到不同的 reducer，分发算法是 hash 散列。
4. Cluster by(字段) 除了具有 Distribute by 的功能外，还会对该字段进行排序。
5. 如果 distribute 和 sort 的字段是同一个时，此时，cluster by = distribute by + sort by
```

4. ### Join

```sql
　　Hive 中除了支持和传统数据库中一样的内关联、左关联、右关联、全关联，还支持 left semi join 和 cross join，但这两种 JOIN 类型也可以用前面的代替。
　　Hive 支持等值连接 （a.id = b.id ）,  不支持非等值( (a.id>b.id) ) 的连接，因为非等值连接非常难转化到 map/reduce 任务。另外，Hive 支持多 2 个以上表之间的 join。

1. join 时，每次 map/reduce 任务的逻辑
reducer 会缓存 join 序列中除了最后一个表的所有表的记录，再通过最后一个表将结果序列化到文件系统。这一实现有助于在 reduce 端减少内存的使用量。实践中，应该把最大的那个表写在最后（否则会因为缓存浪费大量内存）。

2. left ， right 和 full outer 关键字用于处理 join 中空记录的情况
select a.val, b.val from a left outer join b on (a.key=b.key)对应所有 a 表中的记录都有一条记录输出。输出的结果应该是 a.val, b.val，当a.key=b.key 时，而当 b.key 中找不到等值的 a.key 记录时也会输出:a.val, null所以 a 表中的所有记录都被保留了；“a right outer join b”会保留所有 b 表的记录。

3. join 发生在 where 子句 之前
如果你想限制 join 的输出，应该在 where 子句中写过滤条件——或是在 join 子句中写。
select a.val, b.val from a
left outer join b on (a.key=b.key)
where a.ds='2009-07-07' and b.ds='2009-07-07'
这会 join a 表到 b 表（outer join），列出 a.val 和 b.val 的记录。where 从句中可以使用其他列作为过滤条件。但是，如前所述，如果 b 表中找不到对应 a 表的记录，b 表的所有列都会列出null，包括 ds 列。也就是说，join 会过滤 b 表中不能找到匹配 a 表 join key 的所有记录。这样的话，left outer 就使得查询结果与 where 子句无关了。解决的办法是在 outer join 时使用以下语法：
select a.val, b.val from a left outer join b
on (a.key=b.key and b.ds='2009-07-07' and a.ds='2009-07-07')

4. hive中的特别join
select * from a left semi join b on a.id = b.id;
等同于
select a.id,a.name from a where a.id in (select b.id from b); 在hive中效率极低
select a.id,a.name from a join b on (a.id = b.id);
select * from a inner join b on a.id=b.id;
```

5. ### 行转列

```sql
select lesson_plan_id,quiz_id,
  concat_ws('',collect_set(if(user_type=1,user_id,''))) teacher_id,
  concat_ws('',collect_set(if(user_type=0,user_id,''))) student_id,
  concat_ws('',collect_set(if(user_type=1,count,''))) t_count,
  concat_ws('',collect_set(if(user_type=0,count,''))) s_count
  from tmp.learning_record_user_quiz_summary
  group by lesson_plan_id,quiz_id
```

6. ### json字符串解析

```sql
一个Map结构嵌套了Map，再嵌套了一个数组结构。
{"username":"king","actionInfo":{"id":1,"age":"22","partList":[{"code":"123","uname":"king"},{"code":"0012","uname":"king"}]}}

通过json_tuple, get_json_object,explode等函数将string类型解析出来,使用正则的方式，将中括号替换掉，然后在转化为数组
select username,ai.id,ai.age,p.uname,p.code from test1 
lateral view json_tuple(actioninfo,'id','age','partlist') ai as id,age,partlist
lateral view explode(split(regexp_replace(regexp_extract(partlist,'^\\[(.+)\\]$',1),'\\}\\,\\{', '\\}\\|\\|\\{'),'\\|\\|')) partlist as p
lateral view json_tuple(p,'code','uname') p as code,uname
```

7. 查看函数功能

```sql
desc function extended datediff;
```



## 四、动态分区

1. ### 静态分区、动态分区、单分区和复合分区

```
hive分区可以方便快速定位，查找( 设置分区，可以直接定位到hdfs上相应的文件目录下，避免全表扫描)。 
hive分区可以分为静态分区、动态分区，另外静动态分区又都可以分为复合分区和单分区表。

1. Hive分区的概念与传统关系型数据库分区不同
传统数据库的分区方式：就oracle而言，分区独立存在于段里，里面存储真实的数据，在数据进行插入的时候自动分配分区。
Hive的分区方式：由于Hive实际是存储在HDFS上的抽象，Hive的一个分区名对应一个目录名，子分区名就是子目录名，并不是一个实际字段。所以我们在插入数据的时候指定分区，就是新建一个目录或者子目录，或者在原来目录的基础上来添加数据。对于hive分区而言，可以分为静态分区和动态分区这两个类。

2. 静态分区表
新建表时定义的分区顺序，决定了文件目录顺序（谁是父目录谁是子目录），正因为有了这个层级关系，当我们查询所有year=1024的时候，2014以下的所有日期下的数据都会被查出来。如果只查询月份分区，但父目录都有该日期的数据，那么Hive会对输入路径进行修剪，从而只扫描日期分区，性别分区不作过滤（即查询结果包含了所有性别）。

3. 动态分区表
在使用静态分区的时候，我们首先要知道有什么分区类型，然后每个分区来进行数据的加载，这个操作过程比较麻烦；而动态分区不会有这些不必要的操作，动态分区可以根据查询得到的数据动态地分配到分区中去，动态分区与静态分区最大的区别是不指定分区目录，由系统自己进行过选择。
动态分区模式可以分为严格模式(strict)和非严格模式(non-strict),二者的区别是：严格模式在进行插入的时候至少指定一个静态分区，而非严格模式在进行插入的时候可以不指定静态分区

4. 设置动态分区相关的参数
set hive.exec.dynamic.partition=true //使用动态分区
set hive.exec.dynamic.partition.mode=nonstrick;//无限制模式，如果模式是strict，则必须有一个静态分区，且放在最前面
set hive.exec.max.dynamic.partitions.pernode=10000;//每个节点生成动态分区的最大个数
set hive.exec.max.dynamic.partitions=100000;//生成动态分区的最大个数
set hive.exec.max.created.files=150000;//一个任务最多可以创建的文件数目
set dfs.datanode.max.xcievers=8192;//限定一次最多打开的文件数
set hive.merge.mapfiles=true; //map端的结果进行合并
set mapred.reduce.tasks =20000;  //设置reduce task个数
```

2. ### 未分区转动态分区

```sql
动态分区可以一次性把所有数据都插入不同分区

插入数据（最后字段的名字需要跟动态分区字段的名字一致）
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.dynamic.partition=true;
insert into ods_db_bidata.yxy_curriculum_status_log_bak_1
partition (dt)
select *,regexp_replace(stat_date,"-","") dt
from tmp.yxy_curriculum_status_log_bak_1
distribute by stat_date;

注意
1 外部表同样适用 
2 若分区字段为空，也就是本例中logTime 为空 
3 动态分区不允许主分区静态，从分区动态
4 如果分区是可以确定的话，千万不要用动态分区，动态分区相较与静态分区，效率会低一些。因为动态分区的值是在reduce运行阶段确定的，也就是会把所有的记录distribute by，而Distribute by 按指定字段，将数据划分到不同的Reduce中，所以当数据大的时候，Reduce的数量直接影响着效率的高低。
```



