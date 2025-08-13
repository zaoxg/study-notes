# HADOOP

## hadoop安装
配置文件位置 `hadoop/etc/hadoop`
### 配置文件
- 配置 `workers` 文件
  - 进入配置文件目录
    `cd etc/hadoop`
  - 编辑 `workers` 文件
    `vim workers`
  - 输入以下内容，要删掉 `localhost`
    ```
    node102
    node103
    node104
    ```
    ![image-20240716230754488](./images/image-20240716230754488.png)

- 配置 `hadoop-env.sh` 文件
  ```
  export JAVA_HOME=/server/jdk/jdk1.8.0_411
  export HADOOP_HOME=/server/hadoop-3.4.0
  export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
  export HADOOP_LOG_DIR=$HADOOP_HOME/logs
  ```

- 配置 `core-site.xml` 文件
  ```xml
  <configuration>
      <!--  设置默认使用的文件系统-->
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://node102:8020</value>
      </property>
    <!--  设置hadoop本地数据保存路径-->
    <property>
      <name>hadoop.tmp.dir</name>
      <value>131072</value>
    </property>
    <property>
      <name>io.file.buffer.size</name>
      <value>131072</value>
    </property>
  </configuration>
  ```

  **对于 `fs.defaultFS` (HDFS地址)**

  - **必须**在所有Hadoop节点（NameNode和所有DataNodes）上配置且保持一致。

- 配置 `hdfs-site.xml`文件

  ```xml
  <configuration>
    <property>
      <name>dfs.datanode.data.dir.perm</name>
      <value>700</value>
    </property>
    <property>
      <name>dfs.namenode.name.dir</name>
      <value>/data/hadoop/nn</value>
    </property>
    <property>
      <name>dfs.namenode.hosts</name>
      <value>node102,node103,node104</value>
    </property>
    <property>
      <name>dfs.blockssize</name>
      <value>268435456</value>
    </property>
    <property>
      <name>dfs.namenode.handler.count</name>
      <value>100</value>
    </property>
    <property>
      <name>dfs.datanode.data.dir</name>
      <value>/data/hadoop/dn</value>
    </property>
  </configuration>
  ```
  + `dfs.datanode.dato.dir.perm`: hdfs文件系统，默认创建的文件权限设置
    值:700，即:rwx------
  + `dfs.namenodename.dir`:NameNode元数据的存储位置值:`/data/hadoop/nn`，在 `node102`节点的 `/data/hadoop/nn` 目录下
  + `dfs.namenode.hosts`:NameNode允许哪几个节点的DataNode连接(即允许加入集群)值: `node102`、`node103`、`node104`这三台服务器被授权
  + `dfs.blocksize` hdfs默认块大小
    值:268435456(256MB)
  + `dfs.namenode.handler.count` namenode处理的并发线程数值:100，以100个并行度处理文件系统的管理任务
  + `dfs.datanode.data.dir`：从节点DataNode的数据存储目录值:/data/dn，即数据存放在nodel、node2、node3，三台机器的/data/dn内

- 准备数据目录
  + 根据配置项创建目录
  + 在主节点
    ```
    sudo mkdir -p /data/hadoop/nn
    sudo mkdir /data/hadoop/dn
    ```
  + 在 `node103`、`node104`节点
    `sudo mkdir -p /data/hadoop/dn`

- 分发hadoop文件夹
  以上操作已经完成了hadoop的配置，可以把配置好的hadoop发往其他机器
  ```
  cd /server
  scp -r hadoop-3.4.0 node103:`pwd`/
  scp -r hadoop-3.4.0 node104:`pwd`/
  ```
