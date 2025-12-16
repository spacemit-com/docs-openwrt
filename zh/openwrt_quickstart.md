sidebar_position: 2

# 下载和编译

以下文档基于ubuntu22.04描述.

## 环境准备

### 安装依赖

```sh
sudo apt install build-essential clang flex bison g++ gawk \
gcc-multilib g++-multilib gettext git libncurses-dev libssl-dev \
python3-distutils rsync unzip zlib1g-dev file wget jq device-tree-compiler 

```

## 下载代码

代码仓库分为 `bl-v1.0.y,bl-v2.0.y` 两个分支，分别对应 `linux-6.1`、`linux-6.6`。后续主要维护 `bl-v2.0.y` 分支。

```sh
git clone https://gitee.com/bianbu-linux/openwrt.git -b bl-v2.0.y
```

## 拉取 feeds

首次或想更新包时需要运行以下命令。更新包会从 OpenWrt 官网等下载压缩包，请确保网络可用。如果出现某些包下载失败，可重复执行命令

```sh
cd openwrt
./scripts/feeds update -a 
./scripts/feeds install -f -p spacemit_openwrt_feeds -a
./scripts/feeds install -a
```

## 固件编译

以下编译命令的可选参数`V=s`为输出详细日志。目前提供两种方案的编译启动。

```sh
cp feeds/spacemit_openwrt_feeds/spacemit_k1_defconfig .config
make -j12 V=s
```

固件位于`bin/targets/spacemit/DEVICE_debX/*.zip`

## 清理

- 全部清理命令，会把`bin`、`build_dir`、`staging_dir`、`feeds`、`dl`等目录以及`.config`文件都删掉,代码仓库为最原始的状态

```sh
make distclean
```

- 局部清理命令，会把编译输出目录`bin、build_dir、staging_dir`删掉，不包含`dl`目录、`.config`文件等

```sh
make dirclean
```

## 单包编译

### 单编uboot

- 编译

```sh
make package/boot/uboot-spacemit/compile V=s
```

- 清理

```sh
make package/boot/uboot-spacemit/clean V=s
```

### 单编opensbi

- 编译

```sh
make package/boot/opensbi-spacemit/compile V=s
```

- 清理

```sh
make package/boot/opensbi-spacemit/clean V=s
```

### 单编linux

- 编译

```sh
make target/linux/compile V=s
```

- 清理

```sh
make target/linux/clean V=s
```

### 单编adb包

其他包的编译都类似一下的编译方式

- 编译

```sh
make package/utils/adb/compile V=s
```

- 清理

```sh
make package/utils/adb/clean V=s
```

## 烧写

固件 `*.zip`，使用 Titan Flasher 工具刷写至设备板载存储介质，刷机工具使用参考[刷机工具使用](https://developer.spacemit.com/documentation?token=O6wlwlXcoiBZUikVNh2cczhin5d)

固件 `*sdcard.img`，可使用 `dd` 命令写至卡上，设备插卡上电即可实现卡启动

## 支持设备列表

- BPI-F3
- MUSE-Pi

## 软路由

在以上支持设备列表的板型默认开启软路由功能。系统开机后默认开启WiFi AP模式，其中:

SBC方案有线网卡eth1为lan口，eth2为wan

## FAQ

### 内核编译链接 `pthread_once` 出错

基于 ubuntu 20.04 编译 openwrt 出现`pthread_once`的编译报错，可修改`linux-*/certs/Makefile`

```C
openwrt/build_dir/target-riscv64_riscv64_musl_*/linux-spacemit_*/linux-6.1.15/certs/Makefile
最后一行改成：
HOSTLDLIBS_extract-cert =  -lcrypto -pthread
```

### 如何更新uboot/opensbi/linux版本

1. openwrt跟踪`https://gitee.com/bianbu-linux`的uboot/opensbi/linux仓库版本。openwrt会不定期更新该仓库的版本

2. 如果需要手动更新最新的 `uboot/opensbi/linux` 版本，可参考一下方式

   - 确认 gitee 仓库的最新版本的 tar 包已经上传到`https://archive.spacemit.com/openwrt/dl/`，如 `linux-6.1-v1.0.15.tar.xz`

   <img src="static/image.png" alt="" width="800">

   - 更改 Makefile 版本号

```diff
diff --git a/package/boot/opensbi-spacemit/Makefile b/package/boot/opensbi-spacemit/Makefile
index d4572af253..208d2fae6d 100644
--- a/package/boot/opensbi-spacemit/Makefile
+++ b/package/boot/opensbi-spacemit/Makefile
@@ -16,7 +16,7 @@ else
 PKG_NAME:=opensbi
 PKG_RELEASE:=1
 PKG_VERSION:=1.3
-PKG_SOURCE_VERSION:=1.0.5
+PKG_SOURCE_VERSION:=v1.0.15
 
 PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz
 PKG_SOURCE_URL:=https://archive.spacemit.com/openwrt/dl/
diff --git a/package/boot/uboot-spacemit/Makefile b/package/boot/uboot-spacemit/Makefile
index 51e6e97fb7..c69173cfa8 100644
--- a/package/boot/uboot-spacemit/Makefile
+++ b/package/boot/uboot-spacemit/Makefile
@@ -18,7 +18,7 @@ else
 PKG_NAME:=uboot
 PKG_RELEASE:=1
 PKG_VERSION:=2022.10
-PKG_SOURCE_VERSION:=1.0.5
+PKG_SOURCE_VERSION:=v1.0.15
 
 PKG_SOURCE:=uboot-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz
 PKG_SOURCE_URL:=https://archive.spacemit.com/openwrt/dl/

diff --git a/target/linux/spacemit/Makefile b/target/linux/spacemit/Makefile
index ac230af366..148e5a79b8 100644
--- a/target/linux/spacemit/Makefile
+++ b/target/linux/spacemit/Makefile
@@ -20,7 +20,7 @@ CONFIG_EXTERNAL_KERNEL_TREE=$(TOPDIR)/../bsp-src/linux-6.1
 CONFIG_KERNEL_GIT_CLONE_URI=""
 else
 ## download tar.xz from url.
-LINUX_VERSION_CUSTOM:=linux-6.1-1.0.5
+LINUX_VERSION_CUSTOM:=linux-6.1-v1.0.15
 LINUX_SOURCE:=$(LINUX_VERSION_CUSTOM).tar.xz
 LINUX_KERNEL_HASH:=skip
 endif

```

- 更新 kernel config

```sh
make kernel_menuconfig
```

- 编译
   可能会出现失败，如果编译异常，请根据报错原因修正

```sh
make -j12 V=s
```

### 更改下载源为 SpacemiT

SpacemiT 维护一套预编译的安装包，可通过更改源来下载安装

- 修改软件源地址（基于设备上修改）
   `bl-v2.0.y` 为具体版本的标签，请根据需求来修改。

```sh
//vim /etc/opkg/distfeeds.conf

src/gz openwrt_base https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/base
src/gz openwrt_luci https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/luci
src/gz openwrt_packages https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/packages
src/gz openwrt_routing https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/routing
src/gz openwrt_telephony https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/telephony
src/gz openwrt_spacemit_packages https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/targets/spacemit/DEVICE_MUSE-N1/packages
```
