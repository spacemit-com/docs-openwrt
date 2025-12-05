# 设备管理

本文档介绍如何通过 EEPROM 实现自适应设备(板型)，包括如何添加新板型支持和实现自适应启动。

后续文档描述的设备和板型等效

## 添加新设备

以下以新增 k1-x_MUSE-Pi 板型为例说明

### Kernel 设备树

如参考 deb1 硬件设计，则拷贝 `k1-x_deb1.dts` 到如下文件，修改相应内容

```sh
target/linux/spacemit/dts/k1-x_MUSE-Pi.dts

```

对于新增的版型，dts文件均存放在以上目录。编译的时候打包进 bootfs 分区，uboot 启动 Kernel 前加载设备树文件。

### uboot 设备树

新增板型需要在 uboot 源码仓库修改，如下：

1. 添加 uboot 设备树 `u-boot-2022.10/arch/riscv/dts/k1-x_MUSE-Pi.dts`
2. 修改 Makefile，将 `k1-x_MUSE-Pi.dtb` 添加到里面，如下：(注意后缀改成dtb)

```diff
 11 dtb-$(CONFIG_TARGET_SPACEMIT_K1X) += k1-x_evb.dtb k1-x_deb2.dtb k1-x_deb1.dtb k1-x_hs450.dtb \
 12                      k1-x_kx312.dtb k1-x_MINI-PC.dtb k1-x_mingo.dtb k1-x_MUSE-N1.dtb \
 13                      k1-x_MUSE-Pi.dtb k1-x_spl.dtb k1-x_milkv-jupiter.dtb \
 14                      k1-x_MUSE-Book.dtb
 15 
```

3. 修改 `uboot-2022.10/board/spacemit/k1-x/configs/uboot_fdt.its`，添加新节点。

```diff
@@ -46,15 +46,6 @@

            algo = "crc32";
         };
      };
+     fdt_4 {
+                        description = "k1-x_MUSE-Pi";
+                        type = "flat_dt";
+                        compression = "none";
+                        data = /incbin/("../dtb/1-x_MUSE-Pi.dtb");
+                        hash-1 {
+                                algo = "crc32";
+                        };
+                };
   };
 
   configurations {
@@ -74,10 +65,5 @@
         loadables = "uboot";
         fdt = "fdt_3";
      };
+     conf_4 {
+                        description = "k1-x_MUSE-Pi";
+                        loadables = "uboot";
+                        fdt = "fdt_4";
+                };
   };
 };
```

4. 修改 `uboot-2022.10/include/configs/k1-x.h`，更新`DEFAULT_PRODUCT_NAME`，FSBL 和 u-boot 将默认根据 `DEFAULT_PRODUCT_NAME`加载 dtb。如果板子有 EEPROM 记录 product_name 等信息，则优先级更高，FSBL 和 u-boot 将使用 EEPROM 的信息实现自适应启动。

```sh
@@ -25,7 +25,7 @@
 #define CONFIG_GATEWAYIP 10.0.92.1
 #define CONFIG_NETMASK   255.255.255.0

-#define DEFAULT_PRODUCT_NAME "k1_deb1"
+#define DEFAULT_PRODUCT_NAME "k1-x_MUSE-Pi"

 #define K1X_SPL_BOOT_LOAD_ADDR  (0x20200000)
 #define DDR_TRAINING_DATA_BASE  (0xc0829000)
```

### 加入方案

如是开发板形态，则修改 `openwrt/target/linux/spacemit/image/k1-sbc.mk`，添加 `k1-x_MUSE-Pi` 板型支持

```sh
DEVICE_DTS := k1-x_deb1 k1-x_MUSE-Pi

```

如果需要新增方案，请参考[方案管理](openwrt_solution_management.md)

## 支持单CS DDR

FSBL 默认支持双 CS DDR，修改 `uboot-2022.10/arch/riscv/dts/k1-x_spl.dts`可以支持单 CS DDR。

```diff
@@ -79,7 +79,7 @@
    ddr@c0000000 {
        /* dram data rate, should be 1200, 1600, or 2400 */
        datarate = <2400>;
-       cs-num = <2>;
+       cs-num = <1>;
        u-boot,dm-spl;
   };
```

如果设备有 EEPROM，支持通过 EEPROM 实现自适应，对应的 key_name 为`ddr_cs_num`,可用 Titantools 工具写号。刷机工具的时候请参考文档[刷机工具使用](https://developer.spacemit.com/documentation?token=O6wlwlXcoiBZUikVNh2cczhin5d)

## 通过 EEPROM 实现自适应

通过进迭 Titanflasher 工具给板子 eeprom 写号，实现启动的自适应，即一个固件支持多个板子

系统启动时读取 eeprom 的信息，匹配 `uboot_fdt.its` 中信息，从而实现多板型支持。

相关文件：

```shell
uboot-2022.10/arch/riscv/dts/k1-x_spl.dts
uboot-2022.10/arch/riscv/dts/k1-x_*.dts
```

### EEPROM 支持列表

- `atmel,24c02`

### 添加新 EEPROM

1. 修改 `uboot-2022.10/arch/riscv/dts/k1-x_spl.dts`，更新 EEPROM 的 I2C 地址，例如新地址为 `0xA0`。

```c
@@ -121,7 +121,7 @@
      eeprom@50{
         compatible = "atmel,24c02";
         u-boot,dm-spl;
         reg = <0x50>;
         reg = <0xA0>;
         bus = <6>;
         #address-cells = <1>;
         #size-cells = <1>;
```

2. 修改 `uboot-2022.10/arch/riscv/dts/k1-x_*.dts`，添加新 EEPROM 的配置。

   ```c
   @@ -60,9 +60,9 @@
        pinctrl-0 = <&pinctrl_i2c2_0>;
        status = "okay";
    
   -    eeprom@50{
   -        compatible = "atmel,24c02";
   -        reg = <0x50>;
   +    eeprom@A0{
   +        compatible = "atmel,24c04";
   +        reg = <0xA0>;
            vin-supply-names = "eeprom_1v8";
            status = "okay";
        };
   ```

### 使用 `tlv_eeprom` 写号

写号是指将 `product_name` 等信息写入 EEPROM。目前，在 EEPROM 中的信息是按照 TLV 编码的，可以使用 u-boot 的 `tlv_eeprom` 命令写号。

1. PC 接上设备的调试串口，设备启动时，在PC串口终端按下键盘的 `s` 键，直到进入 u-boot shell。

   ```shell
   Autoboot in 0 seconds
   => 
   ```

2. 烧写 `product_name`，例如 `k1-x_MUSE-Pi`。

   ```shell
   => tlv_eeprom set 0x21 k1-x_MUSE-Pi
   => tlv_eeprom write
   Programming passed.
   ```

   `Programming passed.` 表示写入成功。

   请以设备 dts 的文件名（不带后缀， 如 k1-x_MUSE-Pi）命名，方便 u-boot 自动加载 dtb。

3. 重启设备，检查是否可以加载 `k1-x_MUSE-Pi` 的 dtb，正常的话 u-boot 有如下打印。

   ```shell
   product_name: k1-x_MUSE-Pi
   ```

SDK 还支持从 EEPROM 读取以下信息：

- Serial Number：`0x23`
- Base MAC Address：`0x24`
- Manufacture Date：`0x25`
- Device Version：`0x26`
- MAC Addresses：`0x2A`
- Manufacturer：`0x2B`
- SDK Version：`0x40`

其中 MAC Address 会更新到 dtb，作为网卡物理地址。
