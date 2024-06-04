# Versal开发环境搭建

## Linux Versal 环境搭建

### Petalinux 环境搭建

PetaLinux提供了很对开发环境和Xilinx器件的发布包适用的功能，包含有常用的命令。PetaLinux密切集成了Vitis和Vivado的功能，更加方便用户对Xilinx器件的开发。

### Yocto环境搭建

Yocto环境是一个更加通用的Linux构建系统，支持广泛的硬件架构和处理平台，允许定制系统。

### 选择方案

Intel平台提供了Yocto Project和分步骤构建的方案，Xilinx器件提供了统一的Yocto Project构建平台，纯粹的Yocto构建平台内部依赖处理丰富，可以避免很多开发问题，方便进行项目集成和项目迁移。

同时当前Intel 和 AMD选用的Linux Kernel长期支持版本均为Linux 6.1 LTS，Yocto Project采用Xilinx Release的版本：4.1 Yocto Codename Langdale。

## 开发环境选择

Xilinx提供了两种Flow的开发环境，分别时Petalinux和Yocto，Petalinux使用BSP创建工程，可以针对开发板创建对应的BSP工程。

PetaLinux

#### 遇到问题：

参考文档：[https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2824503297/Building+Linux+Images+Using+Yocto](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2824503297/Building+Linux+Images+Using+Yocto)

问题一：

```
gaojie@nova:~/workspace/xilinx/versal_vck190/xilinx-yocto$ MACHINE=vck190-versal bitbake petalinux-image-minimal
ERROR: The following required tools (as specified by HOSTTOOLS) appear to be unavailable in PATH, please install them in order to proceed:
  chrpath diffstat lz4c
```

缺少工具

```
sudo apt install chrpath diffstat liblz4-tool
```

问题二：

```
gaojie@nova:~/workspace/xilinx/versal_vck190/xilinx-yocto$ MACHINE=vck190-versal bitbake petalinux-image-minimal
NOTE: Started PRServer with DBfile: /home/gaojie/workspace/xilinx/versal_vck190/xilinx-yocto/build/cache/prserv.sqlite3, Address: 127.0.0.1:39747, PID: 689362
ERROR:  OE-core's config sanity checker detected a potential misconfiguration.
    Either fix the cause of this error or at your own risk disable the checker (see sanity.conf).
    Following is the list of potential problems / advisories:

    Your Python 3 is not a full install. Please install the module distutils.sysconfig (see the Getting Started guide for further information).


Summary: There was 1 ERROR message, returning a non-zero exit code.
```

Python安装dev版本

```
sudo apt install python3-dev
```

## Windows环境搭建

Windows环境下仅需要安装Vivado即可：Vivado安装包领域内部获取下载地址。

```
/opt/software/Xilinx_Unified_2022.2_1014_8888/Xilinx_Unified_2022.2_1014_8888
172.16.87.2
账号：wangming
密码：WANG@ming
```

## Yocto Project本地化

为方便管理，Yocto Project为单独文件夹 source ，由于Xilinx yocto的meta层较多，同时当前开发并不熟悉，所以build 和 source 目录分开，当需要修改或需要进行搜索查找时，仅打开source目录即可。

由于默认在Github上拉取代码，拉取速度较慢——是否全部修改为本地？

### 修改Kernel源为本地仓库

添加了适用于Nova的meta层，类似PetaLinux中的meta-user目录，再meta中新建“补充配置”文件，对原有的文件及变量进行覆盖。本地kernel远端为：[http://172.16.81.146:8081/embedded\_bsp/xilinx\_source/xilinx-yocto/linux-xlnx](http://172.16.81.146:8081/embedded\_bsp/xilinx\_source/xilinx-yocto/linux-xlnx) 采用当前linux-xlinx的6.10内核，选择版本为Release 6.10.60 版本。

Kernel新建补充文件为：sources/meta-novastar/meta-novastar-bsp/recipes-kernel/linux/linux-xlnx\_%.bbappend

内容为：

```bash
FILESEXTRAPATHS:prepend := "${THISDIR}/${PN}:"

do_compile[nostamp] = "1"

SRC_URI:append = " file://2110core.cfg"
KERNEL_FEATURES:append = " 2110core.cfg"

KBRANCH="xlnx_rebase_v6.1_LTS"
KERNELURI = "git://git@172.16.81.146:/embedded_bsp/xilinx_source/xilinx-yocto/linux-xlnx.git;protocol=ssh;name=machine"

SRCREV = "${AUTOREV}"
```

此时重新编译Kernel即可从本地远端拉取代码。

### 修改Uboot为本地源码仓库

uboot 代码仓库与视频产品线进行代码复用。（Kernel不复用是因为当前MPSoC选择分支为Kernel 5.15分支

## Xilinx Yocto 开发环境使用

参考Xilinx官方文档：[https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2824503297/Building+Linux+Images+Using+Yocto](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2824503297/Building+Linux+Images+Using+Yocto)

### 拉取代码

先设置git，使用repo工具

```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
/home/gaojie/Public/repo init -u git@172.16.81.146:embedded_bsp/xilinx_source/xilinx-yocto/yocto-manifests.git -b rel-v2023.2 -m novastar.xml
```

同步代码：

```bash
repo sync
repo start <name> --all
```

编译Yocto SDK

<pre class="language-bash"><code class="lang-bash"><strong>source setupsdk
</strong><strong>MACHINE=vck190-versal bitbake petalinux-image-minimal
</strong></code></pre>

编译过程需要依赖git下载一些仓库，我们将常用仓库在本地进行管理，DL\_DIR本地路径为：

```bash
build/conf/local.conf
DL_DIR ?= "/home/common/xilinx/download-2023.2"

/home/common/xilinx/download-2023.2
```

### Configuring and Building

使能Yocto环境变量：

```bash
source /opt/tools/PetaLinux/2023.1/tool/settings.sh

source setupsdk build_nova/vc1902_vck190/
```

导入硬件设计配置

<pre class="language-bash"><code class="lang-bash">petalinux-config --get-hw-description &#x3C;path to XSA file>
# 第一步
mkdir sources/meta-novastar
<strong>bitbake-layers create-layer sources/meta-novastar/meta-novastar-bsp
</strong><strong>
</strong><strong># 修改meta中配置文件
</strong>sources/meta-novastar/meta-novastar-bsp/conf/machine/novastar-vck190-versal.conf

<strong># 在local中指定配置文件
</strong>build_nova/vc1902_vck190/conf/local.conf
MACHINE ?= "novastar-vck190-versal"
</code></pre>

编译系统镜像

```
petalinux-build # The default image built is petalinux-image-minimal

bitbake petalinux-image-minimal 
```

Packing and Booting

生成boot image(生成用于启动的镜像boot.bin)

<pre><code><strong>petalinux-package --boot --u-boot
</strong>bitbake xilinx-bootbin
</code></pre>

生成用于QSPI FLASH启动用的MCS Image

```
petalinux-package --boot --u-boot --format MCS
Yocto 参考 README.booting.flash.md
```

