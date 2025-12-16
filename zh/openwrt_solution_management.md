sidebar_position: 4

# 方案管理

本文档介绍进迭 OpenWrt SDK 的方案管理，目前发行的 SDK 默认适配 k1-sbc。
每个方案支持多个板型，如 k1-sbc 方案支持 k1-x_MUSE-Pi、k1-x_deb1 板型，未来会持续更新。

## 方案总览

以开发板方案 k1-sbc 为例，通常跟以下配置文件有关，后续章节逐一介绍

```sh
#方案配置
feeds/spacemit_openwrt_feeds/spacemit_k1_defconfig

#方案编译入口
openwrt/target/linux/spacemit/Makefile

#方案定义
openwrt/target/linux/spacemit/k1-sbc/config-6.1
openwrt/target/linux/spacemit/k1-sbc/target.mk
openwrt/target/linux/spacemit/k1-sbc/base-files/

#方案设备树管理
openwrt/target/linux/spacemit/dts/

#方案启动参数
openwrt/target/linux/spacemit/image/env_k1-x.txt

#固件定义
openwrt/target/linux/spacemit/image/k1-sbc.mk
#方案的固件分区
openwrt/target/linux/spacemit/image/partition_tables/partition_2M.json
openwrt/target/linux/spacemit/image/partition_tables/partition_flash.json
openwrt/target/linux/spacemit/image/partition_tables/partition_universal.json

#方案首次启动配置
openwrt/target/linux/spacemit/base-files/etc/uci-defaults/
```

### 方案配置

`feeds/spacemit_openwrt_feeds/spacemit_k1_defconfig`

k1-sbc 方案的编译配置，用于指导 OpenWrt 的编译行为

### 方案 Makefile

`openwrt/target/linux/spacemit/Makefile`

SUBTARGETS 添加方案名称

```c
    ...
 12 SUBTARGETS:=k1-nas k1-sbc
    ...
```

### 方案目录

`openwrt/target/linux/spacemit/k1-sbc`

创建跟方案名称同名目录：`k1-sbc`，包含以下内容

- `openwrt/target/linux/spacemit/k1-sbc/config-6.1`

  `config-6.1` 为方案的 Kernel 配置，编译 Kernel 时合并 `openwrt/target/linux/generic/config-6.1` 与本方案 `config-6.1`，相同的配置选项以本方案为主

- `openwrt/target/linux/spacemit/k1-sbc/target.mk`

  定义方案的通用信息，如 `DEVICE_TYPE:=router`。

  此为 OpenWrt 定义，不同 device type 默认包含不同软件包

- `openwrt/target/linux/spacemit/k1-sbc/base-files/`

  包含要打包进 rootfs 的配置文件，如下：

```sh
openwrt/target/linux/spacemit/k1-sbc/base-files$ tree
.
├── etc
│   ├── board.d
│   │   ├── 01_leds
│   │   └── 02_network
│   ├── hostapd.conf
│   ├── hosts
│   ├── init.d
│   │   └── custom_wifi_ap
│   └── inittab
├── lib
│   ├── preinit
│   │   └── 79_move_config
│   └── upgrade
│       └── platform.sh
└── usr
    └── bin
        ├── uas-gadget2-bot.sh
        ├── uas-gadget2.sh
        └── uas-gadget3.sh

8 directories, 11 files
```

### 方案设备树

`openwrt/target/linux/spacemit/dts/`

不同板型的内核设备树，可参考[设备管理](openwrt_device_management.md)添加一个新板型支持

### 方案分区表

`openwrt/target/linux/spacemit/image/partition_tables/`

这里的分区表是多个方案通用。里面包含不同存储介质的分区表配置，

也可自行添加跟板载存储介质容量一致的分区表，TitanFlasher工具烧写固件时自动匹配

- `partition_2M.json`，用于nor刷机启动，一般与 emmc/ssd 等 blk 设备一起使用
- `partition_universal.json`，用于 blk 设备的刷机启动，
- `partition_flash.json`，用于 SpacemiT Titanflasher 工具制作卡量产

