# Smartisan Pro 2 刷入ROOT

### 参考文章

- [**坚果Pro2搞机指南（2021.5版）**](https://www.coolapk.com/feed/27284754?shareKey=MTI3Mjk4NTZjZTQ3Njc4NWYyZjk~&shareUid=640503&shareFrom=com.coolapk.market_15.0.1)
- [**坚果Pro2搞机指路**](https://www.coolapk.com/feed/19472831?shareKey=NjlhZmI5OGU0N2RjNjc4NWM2Zjg~&shareUid=640503&shareFrom=com.coolapk.market_15.0.1)

## 刷机固件准备

- https://h5.cloud.189.cn/share.html#/t/ZBrYN3IbQjYv

> twrp、面具

### 进入9008方法

完全关机后按住9008线的按钮插入安卓设备，等待win设备管理器出现9008端口

### win设备安装9008驱动

如果安装过9008驱动可以卸载重装

首先要开启开发者

```she
# 重启到bootloader
adb reboot bootloader
```



## 解锁bootloader

## 刷入TWRP

### 方式一

进入 `fastboot` 模式

使用安卓开发工具刷入

```she
fastboot flash recovery twrp-x.x.x-x-x.img
```



### 方式二，使用QFIL工具刷入

#### 连接到9008端口

#### 工具选项配置
1. `Select Buil Type` 选择 `Flat Build`
2. `Select Programmer` 选择要安装的 `*.elf`文件

![image-20250115093807401](images/image-20250115093807401.png)

#### 完成安装
出现以下提示说明安装成功

![image-20250114135151495](images/image-20250114135151495.png)


### 进入recovery验证安装

按 `音量-` 和 `电源键` 进入recovery模式

## 安装面具

面具 **[Magisk](https://github.com/topjohnwu/Magisk)** 开源项目地址

> https://github.com/topjohnwu/Magisk

下载安装包到pc

使用 `adb install mgaisk.apk` 命令将面具安装到安卓设备

#### 从卡刷包中提取boot.img

解压后直接就有 `boot.img` 所以不用自己提取了

![image-20250114165504519](images/image-20250114165504519.png)

##### 特殊情况处理

1. 解压卡刷包，找到 `payload.bin` 文件
2. 使用 **[Payload Dumper](https://github.com/vm03/payload_dumper)** 工具提取 `boot.img`
   - 下载并安装 Payload Dumper 工具（Python 版或 GUI 版）。
   - 将 `payload.bin` 复制到工具目录。
   - 在命令行运行：`python payload_dumper.py payload.bin`
   - 提取的 `boot.img` 会保存在输出目录中。

#### 修补boot.img文件

想办法搞一个当前安卓设备的版本的boot.img ，用magisk app进行修补

```she
PS F:\smartisan\U3 PRO\6.7.4.3-202101191441-user-ob-dfde6640b5> adb push .\boot.img /sdcard/Download/
.\boot.img: 1 file pushed, 0 skipped. 11.4 MB/s (17622316 bytes in 1.472s)
PS F:\smartisan\U3 PRO\6.7.4.3-202101191441-user-ob-dfde6640b5>
```

##### 使用adb命令拿到并保存修补后的文件

```she
adb pull /sdcard/Download/magisk_patched-28100_crSLE.img .\magisk_patched-28100_crSLE.img
```

#### 重启到recovery

刷入卡fastboot补丁

刷入面具

## 刷入ROOT

