# Quartus Project - GHRD

从框图入手，理解概念：

于FPGA之间接口交互，通过“桥”连接 F To H， H To F。

* Low H To F， 通常用作寄存器级别的配置。
* High H To F，大容量的Memory的设备，支持更好的BRODGE的传输。

FPGA To HPS Brodge，当有一个桥接时，有一个Master设备对于Slave设备来访问的，允许FPGA通过这个桥来访问桥。

主要的设备都需要有一个Memory设备，HPS必须有Memory。

HPS MPFE 多端口前端的模块——允许FPGA访问HPS DDR。

## SDM模块

FPGA侧的一个Master，自带需要运行的Fimware，和安全相干的feature相关，HPS 的Boot流程中需要SDM深度参与。

## Memory

TBU Translate buffer Unit&#x20;

TCU Translat&#x20;

最早的系统上是使用的MMU，A53 内部自带MMU，上面的两个模块是MMU扩展出来的概念，TCU在DDR中维护一个虚拟地址到物理地址的转换表。

## MPFE（Mult-Port Front End)

MPFE 支持HPFE 访问 DDR，允许FPGA 通过 MPFE来访问DDR。

Switch中主要有一个缓存一致性单元，MPU子系统会有两级缓存，

CCU的作用就是有一个DDR数据区域，恰好缓存在L2的Cache中，Master 通过 Switch 走到CCU中，同时看到L2缓存的数据。主要是用作同步路径，被各个Master同步到。

通过刷新Cache的操作，让物理地址重新写入，通过这个功能来优化。

细节：FPGA访问DDR接口协议需要支持ACE-Lite协议接口，只有满足ACE-Lite接口情况下，才能被传输接口路由到CCU中。

## L3 Address Regions

HPS 能支持48bit

2G DDR 一定是 给DDR用的。超过的部分是从4G以上开始的，最多支持到128GB。

HPS 的外设放在中间。

H To F：1.5GB桥 Base Address 在哪里？（Quartus 工程中仅展示Offect）

LWH2F：2MB（lospace1）

DDR部分通常由OS管理，通常在DTS中给到一个准确的描述。

## HPS

HPS Dedicated I/O interface HPS专用管脚，允许 route 到FPGA侧。

FPGA Fabric interface

* 对于FPGA 接口叫 Avalon
* 对于HPS接口叫 AXI

Cold Reset 目前没有，目前反应在SDM，可以定义为SDM的Cold Reset，还有上电的Reset。

## Boot流程

在FPGA上游设计，由上游设计交给下游设计。

交付.sof 编译好的 FSBL + bitstream 生成，rbf 文件，生成的工具，是quratus\_pfg工具，安装quratus后才可以。

是否有 -o 选项，生成的是 FPGA First 还是 HPS First，有-hps=on 就是HPS First；

从QSPI启动就是ASX4 模式

软件需要适配硬件的工作，可能需要修改，熟悉FPGA侧工具唯一的工作就是熟悉 pfg工具。

quratus工程上有选项要需注意：configurat 需要注意。

HPS debuf access port 建议复用SDM Pin，和FPGA JTAG 复用一个接口。

* rocketboard 资源
* Altera github

Start 当前正在维护的 Active 的 Board的信息。



pfg工具，指导放FPGA bit工具，FPGA，Uboot的固件，jic文件是通过PSG文件来生成的。

有GSRD提供，所有的已经编译好的东西都是可以提供的。

release.rocketboards.org/2024.04

参考设计都在这里。
