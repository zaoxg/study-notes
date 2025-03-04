# 安装和配置

## 在docker中安装

使用`docker search redis` 命令搜索

```shell
docker pull redis:latest
```

###  下载安装版本匹配的配置文件

```shell
https://redis.io/docs/latest/operate/oss_and_stack/management/config/
```

### 在宿主机中创建数据和配置文件的目录

## 在windows下的docker安装

```shell
docker run -d --name redis -p 6379:6379 -v E:\docker\home\redis\data:/data -v E:\docker\home\redis\conf\redis.conf:/etc/redis/redis.conf redis redis-server /etc/redis/redis.conf
```