### 配置环境变量
为了方便操作hadoop，把hadoop的脚本、程序配置到 `PATH` 中方便后续使用。
1. 编辑环境变量
```
# 在/etc/profile.d/my_env.sh里追加
# HADOOP_HOME
export HADOOP_HOME=/server/hadoop-3.4.0
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
刷新 `source /etc/profile`
2. 在其余节点配置同样的环境变/量

### 修改文件夹权限
#### 授权hadoop用户
为了确保安全，hadoop不以root用户启动，以普通用户启动，所以对文件权限进行授权。
以root身份在各个节点上执行如下命令
```
chown -R hadoop:hadoop /data
chown -R hadoop:hadoop /server/hadoop-3.4.0
```

### 格式化整个文件系统
#### 格式化 `namenode`
```
# 以hadoop用户执行
su hadoop
hadoop namenode -format
```
检查 `/data/hadoop/nn` 内有没有 `current` 文件，如果没有，根据日志找放到哪了，然后检查配置文件 `hdfs-site.xml`

#### 启动
- 一键启动hdfs集群
  `start-dfs.sh`

  ![image-20240717154521741](./images/image-20240717154521741.png)

- 一键关闭hdfs集群
  `stop-dfs.sh`

  ![image-20240717152211944](./images/image-20240717152211944.png)
> 起不起来检查配置，看看哪没写对

### 管理页面
![image-20240717151606984](./images/image-20240717151606984.png)

在namenode的9870端口

```
node102:9870
http://192.168.20.102:9870
```
## hadoop的功能

## HDFS

### 一键启停
####  一键启动HDFS集群 `start-dfs.sh`
> $HADOOP_HOME/sbin/start-dfs.sh

执行原理
- 在执行此脚本的机器上启动 `SecondaryNameNode`
- 读取 `core-site.xml` 内容 (`fs.defaultFS`项)，确认 `NameNode` 所在的机器，启动 `NameNode`
- 读取 `workers` 内容，确定 `DataNode` 所在的机器，启动全部 `DataNode`
####  一键关闭HDFS集群 `stop-dfs.sh`
> $HADOOP_HOME/sbin/stop-dfs.sh

执行原理
- 在执行此脚本的机器上关闭 `SecondaryNameNode`
- 读取 `core-site.xml` 内容 (`fs.defaultFS`项)，确认 `NameNode` 所在的机器，关闭 `NameNode`
- 读取 `workers` 内容，确定 `DataNode` 所在的机器，关闭全部 `DataNode`

### 单进程启停
需要切换到所在的机器上执行才有效果
#### 单独控制所在机器的进程的启停(脚本)
`hadoop-daemon.sh (start|status|stop) (namenode|secondarynamenode|datanode)`
#### 单独控制所在机器的进程的启停(程序)
`hdfs --daemon (start|status|stop) (namenode|secondarynamenode|datanode)`

### 操作
#### 创建文件夹
- `hadoop fs -mkdir [-p] <path> ...`
- `hdfs dfs -mkdir [-p] <path> ...`
>`-p` 的选项与linux `mkdir -p` 一致
>`path` 为待创建的目录
```shell
# 这样是写本地
hadoop fs -mkdir -p file:///home/node102/test
# 这样是写hdfs
hadoop fs -mkdir -p hdfs://node102:8020/bigdata/test
```

#### 查看指定目录下的内容
- `hadoop fs -ls [-h] [-R] [<path> ...]`
- `hdfs dfs -ls [-h] [-R] [<path> ...]`
> `path` 指定目录路径
> `-h` 人性化显示文件size
> `-R` 递归查看指定目录及其子目录

![image-20240717160607625](./images/image-20240717160607625.png)

#### 上传文件到HDFS指定目录下
- `hadoop fs -put [-f] [-p] <localsrc> ... <dst>`
- `hdfs dfs -put [-f] [-p] <localsrc> ... <dst>`
  如果文件已经存在会提示 `put: 'xx.txt': File exists`
> `-f` 覆盖目标文件
> `-p` 保留访问和修改时间、所有权和权限
> `localsrc` linux文件路径
> `dst` hdfs路径

![image-20240717164813459](./images/image-20240717164813459.png)

#### 查看HDFS文件内容
- `hadoop fs -cat <src> ...`
- `hdfs dfs -cat <src> ...`
```shell
# 读取指定文件全部内容，显示在标准输出
hadoop fs -cat /bigdata/test/test.txt
```
读取大文件可以使用管道符配合 `more`
- `hadoop fs -cat <src> | more`
- `hdfs dfs -cat <src> | more`

#### 下载HDFS内容
- `hadoop fs -get [-f] [-p] <src> ... <localdst>`
- `hdfs dfs -get [-f] [-p] <src> ... <localdst>`
  `hadoop fs -get /bigdata/test/test.txt .`

#### 拷贝HDFS文件
命令是HDFS对HDFS的复制
- `hadoop fs -cp [-f] <src> ... <dst>`
- `hdfs dfs -cp [-f] <src> ... <dst>`

`hadoop fs -cp /bigdata/test/test.txt /bigdata/yellow/seed.txt`
复制目的地的文件夹不存在会报错所以要提前创建
![image-20240717171115061](./images/image-20240717171115061.png)

#### 追加数据到HDFS文件
将给定的本地文件的内容追加到给定的dst文件
想要修改文件，要么追加，要么删了修改重新上传
- `hadoop fs -appendToFile <localsrc> ... <dst>`
- `hdfs dfs -appendToFile <localsrc> ... <dst>`
> `dst` 如果dst文件不存在，将创建该文件
> `localsrc` 如果为-，则输入从标准输入中读取

![image-20240717175717651](./images/image-20240717175717651.png)

#### HDFS数据移动操作

- `hadoop fs -mv <src> ... <dst>`
- `hdfs dfs -mv <src> ... <dst>`
  移动文件到指定文件夹下
  可以使用该命令移动数据，重命名文件的名称

![image-20240717175237224](./images/image-20240717175237224.png)

#### HDFS数据删除操作
- `hadoop fs -rm -r [-skipTrash] URI [URI ...]`
- `hadoop fs -rm -r [-skipTrash] URI [URI ...]`
> 删除指定路径的文件或文件夹，指定 `-r` 删除文件夹
> `-skipTrash` 跳过回收站，直接删除

回收站功能默认关闭，如果需要开启需要在 `core-site.xml`内配置
```xml
<configuration>
  <property>
    <name>fs.trash.interval</name>
    <value>1440</value>
  </property>
  <property>
    <name>fs.trash.checkpoint.interval</name>
    <value>120</value>
  </property>
