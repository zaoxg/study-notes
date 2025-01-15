# ADB命令

首先要打开usb调试

**adb shell** 进入手机

**adb install** 把apk安装到手机上

## 应用管理

### 查看应用列表

```shell
adb shell pm list packages
```

#### 查看所有应用

```shell
adb shell pm list packages
```

#### 查看系统应用
```shell
adb shell pm list packages -s
```

#### 查看第三方应用
```shell
adb shell pm list packages -3
```

#### 包名包含某字符串的应用
```shell
adb shell pm list packages smartisan
```


