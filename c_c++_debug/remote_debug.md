# lldb remote debug


## 1. 获取 lldb-server

如果已经使用 AS 调试过程序，在手机的 /data/local/tmp 已经有了 lldb-server，不用再往手机添加。没有的话，可以在下载的 ndk 的 `toolchains/llvm/prebuilt/<平台类型，如darwin-x86_64>/lib64/clang/9.0.8/` 目录下找到。

然后 push 到手机上。

```
$ adb push lldb-server /data/local/tmp/
$ adb shell
cd /data/local/tmp
chmod 755 lldb-server
```

## 2. 启动 lldb-server

继续在 adb shell 中执行
```
./lldb-server p --server --listen unix-abstract:///data/local/tmp/debug.sock
```

## 3. 启动 lldb 作为客户端

在另一个终端执行

```
$ lldb
platform list # 查看支持连接平台的插件
platform select remote-android # 选择连接安卓设备
platform status # 查看连接状态
```


## 4. 加载流程

1. start lldb server
2. attaching to the app
3. loaded modile: LLVM module