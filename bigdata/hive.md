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