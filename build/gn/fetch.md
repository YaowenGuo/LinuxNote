# dept tools 的 fetch

fetch 是对版本控制命令的一个封装，简化版本控制操作。

```
fetch [option1 [option2 ...]] <config> [--property=value [--property2=value2 ...]]
```


## 支持的选项参数 `option`

```
-h, --help, help   Print this message.
--nohooks          Don't run hooks after checkout.
--force            (dangerous) Don't look for existing .gclient file.
-n, --dry-run      Don't run commands, only print them.
--no-history       Perform shallow clones, don't fetch the full git history.
```

## config

`config` 是 `depot_tools/fetch_configs` 下的一个配置文件名。这个文件配置了要拉取的仓库的信息。

要返回一个这种格式的 json 结构

```
{
  "type": "gclient_git",
  "gclient_git_spec": {
    "solutions": [
      {
        "name": "src",
        "url": "https://webrtc.googlesource.com/src.git",
        "deps_file": "DEPS",
        "managed": false,
        "custom_deps": {}
      }
    ],
    "with_branch_heads": true,
    "target_os": [
      "linux",
      "android",
      "mac"
    ]
  }
}
```

- type: 支持三种：
    - gclient
    - gclient_git
    - git

但是 `git `都没有实现，还不支持。



## 支持的属性参数 `--property`

参数的值可以通过逗号 `,` 分割多个，例如 `--target_os=linux,android,mac`。



## 