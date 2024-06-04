# Versal系统规划和设计流程

## Vivado工具

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

参考教程连接：[https://xilinx.github.io/Embedded-Design-Tutorials/docs/2022.2/build/html/docs/Introduction/Versal-EDT/Versal-EDT.html](https://xilinx.github.io/Embedded-Design-Tutorials/docs/2022.2/build/html/docs/Introduction/Versal-EDT/Versal-EDT.html)

## CIPS IP 核心

CIPS IP 支持您完成以下配置：

* 配置 PMC、PS、NoC 和（可选）PL 的器件时钟设置
* 配置 PMC 闪存控制器、外设及其关联的多路复用 I/O (MIO)
* 配置 PS 外设及其关联的 I/O
* 配置 PS-PL 中断和交叉触发
* 配置 CPM（含 DMA 和高速缓存一致性互连的 Integrated Block for PCIe® ）
* 配置连接至 NoC 和 PL 的 PS 和 CPM AXI 接口
* 配置系统监控器供电和温度监控和警报
* 配置 HSDP 用于高速调试

参考：PG352：《Control, Interface and Processing System LogiCORE IP 产品指南》

NoC 是使用 AXI NoC IP 配置的。此 IP 充当 NoC 的逻辑表示法。AXI NoC IP 支持 AXI 存储器映射协议，并且 AXIS NoC IP 也支持 AXI4‑Stream 协议。Versal 自适应 SoC 设计可包含每种 IP 类型的多个实例。

DDR 存储器控制器已集成到 AXI NoC IP 中。AXI NoC 实例可配置为包含 1、2 或 4 个 DDR 存储器控制器实例。

您必须使用 NoC IP 来与集成 DDR 存储器控制器进行通信。

在确认步骤中，Versal NoC 编译器按统一流量规格运行。确认后，“NoC Viewer”（NoC 查看器）窗口支持您查看并编辑 NoC 解决方案。

PG313《Versal Adaptive SoC Programmable Network on Chip and Integrated Memory Controller LogiCORE IP 产品指南》

## Vitis工具

Vitis工具有助于创建、验证和集成完整系统的不同要素：

AI引擎工具、Vitis HLS 和 Vitis 编译器、Vivado IP 封装器、Vitis 连接器、Vitis 封装器

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

## 嵌入式软件仿真

嵌入式软件仿真可测试仅以 PS 为目标的软件设计。它以 Quick Emulator (QEMU) 为基础，后者可对 Versal 自适应 SoC 内集成的双核 Arm® Cortex®-A72 的行为进行仿真。此仿真支持对平台操作系统进行快速紧凑的功能确认。此流程包含 SystemC 传输事务级系统模型，支持尽早进行系统浏览和验证。

参考: UG1304 《Versal 自适应 SoC 系统软件开发者指南》中相应内容。

{% embed url="https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/821395464/QEMU+User+Documentation" %}

## 启动镜像

Versal 自适应 SoC PMC 使用专用的启动和配置文件格式来对 Versal 自适应 SoC 进行编程和配置，此格式称为可编程器件镜像 (programmable device image, PDI)。PDI 由以下几个部分组成：头文件、PLM 镜像和将加载到 Versal 自适应 SoC 中的设计数据镜像分区。

PDI 还包含配置数据、ELF 文件、NoC 寄存器设置等。通过 PMC 块由 BootROM 和 PLM 对 PDI 镜像进行编程。

UG1283 《Bootgen 用户指南》

PDI的生成因流程而异：

* 面向仅限硬件系统的传统设计流程
  * 运行 `write_device_image` 命令。UG908 《Vivado Design Suite 用户指南：编程和调试》
* 面向嵌入式系统的传统设计流程
  * 运行 Bootgen 工具。UG1400《Vitis 统一软件平台文档：嵌入式软件开发》

## 嵌入式开发流程

Versal嵌入式开发流程整体上分为BSP和业务软件，业务软件开发与BSP解耦，同时构建新的打包环境便于持续集成和交付。

BSP平台产出 image 镜像包 + SDK 安装包

业务交付拉取SDK安装包、打包同步结合image镜像包
