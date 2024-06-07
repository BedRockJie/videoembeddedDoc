# Uboot启动流程

## Uboot启动说明

Uboot标准启动过程中会调用 `board_init_f()` 函数。这个函数通常被用来初始化特定板级支持（Board Support Package，BSP）相关的功能，例如初始化硬件设备、设置时钟、初始化存储等。

在 `board_init_f()` 函数执行完成后，会调用 `arch_cpu_init()` 函数，该函数用于初始化 CPU 相关的功能，例如设置缓存、初始化中断控制器等。

接着，会调用 `board_init_r()` 函数，该函数用于进一步初始化板级支持相关的功能，并最终启动 U-Boot 的命令行解释器。

在 `board_init_r()` 函数执行完成后，U-Boot 将进入命令行模式，等待用户输入命令或者执行启动命令（如 `boot` 命令）来加载内核并启动系统。

`main_loop()` 函数通常在 `board_init_r()` 函数的最后被调用。在 `board_init_r()` 函数中，会初始化各种设备和系统功能，然后通过 `console_start()` 函数启动控制台，最终调用 `main_loop()` 进入主循环。

在 `main_loop()` 中，U-Boot 会不断等待用户输入命令，并根据用户输入执行相应的操作，例如加载内核、启动系统等。同时，`main_loop()` 也负责处理定时器、网络数据包等事件。

总之，`main_loop()` 函数是 U-Boot 的主循环，它负责驱动整个系统的运行。

所有与初始化相关的函数都定义在 `include/init.h` 中，这部分的函数需要厂商自己实现。\


## Uboot启动方案

虽然基于 x86 的平台对于从初始开机时的 BIOS/UEFI 到切换到 Linux 或 Microsoft Windows 等强大的操作系统的启动方式具有事实上的标准化，但这在很大程度上是偶然的，并且与市场上基于 x86 的供应商数量有限有很大关系。相比之下，基于 Arm 的 SoC 平台数量达数十个，几乎每个 SoC 都有某种形式的独特启动过程。

支持所有这些独特的 Arm 平台在最好的情况下可能很繁琐，在最坏的情况下很快就会导致每个平台的期望和体验不一致。这些不一致很快就会升级为相关软件团队的开发时间和维护成本的增加。

## Distro Boot！

这是一种以嵌入式为中心的启动引导方法，目的是为了使其更加“可预测”，支持不同的启动介质。

主要目的是为了启动时发现不同的启动介质，并启动介质发现的逻辑和引导的程序分开，使程序更加具备兼容性。

## 如何设置Distro Boot

发行版启动功能主要作为 U-Boot 中现有 U-Boot bootcmd 功能的扩展来实现。在标准 U-Boot 启动过程中，bootcmd 变量包含启动倒计时结束后自动执行的命令序列（例如，如果未按下任何键）。bootcmd 变量包含按顺序执行的命令序列，以便加载并移交给启动的下一阶段。

### 使用 bootcmd 变量进行发行版启动

```
bootcmd=run distro_bootcmd
```

### 启动目标Boot Targets <a href="#boot-targets" id="boot-targets"></a>

distro\_bootcmd 变量的定义和使用方式因供应商而异，但它通常包含一系列命令，用于扫描预定义的潜在启动目标列表以搜索启动附加信息，如下所示。

```
boot_targets=mmc0 jtag mmc0 mmc1 qspi0 nand0 usb0 usb1 scsi0 pxe dhcp
distro_bootcmd=scsi_need_init=; for target in ${boot_targets}; do run bootcmd_${target}; done
```

该命令通过遍历boot\_targets 中定义的引导列表。这个列表目前被定义在：include/configs/xilinx\_versal.h

头文件中，可以通过不同的宏定义来对引导列表进行区分和开关。

```cpp
#define BOOT_TARGET_DEVICES(func) \
	BOOT_TARGET_DEVICES_JTAG(func) \
	BOOT_TARGET_DEVICES_MMC(func) \
	BOOT_TARGET_DEVICES_XSPI(func) \
	BOOT_TARGET_DEVICES_USB_DFU(func) \
	BOOT_TARGET_DEVICES_USB_THOR(func) \
	BOOT_TARGET_DEVICES_PXE(func) \
	BOOT_TARGET_DEVICES_DHCP(func)
```

## 使用boot.scr方法

这在petalinux中是默认方法。

（第一个Fat32分区）如果没有有效的 extlinux.conf 文件，U-Boot 将扫描 boot\_targets 列表，查找名为 boot.scr.uimg 或 boot.scr 的文件（按此顺序），并运行位于脚本文件中的任何命令。脚本文件语法的示例如下所示。如果您的环境需要不同的命令或行为序列，您可以编辑 boot.scr 文件以满足您的需求。

为了实现QSPI 和 SD卡及 EMMC通用的流程，默认不存放extlinux文件到QSPIFLASH，而是使用boot.scr 的方法。

## 综上

启动组件的优先查找顺序

* extlinux.conf
* boot.scr file + image.ub&#x20;
* boot.scr file + individual files ( Image , system.dtb, etc.)

我们选用 boot.scr file  + image.ub 的启动方案来验证。

## Boot scr方案启动AB系统

