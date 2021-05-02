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

对应于 `depot_tools/fetch_configs/[config].py` 的配置格式为。

```python
class Chromium(config_util.Config):
  """Basic Config class for Chromium."""

  @staticmethod
  def fetch_spec(props):
    url = 'https://chromium.googlesource.com/external/gyp.git'
    solution = { 'name'   :'gyp',
                 'url'    : url,
                 'managed'   : False,
                 'custom_deps': {},
    }
    // spec 指定的 json 格式只需要和 .gclient 的格式一致即可。
    spec = {
      'solutions': [solution],
    }
    return {
      'type': 'gclient_git',
      'gclient_git_spec': spec,
    }

  // 指定 gclient 下载的仓库根目录名。
  // 如果是 git, 类似于
  // git <repository> src
  @staticmethod
  def expected_root(_props):
    return 'src'


def main(argv=None):
  return Chromium().handle_args(argv)


if __name__ == '__main__':
  sys.exit(main(sys.argv))
```

- type: 支持三种：
    - gclient
    - gclient_git
    - git
    但是 `git `都没有实现，还不支持。

- [type]_spec: 指定和 `.gclient` 的 json 格式的配置。

- 指定的 property 配处理成和 solution 同级的 json 数据。例如，
```
$ fetch --nohooks  fetch_test  --target_os=linux,android,mac
```

`--target_os=linux,android,mac`对应于

```json
{
  ...
  "gclient_git_spec": {
    "solutions": [
      ...
    ],
    "target_os": [
      "linux",
      "android",
      "mac"
    ]
  }
}
```

## 支持的属性参数 `--property`

参数的值可以通过逗号 `,` 分割多个，例如 `--target_os=linux,android,mac`。



## 