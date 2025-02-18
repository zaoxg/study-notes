# WASM处理方法

## 准备

下载工具

> [WebAssembly/wabt](https://github.com/WebAssembly/wabt)
>
> [代码](https://github.com/WebAssembly/wabt/tree/main/wasm2c)

如果报错说 `libcrypto-1_1-x64.dll` 缺失，则需要找到该 `dll` 文件放到 `C:\Windows\System32` 目录下，

> 下载的最新的版本不能用，还好在 `C:\Windows\System32\DriverStore\FileRepository\iclsclient.inf_amd64_dec9bbf83f76d9e5\lib` 目录有一个 `dll`，就直接复制到 `C:\Windows\System32` 下测试可用

## 翻译为C语言

```she
wasm2c Wasm.wasm -o Wasm.c
```

可以得到两个文件，`wasm.c` 和 `wasm.h`
