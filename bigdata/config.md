# 配置

## hadoop

### core-site.xml

```xml
<configuration>
    <!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node102:8020</value>
    </property>
    <!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop</value>
    </property>
    <!-- 文件系统垃圾桶 -->
    <property>
        <name>fs.trash.interval</name>
        <value>1440</value>
    </property>
    
    <!-- 配置HDFS网页登录使用的静态用户为root -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value>
    </property>
    <!-- 整合Hive -->
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

### hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/data/hadoop/nn</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/data/hadoop/dn</value>
    </property>
    
    <property>
        <name>dfs.namenode.hosts</name>
        <value>node102,node103,node104</value>
    </property>
    
    <!-- nn web端访问地址 -->
    <property>
        <name>dfs.namenode.http-address</name>
        <value>node102:9870</value>
    </property>
    <!-- 2nn web端访问地址 -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node104:50090</value>
    </property>
</configuration>
```

### mapred-site.xml

```xml
<!--修改为如下内容：-->
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>node102:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node102:19888</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
</configuration>
```

### yarn-site.xml
```xml
<configuration>
    <!-- Site specific YARN configuration properties -->
    <!-- 指定MR走shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    
    <!-- 指定RM地址 -->
    <property>
        <description>The hostname of the RM.</description>
        <name>yarn.resourcemanager.hostname</name>
        <value>node103</value>
    </property>

    <!-- 是否对容器实施物理内存限制 -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <!-- 是否对容器实施虚拟内存限制 -->
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>

    <!-- 如果不配置日志聚集功能，web页面上的yarn历史任务的日志会查看不了。 -->
    <!-- 开启日志聚集功能 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!-- 设置日志聚集服务器地址 -->
    <property>
        <name>yarn.log.server.url</name>
        <value>http://node103:19888/jobhistory/logs</value>
    </property>
    <!-- 设置日志保留时间为7天 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
</configuration>
```

使用以下命令把etc/hadoop目录的配置发到其他节点

 `scp -r etc/hadoop node103:`pwd`/`



## Hive

### hive-site.xml

``````
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://node102:3306/hive_metastore?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8&amp;useSSL=false&amp;allowPublicKeyRetrieval=true</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
    <!-- 指定存储元数据要连接的地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://node102:9083</value>
    </property>
    <!-- 指定hiveserver2连接的host -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>node102</value>
    </property>
    <!-- 指定hiveserver2连接的端口号 -->
    <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
    </property>
    <!-- HDFS 仓库目录 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    <property>
        <name>hive.exec.scratchdir</name>
        <value>/user/hive/tmp</value>
    </property>
    <!-- hiveserver2的高可用参数，如果不开会导致了开启tez session导致hiveserver2无法启动 -->
    <property>
        <name>hive.server2.active.passive.ha.enable</name>
        <value>true</value>
    </property>
    <!--解决Error initializing notification event poll问题-->
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
</configuration>

``````

