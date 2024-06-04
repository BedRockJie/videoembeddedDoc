# 初识Agilex

Altera FPGA目前已独立于Intel作为独立公司运营。Altera FPGA内部有多个系列。

目前我们使用的为Intel 10nm或者 Intel 7工艺制程FPGA，使用Agilex 7系列FPGA，我们使用的型号为：AGF014

Agilex 7 系列FPGA DDR时钟最多以1/4速率传输到内核。

## Agilex 7 Soc中的硬核处理器系统

硬核处理系统（HPS）由多核Arm处理器组成，此外，HPS还增加了用于实现全系统的硬件虚拟化的系统存储管理单元，HPS 结构图：

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

逻辑内核互联有&#x20;

* HPS 到 FPGA 桥接
* HPS 到 SDM 核 SDM 到 HPS 桥接
* 轻型HPS 到 FPGA桥接 轻型32位 AMBA AXI 接口
* FPGA到HPS桥接

## Intel FPGA中的器件配置核SDM

所有的Agilex 7 系列Soc都包含一个SDM，SDM是一种三冗余处理器，用做所有的JTAG命令和配置命令进入器件的进入点，此外，Agilex FPGA和Soc种的SDM系统认证符合 FIPS140-3第二层标准

SDM引导Intel Soc中的HPS，这种引导确保了HPS使用与FPGA相同的安全特性进行启动。

SDM结构图：

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

