# vmware网络配置

1. 编辑
2. 编辑虚拟网络编辑器
![image-20240708140548049](../images/image-20240708140548049.png)

## 编辑网络环境
![image-20240708144928109](../images/image-20240708144928109.png)

### 设置网段和网关

子网ip设置为以下，避免192.168.10.x有占用，修改为192.168.xx.0，后面所有配置都要一同改
```
192.168.10.0
```
![image-20240708145455290](../images/image-20240708145455290.png)

设置网关完成后点击 NAT设置

![image-20240708145935645](../images/image-20240708145935645.png)

设置为192.168.10.2，网关和子网ip要同一个网段

![image-20240708150002827](../images/image-20240708150002827.png)

### 配置windows网络

找到刚刚的net8

![image-20240708151003055](../images/image-20240708151003055.png)

![image-20240708151826983](../images/image-20240708151826983.png)

## 虚拟机网络配置
### 修改ip地址
修改网卡配置文件
```
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```
修改BOOTPROTO配置

```
BOOTPROTO="static"
```

在后面加入一行，因为上面网段开始是 192.168.10.128，所以设置为131

```
IPADDR="192.168.10.101"
NETMASK="255.255.255.0"
GATEWAY="192.168.10.2"
DNS1="192.168.10.2"
```

### 设置主机名
改个名字，可以不是必须，好分辨，效果
```
hostnamectl set-hostname node1
```
![image-20240708171929834](../images/image-20240708171929834.png)

### 配置主机名映射

#### 修改windows hosts

> C:\Windows\System32\drivers\etc\hosts

```
192.168.20.101 node1
192.168.20.102 node2
192.168.20.103 node3
```

#### 修改每台linux的hosts文件

> /etc/hosts

```
192.168.20.101 node1
192.168.20.102 node2
192.168.20.103 node3
```

