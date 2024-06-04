# Agilex开发环境搭建

Agilex的开发环境分为两种

* Yocto Project
* Altera Opensource

我们采用Yocot Project，Yocto Project可以一键生成我们所需的各种类型的固件。

单独编译内核需要安装多个扩展工具：

* bison
* flex
* python3-setuptools
* swig
* libssl-dev

目前这些工具均已单独安装。

## Windwos 环境安装

Windows需要安装如下工具：QuartusProProgrammerSetup-23.4.0.79-windows.exe

这个工具主要针对Soc Programmer，下载程序及仿真调试，无需进行综合仿真。

## Linux环境安装

Embedded Linux开发环境：QuartusProProgrammerSetup-23.4.0.79-linux.run

这个工具也是主要针对Soc Programmer，可以用于程序打包及烧录。

## Design Flow

Soc的设计流程比较关键，遵循原厂的开发流程可以获得更加良好的技术支持。原厂推荐该Soc设计方案应该按照如下流程进行：

* 系统规划：规划、设计规范、IP选择
* 器件选择：设备需求、设备能力容限、软件包产品、速率等级
* 安全考虑：身份验证、加密、base security、firewalls、fuses
* 硬处理系统：带宽分析、防火墙规划、HPS引导方法、IO规划、bridge 和 SDRAM配置
* 设计输入：Coding、Design设计、平台设计、分层设计
* 单板设计考虑：电源及热功率、热设计管理、板机设计指南、配置方案、引导方案、信号完整性、IO和时钟规划、引脚连接、重置计划、内存接口
* 设计验证：System Console、仿真、时钟调试
* 调试：调试工具、远程调试、system consile、JTAG
* 嵌入式软件设计：软件需求和体系结构、工具、驱动、应用程序开发、测试和验证。

## 嵌入式设计流程