</configuration>
```
> 不需要重启集群，立即生效，在哪个机器配置的，在哪个机器执行命令就生效
> 回收站默认位置在 `/user/用户名hadoop/.trash`



## MapReduce



## YARN

集群物理层面

- ResourceManager
- NodeManager

App层面

- ApplicationMaster（App Mstr）

#### ResourceManager（RM）

YARN集群中的 **主角色**，决定系统中所有应用程序之间 **资源分配的最终权限，即最终仲裁者**。

接收用户的作业提交，并通过NM分配、管理各个机器上的计算资源。

#### NodeManager（NM）

YARN中的**从角色**，一台机器上一个，负责 **管理本机器上的计算资源**。

根据RM命令，启动Container容器、监视容器的资源使用情况。并且向MM主角色汇报资源使用情况。

#### ApplicationMaster（AM）

用户提交的每个应用程序均包含一个AM。

**应用程序内的“老大”**，负责程序内部各阶段的资源申请，监督程序的执行情况。

### 核心交互流程

- MR作业提交 Client -> RM
- 资源的申请 MrAppMaster -> RM
- MR作业状态汇报 Container(Map Reduce Task) -> Container(MrAppMaster)
- 节点的状态汇报 NM -> RM

### 整体概述

当用户向 `YARN` 中提交一个应用程序后，`YARN`将分两个阶段运行该应用程序

1. **客户端申请资源启动运行本次程序的 `ApplicationMaster`**
2. **由 `ApplicationMaster`根据本次程序内部具体情况，为它申请资源，并监控它的整个运行过程，直到运行完成**

#### MR提交YARN交互过程

1. 用户通过客户端向YARN中ResourceManager提交应用程序(比如hadoop jar提交MR程序)

2. Resourcellanager为该应用程序分配第一个Container(容器)，并与对应的NodeManager通信，要求它在这个Container中启动这个应用程序的ApplicationMaster。
3. 第3步、ApplicationMaster启动成功之后，首先向Resourceanager注册并保持通信，这样用户可以直接通过ResourceManage查看应用程序的运行状态(处理了百分之几)
4. AM为本次程序内部的各个Task任务向RM申请资源，并监控它的运行状态
5. 一旦 ApplicationMaster 申请到资源后，便与对应的 NodeManager 通信，要求它启动任务,。
6. NodeManager 为任务设置好运行环境后，将任务启动命令写到一个脚本中，,并通过运行该脚本启动任务,





## 同步文件脚本

**xsync同步脚本编写**

该脚本的作用是可以同时给各个子节点发送文件。

``````
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Argument!
    exit;
fi

# 2. 遍历集群所有机器
for host in node102 node103 node104
do
    echo ========== $host ==========
    # 3. 遍历所有目录
    for file in $@
    do
        # 4. 判断文件是否存在
        if [ -e $file ]
        then
            # 5. 获取父目录
            pdir=$(cd -P $(dirname $file); pwd)
            # 6. 获取当前文件的名称
            fname=$(basename $file)
            ssh $host "mkdir -p $pdir"
            rsync -av $pdir/$fname $host:$pdir
        else
            echo $file does not exists!
        fi
    done
done
``````

配置环境变量

**/etc/profile.d/my_env.sh**


```
# XSYNC_HOME
export XSYNC_HOME=/home/hadoop
export PATH=$PATH:$XSYNC_HOME/bin
```

``````
xsync /server/hadoop-3.4.0/etc/hadoop/
``````

