## 基础使用

+   在自己搭建编译环境中使用 [Lean's OpenWrt](https://p3terx.com/go/aHR0cHM6Ly9naXRodWIuY29tL2Nvb2xzbm93d29sZi9sZWRl) 源码生成`.config`文件。

> **TIPS:** 方案默认引用 Lean 的源码。

+   在 Actions 页面选择`Build OpenWrt`，然后点击`Run Workflow`按钮，即可开始编译。

+   最后经过一两个小时的等待，不出意外你就可以在 Actions 页面看到已经打包好的固件目录压缩包。


> **TIPS:** 如需 ipk 文件可以在**进阶使用**章节找到方法。

## 进阶使用

### 自定义环境变量与功能

- 打开 workflow 文件（`.github/workflows/build-openwrt.yml`），你会看到有如下一些环境变量，可按照自己的需求对这些变量进行定义。


```auto
env:
  REPO_URL: https://github.com/coolsnowwolf/lede
  REPO_BRANCH: master
  FEEDS_CONF: feeds.conf.default
  CONFIG_FILE: .config
  DIY_P1_SH: diy-part1.sh
  DIY_P2_SH: diy-part2.sh
  UPLOAD_BIN_DIR: false
  UPLOAD_FIRMWARE: true
  UPLOAD_COWTRANSFER: false
  UPLOAD_WETRANSFER: false
  UPLOAD_RELEASE: false
  TZ: Asia/Shanghai
```

> **TIPS:** 修改时需要注意`:`(冒号)后面有空格。

| 环境变量             | 功能                                                        |
| -------------------- | ----------------------------------------------------------- |
| `REPO_URL`           | 源码仓库地址                                                |
| `REPO_BRANCH`        | 源码分支                                                    |
| `FEEDS_CONF`         | 自定义`feeds.conf.default`文件名                            |
| `CONFIG_FILE`        | 自定义`.config`文件名                                       |
| `DIY_P1_SH`          | 自定义`diy-part1.sh`文件名                                  |
| `DIY_P2_SH`          | 自定义`diy-part2.sh`文件名                                  |
| `UPLOAD_BIN_DIR`     | 上传 bin 目录。即包含所有 ipk 文件和固件的目录。默认`false` |
| `UPLOAD_FIRMWARE`    | 上传固件目录。默认`true`                                    |
| `UPLOAD_COWTRANSFER` | 上传固件到奶牛快传。默认`false`                             |
| `UPLOAD_WERANSFER`   | 上传固件到 WeTransfer 。默认`false`                         |
| `UPLOAD_RELEASE`     | 上传固件到 releases 。默认`false`                           |
| `TZ`                 | 时区设置                                                    |

### DIY 脚本

- 仓库根目录目前有两个 DIY 脚本：`diy-part1.sh`和`diy-part2.sh`，它们分别在更新与安装 feeds 的前后执行，你可以把对源码修改的指令写到脚本中，比如修改默认IP、主机名、主题、添加/删除软件包等操作。但不仅限于这些操作，发挥你强大的想象力，可做出更强大的功能。


> **TIPS:** 脚本工作目录在源码目录，内附几个简单的例子供参考。

### 添加额外的软件包

+   在 DIY 脚本中加入对指定软件包源码的远程仓库的克隆指令。就像下面这样：

```auto
git clone https://github.com/P3TERX/xxx package/xxx
```

+   本地`make menuconfig`生成`.config`文件时添加相应的软件包，如果你知道包名可以直接写到`.config`文件中。

> **TIPS:** 如果额外添加的软件包与 OpenWrt 源码中已有的软件包同名的情况，则需要把 OpenWrt 源码中的同名软件包删除，否则会优先编译 OpenWrt 中的软件包。这同样可以利用到的 DIY 脚本，相关指令应写在`diy-part2.sh`。

- 原理是把软件包源码放到`package`目录下，编译时会自动遍历，与本地编译是一样的。当然方法不止一种，其它方式请自行探索。


### 自定义 feeds 配置文件

- 把`feeds.conf.default`文件放入仓库根目录即可，它会覆盖 OpenWrt 源码目录下的相关文件。


### Custom files（自定义文件）

- 俗称“files 大法”，在仓库根目录下新建`files`目录，把相关文件放入即可。


### 自定义源码

- 默认引用的是 Lean 的源码，如果你有编译其它源码的需求可以进行替换。

- 编辑 workflow 文件（`.github/workflows/build-openwrt.yml`），修改下面的相关环境变量字段。


```auto
        REPO_URL: https://github.com/coolsnowwolf/lede
        REPO_BRANCH: master
```

- 比如修改为 OpenWrt 官方源码19.07分支


```auto
        REPO_URL: https://github.com/openwrt/openwrt
        REPO_BRANCH: openwrt-19.07
```

> **TIPS:** 注意冒号后面有空格

### 编译多个固件

- 基于 GitHub Actions 可同时运行多个工作流程的特性，最多可以同时进行至少20个编译任务。也可以单独选择其中一个进行编译，这充分的利用到了 GitHub Actions 为每个账户免费提供的20个 Ubuntu 虚拟服务器环境。


- 假设有三台路由器的固件需要编译，比如K2P、x86\_64 软路由、新路由3。

  +   生成它们的`.config`文件

  +   分别将它们重命名为`k2p.config`、`x64.config`、`d2.config`放入本地仓库根目录。

  +   复制多个 workflow 文件（`.github/workflows/build-openwrt.yml`）。为了更好的区分可以对它进行重命名，比如`k2p.yml`、`x64.yml`、`d2.yml`。此外第一行`name`字段也可以进行相应的修改。

  +   然后分别用上面修改的文件名替换对应 workflow 文件中下面两个位置的`.config`，不同的机型同样可以使用不同的 DIY 脚本。


```auto
...
    paths:
      - '.config'
...
        CONFIG_FILE: '.config'
        DIY_SH: 'diy.sh'
...
```

### 上传固件到 Releases 页面

- GitHub 的 Releases 页面通常用于发布打包好的二进制文件，无需登录即可下载。Artifacts 和网盘有保存期限，Releases 则是永久保存的。


- 编辑 workflow 文件（`.github/workflows/build-openwrt.yml`），将环境变量`UPLOAD_WERANSFER`的值修改为`true`：


```auto
UPLOAD_RELEASE: true
```

- 编译完成后你可以在 releases 页面找到下载链接。


> **TIPS:** 为了不给 GitHub 服务器带来负担，默认保留 3 个历史记录。

### 定时自动编译

> **TIPS:** 源码更新是不确定的，定时编译经常是在编译没有变动的源码，无意义且浪费资源，所以不建议使用。

- 编辑 workflow 文件（`.github/workflows/build-openwrt.yml`）取消注释下面两行。


```auto
#  schedule:
#    - cron: 0 8 * * 5
```

- 例子是北京时间每周五下午4点（16时）开始编译（周末下班回家直接下载最新固件开始折腾）。如需自定义则按照 cron 格式修改即可，GitHub Actions 的时区为 UTC ，注意按照自己所在地时区进行转换。
