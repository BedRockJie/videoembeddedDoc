# TRD 参考示例（MRMAC）

参考文档：[https://xilinx.github.io/vck190-ethernet-trd/2023.2/build/html/index.html](https://xilinx.github.io/vck190-ethernet-trd/2023.2/build/html/index.html)

准备搭建的示例环境，通过VCK190板卡和Switch交换机之间传输以太网和PTP数据包的IP，MRMAC IP存在于FPGA PL中，PL还在 TX 和 RX方向上具有PTP数据包处理。

支持功能点：

* 4x10G/4x25G 可配置
* 支持运行中切换10G 和 25G
* PTP（1step or 2step）（E2E or P2P）（Over UDP)&#x20;
* MTU Support 1500 to 9000 size

## 快速使用

在VCK190开发板上，使用MRMAC IP处理外设和PL的PTP协议解决方案，它由四通道MRMAC组成，每个通道都可以独立配置为10G/25G,MRMAC IP每个通道都连接到对立MCDMA 的一个 TX 和 RX通道。 Versal 平台 支持的ACAP。

硬件环境

* VCK190 开发板
* 100G QSFP 转4x25G SFP28电缆
* 100G QSFP28 至 100G QSFP28 电缆

单板跳线设置：

1. 确认移除了J326 的 7，8 管脚的条线。
2. SYSCTRL Boot SW11（ON,OFF,OFF,OFF）
