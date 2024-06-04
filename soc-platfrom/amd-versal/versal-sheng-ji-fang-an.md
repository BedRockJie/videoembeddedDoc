# Versal升级方案

## BootGen工具

BootGen工具为Xilinx器件提供Boot引导固件的生成工具，大多数bootgen均使用命令行方式驱动，命令行选项可以编写脚本。Bootgen工具由引导映像格式（BIF）配置文件驱动，文件扩展名为\*.BIF。

BootGen工具在安装Vivado和Vitis后，可以通过XSCT启动并使用该工具。

命令行工具提供了更多用于创建启动镜像的选项，使用BootGen接口查看命令行选项：

* Vitis GUI&#x20;
* Command Line

Versal ACAP Boot镜像格式：Programmer Device Image（PDI）

Platform Management Controller：

PMC 负责 Versal 平台的平台管理，包括引导和配置，其中主要由两个Microblaze处理器、Rom Code unit（RCU）、physical unclonable function（PUF）。

PPU：包含一个Microblaze 处理器和专用的384KB PPU RAM，PPU中的程序主要运行加载PLM。

```
```

目前在Yocto中生成BOOT.BIN 的 bif 文件如下：

```
the_ROM_image:
{
        image {
         { type=bootimage, file=/home/gaojie/workspace/xilinx/versal_vck190/xilinx-yocto/build_nova/vc1902_vck190/tmp/work/novastar_vck190_versal-xilinx-linux/xilinx-bootbin/1.0-r0/recipe-sysroot/boot/base-design.pdi }
         { type=bootloader, file=/home/gaojie/workspace/xilinx/versal_vck190/xilinx-yocto/build_nova/vc1902_vck190/tmp/work/novastar_vck190_versal-xilinx-linux/xilinx-bootbin/1.0-r0/recipe-sysroot/boot/plmfw.elf }
         { core=psm, file=/home/gaojie/workspace/xilinx/versal_vck190/xilinx-yocto/build_nova/vc1902_vck190/tmp/work/novastar_vck190_versal-xilinx-linux/xilinx-bootbin/1.0-r0/recipe-sysroot/boot/psmfw.elf }
        }
        image {
         id = 0x1c000000
         { type=raw,  load=0x1000, file=/home/gaojie/workspace/xilinx/versal_vck190/xilinx-yocto/build_nova/vc1902_vck190/tmp/work/novastar_vck190_versal-xilinx-linux/xilinx-bootbin/1.0-r0/recipe-sysroot/boot/devicetree/system-top.dtb }
         { core=a72-0,  exception_level = el-3,  trustzone, file=/home/gaojie/workspace/xilinx/versal_vck190/xilinx-yocto/build_nova/vc1902_vck190/tmp/work/novastar_vck190_versal-xilinx-linux/xilinx-bootbin/1.0-r0/recipe-sysroot/boot/arm-trusted-firmware.elf }
         { core=a72-0,  exception_level = el-2, file=/home/gaojie/workspace/xilinx/versal_vck190/xilinx-yocto/build_nova/vc1902_vck190/tmp/work/novastar_vck190_versal-xilinx-linux/xilinx-bootbin/1.0-r0/recipe-sysroot/boot/u-boot.elf }
        }
}
```



## 生产烧录方案



## QSPI FLSH启动及刷写

SPI FLASH没有进行分区，目前参考设计中默认将SPI正片256MB作为一个partition，设备树描述如下

<pre><code><strong>flash@0 {
</strong>				#address-cells = &#x3C;0x01>;
				#size-cells = &#x3C;0x01>;
				compatible = "m25p80\0jedec,spi-nor";
				reg = &#x3C;0x00 0x01>;
				parallel-memories = &#x3C;0x00 0x8000000 0x00 0x8000000>;
				spi-tx-bus-width = &#x3C;0x04>;
				spi-rx-bus-width = &#x3C;0x04>;
				spi-max-frequency = &#x3C;0x8f0d180>;

				partition@0 {
					label = "spi0-flash0";
					reg = &#x3C;0x00 0x10000000>;
				};
			};
</code></pre>

挂载QSPI FLAH：

<pre><code><strong>sf probe 0 0 0
</strong><strong>SF: Detected n25q00a with page size 512 Bytes, erase size 128 KiB, total 256 MiB
</strong>
Versal> sf -h
sf - SPI flash sub-system

Usage:
sf probe [[bus:]cs] [hz] [mode] - init flash device on given SPI bus
                                  and chip select
sf read addr offset|partition len       - read `len' bytes starting at
                                          `offset' or from start of mtd
                                          `partition'to memory at `addr'
sf write addr offset|partition len      - write `len' bytes from memory
                                          at `addr' to flash at `offset'
                                          or to start of mtd `partition'
sf erase offset|partition [+]len        - erase `len' bytes from `offset'
                                          or from start of mtd `partition'
                                         `+len' round up `len' to block size