修改分区表可能会影响到系统正常启动，详细的修改方式请参考[启动开发指南](https://bianbu-linux.spacemit.com/device/boot/)

### 方案启动参数

`openwrt/target/linux/spacemit/image/env_k1-x.txt`

uboot 最高优先级环境变量，这里可以设定 bootargs 启动参数，loglevel 等

默认的bootargs为：

```sh
commonargs=setenv bootargs earlycon=${earlycon} earlyprintk console=tty1 console=${console} loglevel=${loglevel} clk_ignore_unused swiotlb=65536 rdinit=${init}
```

可在 `env_k1-x.txt` 中重定义 `earlycon/console/loglevel/init` 等环境变量

```sh
# Common parameter
earlycon=sbi
console=ttyS0,115200
init=/init
bootdelay=0
loglevel=8
```

也可以直接重定义整个 bootargs

```sh
bootargs=earlycon=sbi console=ttyS0,115200 loglevel=4
```

### 方案固件

`openwrt/target/linux/spacemit/image/k1-sbc.mk`

制定方案的固件编译，如包含支持的板型、生成 `sdcard.img`、`spacemit.zip`刷机包等。

比如不想生成 `sdcard.img`，可去掉 `IMAGE/pack.zip中的sdcard-img`。

比如对本方案增加 MUSE-Pi 板型支持，则 DEVICE_DTS 增加 k1-x_MUSE-Pi 设备树名称，

且 `openwrt/target/linux/spacemit/dts/` 需有一份 `k1-x_MUSE-Pi.dts`

```sh
# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2024 Spacemit Ltd.

define Device/debX
  DEVICE_VENDOR := Spacemit
  DEVICE_MODEL :=k1-x deb board
  DEVICE_DTS_DIR:= ../dts
  DEVICE_DTS := k1-x_deb1 k1-x_MUSE-Pi 
  SOC := KeyStone
  KERNEL_NAME := Image
  KERNEL_IMG := Image.itb
  KERNEL := kernel-bin | fit none
  IMAGES := pack.zip
  IMAGE/pack.zip := $(KERNEL_IMG) | boot-common | sdcard-img | archive-zip
endef
TARGET_DEVICES += debX

```

### 方案首次启动

`openwrt/target/linux/spacemit/base-files/etc/uci-defaults/`

UCI 默认设置提供了一种使用 UCI 预配置您的镜像的方法。 要在设备首次启动时设置一些系统默认值，

请在目录此中创建一个脚本。

## 新增方案

以 k1-sbc 方案为例，需要修改以下内容：

1. 修改 `openwrt/target/linux/spacemit/Makefile` 中的 SUBTARGETS，添加方案名称，如 k1-sbc
2. 新增方案目录，如 `openwrt/target/linux/spacemit/k1-sbc/`，添加如下文件

```sh
openwrt/target/linux/spacemit/k1-sbc/config-6.1
openwrt/target/linux/spacemit/k1-sbc/target.mk
```

3. 新增固件编译和兼容板型 `openwrt/target/linux/spacemit/image/k1-sbc.mk`

4. `feeds/spacemit_openwrt_feeds/` 目录下新增编译配置，如 k1-sbc 方案`spacemit_k1_defconfig`

### 新增板型

每个方案下至少有一个或多个板型，新增板型支持可参考[设备管理](openwrt_device_management.md)

### uboot/opensbi 编译配置

如有需要增加 `uboot/opensbi` 新的编译配置，可参考修改以下内容

#### uboot

1. uboot 源码仓库新增编译配置，如 `u-boot-2022.10/configs/k1_defconfig`
2. 修改 `openwrt/package/boot/uboot-spacemit/Makefile`

```sh
 50 define Build/Configure
 51     $(MAKE) -C $(LOCAL_SOURCE_DIR) k1_defconfig
 52 endef
 53 
```

#### opensbi

1. opensbi 源码仓库新增编译配置，如 `opensbi-1.3/platform/generic/configs/k1_defconfig`
2. 修改 `package/boot/opensbi-spacemit/Makefile`

```sh
 59 define Build/Compile
 60     $(eval $(Package/opensbi_$(BUILD_VARIANT))) \
 61         +$(MAKE_VARS) $(MAKE) -C $(LOCAL_SOURCE_DIR) \
 62         PLATFORM=$(PLAT) PLATFORM_DEFCONFIG=k1_defconfig
 63 endef
 64 
```
