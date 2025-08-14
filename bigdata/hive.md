# HIVE

## hive的安装

### 安装前准备

由于Hive是基于Hadoop的，需要保证服务器基础环境正常，Hadoop集群健康可用

#### 服务器基础环境

集群时间同步、防火墙关闭、主机Host映射、免密登录、JDK安装

#### Hadoop集群健康可用

启动Hive之前必须先启动Hadoop集群。需要注意，需 **等待HDFS安全模式完全关闭之后再启动运行Hive**

Hive不是分布式安装运行的软件，其分布式特性主要借由Hadoop完成。包括分布式存储、计算

### Hadoop与Hive整合

**Hive需要把数据存储在HDFS上，并且需要使用MapReduce作为引擎处理数据，因此需要在Hadoop中添加相关配置属性，以满足Hive在hadoop上运行**

- 修改hadoop中的`core-site.xml` 文件，并且集群同步配置文件，重启生效。
  ```xml
  <configuration>
    <!- 整合Hive ->  
    <property>
      <name>hadoop.proxyuser.root.hosts</name>
      <value>*</value>
    </property>
    <property>
      <name>hadoop.proxyuser.root.groups</name>
      <value>*</value>
    </property>
  </configuration>
  ```

### 安装mysql

``````
https://blog.csdn.net/weixin_45626288/article/details/133220238
``````



#### 授权

先创建用户

``````
CREATE USER 'root'@'%' IDENTIFIED BY '123456';
``````

再授权

``````
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
``````

刷新配置

``````
FLUSH PRIVILEGES;
``````

### 安装hive

#### 解决冲突

``````
rm -rf lib/guava-22.0.jar
cp /server/hadoop-3.4.0/share/hadoop/common/lib/guava-27.0-jre.jar ./lib/
``````

#### 修改配置文件 `hive-env.sh`

拷贝一份（改个名）

``````
mv hive-env.sh.template hive-env.sh
``````

vi `hive-env.sh`

``````
export HADOOP_HOME=/server/hadoop-3.4.0
export HIVE_CONF_DIR=/server/hive-4.0.1/conf
export HIVE_AUX_JARS_PATH=/server/hive-4.0.1/lib
``````

#### 安装驱动

``````
wget https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-j-8.1.0.tar.gz

tar -xzvf mysql-connector-j-8.1.0.tar.gz
cp mysql-connector-j-8.1.0/mysql-connector-j-8.1.0.jar /server/hive-4.0.1/lib/
``````

#### 初始化hive数据库

``````
schematool -dbType mysql -initSchema
``````

**验证是否成功**：

``````
mysql -u root -p -e "USE hive_metastore; SHOW TABLES;"
``````

#### 配置 Hadoop 以支持 Hive

``````
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -mkdir -p /user/hive/tmp
hdfs dfs -chmod -R 777 /user/hive
``````

#### 重启hadoop

``````
stop-dfs.sh && stop-yarn.sh
start-dfs.sh && start-yarn.sh
``````



### hive环境变量

``````
# HIVE_HOME
export HIVE_HOME=/server/hive-4.0.1
export PATH=$PATH:$HIVE_HOME/bin
``````





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