sf update addr offset|partition len     - erase and write `len' bytes from memory
                                          at `addr' to flash at `offset'
                                          or to start of mtd `partition'
sf protect lock/unlock sector len       - protect/unprotect 'len' bytes starting
                                          at address 'sector'
sf test offset len              - run a very basic destructive test

</code></pre>

```bitbake
QSPI_KERNEL_OFFSET:${MACHINE} = "0xF00000"
QSPI_KERNEL_SIZE:${MACHINE} = "0x1D00000"
QSPI_RAMDISK_OFFSET:${MACHINE} = "0x2E00000"
QSPI_RAMDISK_SIZE:${MACHINE} = "0x4000000"
```

QSPI默认启动验证：

阅读当前代码，QSP启动有

```
sf probe 0 0 0

Versal> sf erase 0x0  0x10000000
SF: 268435456 bytes @ 0x0 Erased: OK

Versal> fatload mmc 0:1 0x10000000  BOOT.BIN
2929296 bytes read in 375 ms (7.4 MiB/s)

Versal> sf write 0x10000000 0x0 0x2EB6A0
device 0 offset 0x0, size 0x2eb6a0
SF: 3061408 bytes @ 0x0 Written: OK

Versal> fatload mmc 0:1 0x10000000 image.ub
15265396 bytes read in 1925 ms (7.6 MiB/s)

Versal> sf write 0x10000000 0xf40000 0xE8EE74
device 0 offset 0xf40000, size 0xe8ee74
SF: 15265396 bytes @ 0xf40000 Written: OK

Versal> fatload mmc 0:1 0x20000000 boot.scr
3472 bytes read in 13 ms (260.7 KiB/s)

Versal> sf write 0x20000000 0x7f80000 0xD90
device 0 offset 0x7f80000, size 0xd90
SF: 3472 bytes @ 0x7f80000 Written: OK
```

只要BOOT.BIN 烧录到SPI FLASH中的0地址处，Versal就可以从QSPI启动，此时UBoot启动打印日志中就可以看到当前的启动状态：

```
U-Boot 2023.01 (Sep 21 2023 - 11:02:37 +0000)

CPU:   Versal
Silicon: v2
Chip:  v2
Model: Xilinx Versal vck190 Eval board revA
DRAM:  2 GiB (effective 4 GiB)
EL Level:       EL2
Core:  39 devices, 21 uclasses, devicetree: board
MMC:   mmc@f1050000: 0
Loading Environment from SPIFlash... SF: Detected n25q00a with page size 512 Bytes, erase size 128 KiB, total 256 MiB
*** Warning - bad CRC, using default environment

In:    serial@ff000000
Out:   serial@ff000000
Err:   serial@ff000000
Bootmode: QSPI_MODE_32
Net:
ZYNQ GEM: ff0c0000, mdio bus ff0c0000, phyaddr 1, interface rgmii-id

Warning: ethernet@ff0c0000 (eth0) using random MAC address - 1e:db:d8:d7:22:b5
eth0: ethernet@ff0c0000Get shared mii bus on ethernet@ff0d0000

ZYNQ GEM: ff0d0000, mdio bus ff0c0000, phyaddr 2, interface rgmii-id

Warning: ethernet@ff0d0000 (eth1) using random MAC address - 8a:40:9b:12:9b:6d
, eth1: ethernet@ff0d0000
Hit any key to stop autoboot:  0
Versal>
Versal>

```

对应到Uboot中的源码：

```
vim board/xilinx/versal/board.c +149
Xilinx 启动Uboot中会有变量来指示：bootmode
QSPI有：QSPI_MODE_32BIT
SD卡为：SD_MODE
```

而生成BOOT.BIN的工具配方为“xilinx-bootbin”

该工具在生成BOOT时，需要指定bif文件，当前bif文件为内部生成，未从外部指定。可以通过变量：

```bitbake
BIF_FILE_PATH
```

来进行外部指定。

当前QSPI 大小为512Mb，容量为64MB，QSPI分区划分如下:

| Flash Partition Name                 | Partition Offset | Partition Size |
| ------------------------------------ | ---------------- | -------------- |
| BOOT.BIN                             | 0x0              | 4MB            |
| Image.ub(Device tree;Kernel;ramdisk) | 0x400000         | 32MB           |
| bootscr                              | 0x2400000        | 512KB          |
| bootenv(SN;MAC)                      | 0x2480000        | 4M             |
| 扩展                                   | 0x2880000        | \~扩展           |

根据该分区，需要调整如下Uboot参数：

* bootscr脚本加载起始地址：BOOT\_SCRIPT\_OFFSET 0x2400000

```
sources/meta-novastar/meta-novastar-bsp/recipes-bsp/u-boot/files/2110card.cfg

CONFIG_BOOT_SCRIPT_OFFSET=0x2400000
```

* bootscr 中 FIT Image 加载地址及索引

<pre><code>build_nova/vc1902_vck190/conf/plnxtool.conf

<strong>QSPI_FIT_IMAGE_OFFSET:${MACHINE} = "0x400000"
</strong>QSPI_FIT_IMAGE_SIZE:${MACHINE} = "0x2000000"
</code></pre>

## 升级方案



## FPGA逻辑加载

Versal系列逻辑加载需要为（PDI）文件，
