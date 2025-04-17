# 安装和配置

## 在windows的docker中安装mysql

拉取镜像 `docker pull mysql`

创建3个目录用来将数据/日志/配置映射到宿主机 `/mysql/data`, `/mysql/logs`, `/mysql/conf`

启动

```shell
docker run --name mysql 
-p 3306:3306 
-v E:/docker/home/mysql/data:/var/lib/mysql 
-v E:/docker/home/mysql/logs:/var/log/mysql 
-v E:/docker/home/mysql/conf:/etc/mysql/conf.d 
-e MYSQL_ROOT_PASSWORD=123456 
--restart=always 
--privileged=true 
-d mysql
```



