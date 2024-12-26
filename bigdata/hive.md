# HIVE

## hive的安装

## 操作

### 数据导入和导出
从文件向表加载数据
`load data [local] inpath 'path' [overwrite]`

### 分区表

#### 什么是分区表
- 可以选择字段为表分区
- 分区其实就是HDFS的不同文件夹
- 分区表可以极大提高特定场景爱的Hive的操作性能

#### 基本语法
- 创建分区表
`create table tablename(...) partitioned by (分区列 列类型) row format delimited fields terminated by '';`
- 加载数据到分区表
`load data local inpath 'path' into table tablename partition(分区列=字段值)`

##### 创建一个单分区表，分区按照月分区
```hive
create table myhive.score(id string, cid string, score int) 
partitioned by (month string) 
row format delimited fields terminated by '\t';
```

加载数据到分区表中（可以在插入数据时指定分区）
```hive
load data local inpath '/home/hadoop/score.txt' into table myhive.score partition(month='202408');
```
几个分区就有几个文件夹，数据来源于数据文件，分区来源于插入时指定的列

##### 创建一个多分区表，分区层次按照年、月、日
```hive
create table myhive.score2(id string, cid string, score int) 
partitioned by (year string, month string, day string) 
row format delimited fields terminated by '\t';
```
加载数据就要指定多个
```hive
load data local inpath '/home/hadoop/score.txt' into table myhive.score2 partition(year='2024', month='08', day='11');
```
如果有几个分区层次，在插入时要指定每个分区层次 

### 分桶表

分区是将表拆分到不同的子文件夹中进行存储，分桶是将表拆分到固定数量的不同文件中进行存储。分区和分桶可以同时存在。

#### 分桶表创建
开启分桶的自动优化（自动匹配 reduce task 数量和桶数量一致）
`set hive.enforce.bucketing=true;`

创建分桶表

```hive
create table course (c_id string, c_name string, t_id string)
clustered by (c_id)
into 3 buckets
row format delimited fields terminated by '\t'
```

#### 分桶表数据加载

桶表的数据加载不能通过 `load data` 执行，只能通过 `insert select`

1. 创建一个临时表（外部表、内部表都可以），通过 `load data` 加载数据进入表
2. 然后通过 `insert select` 从临时表向桶表插入数据

```hive
-- 创建普通表
create table course_common(c_id string, c_name string, t_id string) row format delimited fields terminated by '\t';
-- 从普通表加载数据
load data local inpath '/home/xxx.txt' into table course_temp;
-- 通过 insert overwrite 给桶表中加载数据
insert overwrite table course select * from course_temp cluster by(c_id)
```
