## 在docker中快速安装mongodb
1. 拉取docker镜像
```
docker pull mongo
```
2. 创建文件夹(创建数据位置、配置、日志的位置文件夹)
```
mkdir -p /home/mongo/conf/
mkdir -p /home/mongo/data/
mkdir -p /home/mongo/logs/
```
3. 新增配置文件，使用配置文件启动mongodb
```
cd /home/mongo/conf/
vi mongod.conf
```
mongd.conf内容
```
# 数据库文件存储位置
dbpath = /data/db
# log文件存储位置
logpath = /data/log/mongod.log
# 使用追加的方式写日志
logappend = true
# 是否以守护进程方式运行
# fork = true
# 全部ip可以访问
bind_ip = 0.0.0.0
# 端口号
port = 27017
# 是否启用认证
auth = true
# 设置oplog的大小(MB)
oplogSize=2048
```
4. 新增日志mongod.log文件
```
cd /home/mongo/logs
vi mongod.log

# 修改权限，不需要在.log输入下面这行
chmod 777 mongod.log
```
5. docker容器构建以及使用配置启动mongodb
```
docker run -it \
	--name mongodb \
	--restart=always \
    --privileged \
    -p 27017:27017 \
    -v /home/mongo/data:/data/db \
    -v /home/mongo/conf:/data/configdb \
    -v /home/mongo/logs:/data/log/  \
    -d mongo \
    -f /data/configdb/mongod.conf
```
6. 进入容器创建账号密码
```
##进入容器##
docker exec -it mongodb /bin/bash

##进入mongodb shell##
mongosh

##切换到admin库##
> use admin

##创建账号/密码##
db.createUser({ user: 'admin', pwd: 'zzaox16', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
```
7. 创建可读写权限账号
```
# 创建前需要先认证
db.auth("admin", "password")
# 查看当前的用户
show users;
# 创建所有数据库管理用户
db.createUser({user: "zzaox", pwd: "zzaox123", roles:["readWriteAnyDatabase", "dbAdminAnyDatabase","clusterAdmin" ]}
)
```

#### 修改用户的密码

**正常修改密码：** `db.changeUserPassword("admin", "new_password")`

**使用 `db.updateUser()` 修改：** `db.updateUser("admin", { pwd: "new_password" })`

**密码丢失：** 需要以 `--noauth` 模式启动 MongoDB 来重置密码



## 在windows下的docker安装

```she
docker run -it --name mongodb --restart=always --privileged -p 27017:27017 -v E:\docker\home\mongo\data:/data/db -v E:\docker\home\mongo\conf:/data/configdb -v E:\docker\home\mongo\logs:/data/log/ -d mongo -f /data/configdb/mongod.conf
```

