# Versal系统架构

AMD Versal™ 器件按不同应用和市场目标划分为多个系列。有些组件在不同系列之间保持一致，有些则在可用性或功能特性上有所不同。主要资源包括但不限于：

* AI 引擎
* 可编程逻辑 (PL)
* 片上网络 (NoC)
* 高速 I/O (XPIO)
* 集成存储器控制器（DDR 存储器控制器）
* 高带宽存储器 (HBM)
* 处理器系统 (PS)
* 平台管理控制器 (PMC)
* Integrated Block for PCIe® ，含 DMA 和高速缓存一致性互连 (CPM)
* 收发器 (GT)
* 高速调试端口 (HSDP)
* 高速连接和加密集成 IP

## Soc可编程逻辑



## NoC

片上网络 (NoC) 属高速通信子系统，可在 PL、PS 和其他集成块中的 IP 端点之间传输数据，以提供统一的裸片内部连接。**NoC 主接口和从接口可配置为 AXI3、AXI4 或 AXI4‑Stream**。NoC 将这些 AXI 接口转换为 128 位宽的 NoC 数据包协议，分别通过水平 NoC (HNoC) 和垂直 NoC (VNoC) 在器件上进行横向和纵向数据移动。

HNoC 在 Versal 自适应 SoC 底部和顶部运行，靠近 I/O bank 和集成块（例如，处理器、存储器控制器和 PCIe）。

VNoC 数量（最多 8 个 VNoC）取决于器件和 DDR 存储器控制器的数量（最多 4 个 DDR 存储器控制器）。

NoC 必须在早期启动时以及使用 NoC 数据路径前，通过 NoC 编程接口 (NPI) 完成配置或编程。NPI 用于对 NoC 寄存器进行编程，这些寄存器可定义布线表、速率调制和 QoS 配置。通过 NPI 对 NoC 进行编程通常无需用户干预。编程完全由平台管理控制器 (PMC) 嵌入式 NPI 控制器自动执行。（AM011）《Versal 自适应 SoC 技术参考手册》

AXI NoC IP 是将 PS 或 PL 连接到 DDR 存储器控制器时所不可或缺的工具。AXI NoC IP 还可用于在 PS 与 PL 之间或在 PL 内的设计模块之间创建其他连接。（PG313）《Versal Adaptive SoC Programmable Network on Chip and Integrated Memory Controller LogiCORE IP 产品指南》

## 适用于 PS、PMC 和 CPM 的 CIPS

如下图所示，PS 模块、PMC 模块与 CPM 模块组合在一起，并使用 Control, Interface, and Processing System (CIPS) IP 核进行配置。

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

在 PS 中，RPU 位于低功耗域 (LPD) 中，APU 位于全功耗域 (FPD) 中，而平台管理控制器 (PMC) 则位于 PMC 功耗域中。

## APU

应用处理单元 (APU) 含连接到 1 MB 统一 L2 高速缓存的双核 Arm® Cortex®-A72 处理器。此 APU 专为无需实时性能的系统控制和计算密集型应用而设计。要提高 Versal 自适应 SoC 性能，就需要提高存储器子系统的性能。

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## PMC

平台管理控制器 (PMC) 子系统含以下功能：

* 启动和配置管理
* Dynamic Function eXchange (DFX)
* 功耗管理
* 可靠性和安全功能
* 生命周期管理，包括器件完整性、调试和系统监控
* I/O 外设

PMC 块可通过执行 BootROM 与 Platform Loader and Manager (PLM) 来对处理器系统、CPM、PL、NoC 寄存器初始化和设置以及 I/O 和中断配置设置的启动和配置操作进行处理。除了启动和配置外，PLM 还可提供生命周期管理服务。与先前器件相比，PMC 总线架构和集中集成可显著提高配置速度和回读性能。下表显示了 Zynq UltraScale+ MPSoC 块与 Versal 自适应 SoC 块的比较结果。

PLM信息 UG1304 《Versal 自适应 SoC 系统软件开发者指南》

AM011《Versal 自适应 SoC 技术参考手册》

## 闪存控制器

PMC 包括 3 种类型的闪存控制器，每一种都可以用作启动器件，也可供应用使用。

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Versal 自适应 SoC 可以支持辅助启动模式（如以太网、USB 等）。

UG1304《Versal 自适应 SoC 系统软件开发者指南》

## 高速连接和加密集成IP

### MRMAC

Versal 自适应 SoC Multirate Ethernet MAC (MRMAC) 可提供高性能低时延的以太网端口，支持广泛的自定义和统计数据收集功能。

* 适用于 25/50/100GE NRZ 支持的第 91 条 RS(528, 514) KR4 FEC
* 适用于 50/100GE PAM4 支持的第 91 条 RS(544, 514) KP4 FEC
* 适用于 10/25/40/50GE 低时延支持的第 74 条 FEC

MRMAC 具有丰富的旁路模式，用于支持访问仅限 FEC 模式（面向定制协议）和 FEC+PCS（面向协议测试器）

参考：PG314《Versal 器件 Integrated 100G Multirate Ethernet MAC (MRMAC) LogiCORE IP 产品指南》
