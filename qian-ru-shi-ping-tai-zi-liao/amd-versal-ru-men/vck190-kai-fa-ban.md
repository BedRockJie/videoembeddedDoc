# VCK190开发板

## Versal ACAP Confguration

参考AM011

SW1开关：

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

## 镜像编译——Petalinux

使用开发板BSP创建PetaLinux工程

<pre class="language-bash"><code class="lang-bash"><strong>petalinux-create -t project -s xilinx-vck190-v2023.2-10140544.bsp
</strong></code></pre>

工程配置：

```
petalinux-config
```

构建系统镜像：

```
petalinux-build
```

运行这个工具时，它默认生成了FIT镜像，同时也生成了RAM磁盘镜像：rootfs.cpio.gz.u-boot。

默认情况下，还生成了与启动相关的配置，如PMU Firmware、Versal PLM 和 PSM Firmware。

* PLM镜像：`plm.elf` 重新生成命令：`petalinux-build -c plm`
* PSM镜像：`psmfw.elf` 重新生成命令： `petalinux-build -c psm-firmware`
* Image Selector:（仅在Zynq MPSoC平台上）：`imgsel.elf`
* First Stage Boot loader for Zynq: `FSBL`
* Trusted Firmwar-A（TF-A）：bl31.elf 重新生成命令：`petalinux-build -c arm-trusted-firmware`

有关 PLM、PSM的更多信息，参考UG1304。

TF-A信息：[https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842107/Arm+Trusted+Firmware](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842107/Arm+Trusted+Firmware)

## 镜像打包

对于Verasal器件来说，需要打包生成BOOT.BIN。

引导镜像通常包含：PDI file（从硬件设备导入），PLM，PSM firmware，Arm trusted firmware，U-boot，and DTB

使用这个命令生成启动镜像：

```basic
petalinux-package --boot --u-boot
```

他会生成BOOT.BIN,包含有如下文件：

```bash
INFO: File in BOOT BIN: "/home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/project-spec/hw-description/vpl_gen_fixed.pdi"
INFO: File in BOOT BIN: "/home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/plm.elf"
INFO: File in BOOT BIN: "/home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/psmfw.elf"
INFO: File in BOOT BIN: "/home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/system-default.dtb"
INFO: File in BOOT BIN: "/home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/bl31.elf"
INFO: File in BOOT BIN: "/home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/u-boot.elf"
INFO: Generating versal binary package BOOT.BIN...
```

他会生成BOOT.BIN BOOT\_bh.bin 和 qemu\_boot.img 在 images/linux 目录下。默认的DTB loader地址为：0x1000，可参考Bootgen User Guide（UG1238）

如需修改DTB load address，需要使用如下命令：

```
petalinux-package --boot --plm --psmfw --u-boot --dtb --load <load_address>
```

同时DTB 的 load address 可以在petalinux-config 在U-Boot的 menuconfig中。

### 生成MCS Image

Versal的MCS映像通常包含PDI文件（从硬件设计导入）、PLM、PSM固件、Arm®可信固件、U-Boot、DTB和Kernel Fit映像（可选）。

```
petalinux-package --boot --u-boot --format MCS
```

他会生成boot.mcs在image/linux 目录下，默认的DTB load address在0x1000

```
petalinux-package --boot --u-boot --kernel --offset 0xF40000 --format MCS
```

## Boot PetaLinux

petalinux-boot 可以支持 --jtag 和 --qemu两个选项，分别在不同方案中引导，prebuilt option Supporte level 支持 1 到 3.

```
petalinux-boot --jtag --prebuilt <BOOT_LEVEL> --hw_server-url hostname:3121
petalinux-boot --qemu --prebuilt <BOOT_LEVEL>
```

* Level 1:
  * 仅包含FPGA BitStream （不适用Versal，无法跳过PLM加载BitStream）
* Level 2：
  * 对Versal 来说，包含cdo 文件，PLM，PSM 固件和TF-A 以及Uboot。
* Level 3：
  * 对Versal来说，在2的基础上增加Kernel boot。

注意：默认情况我们需要生成tcl脚本，所以需要加上：

```
petalinux-boot --jtag --prebuilt 3 --tcl text.txt
```

## 通过SD卡启动PetaLinux镜像

通过SD卡在硬件上启动PetaLinux镜像，方法是手动复制所需的镜像到SD卡中，或者将WIC镜像擦写道SD卡中。

### 方法一：打包wic文件

```
petalinux-package --wic
```

```
INFO: wic create /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/rootfs.wks --rootfs-dir /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/wic/rootfs --bootimg-dir /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/ --kernel-dir /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/ --outdir /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/wic-tmp -n /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/tmp/work/cortexa72-cortexa53-xilinx-linux/wic-tools/1.0-r0/recipe-sysroot-native
INFO: Creating image(s)...

WARNING: bootloader config not specified, using defaults

INFO: The new image(s) can be found here:
  /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/wic-tmp/rootfs-202405070831-sda.direct

The following build artifacts were used to create the image(s):
  ROOTFS_DIR:                   /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/wic/rootfs
  BOOTIMG_DIR:                  /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/
  KERNEL_DIR:                   /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/images/linux/
  NATIVE_SYSROOT:               /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/tmp/work/cortexa72-cortexa53-xilinx-linux/wic-tools/1.0-r0/recipe-sysroot-native

INFO: The image(s) were created using OE kickstart file:
  /home/gaojie/workspace/xilinx/versal_vck190/xilinx-vck190-2023.2/build/rootfs.wks

```

解压缩 wic文件并烧录

```
xz -d petalinux-sdimage.wic.xz
dd if=petalinux-sdimage.wic of=/dev/sd<X> conv=fsync
```

该方法优势为简单，占用存储空间较小。（实际SD空间占用不够）

缺点，生成wic文件大小为6G，使用dd命令进行进行刷写需要耗费比较多的时间！

### 方法二：手动拷贝到SD卡中

#### 第一步：对SD卡进行分区

使用fdisk工具

<pre><code>sudo fdisk /dev/sdb
Command (m for help): n
Select (default p): p
Partition number (1-4, default 1):
<strong>First sector (2048-62333951, default 2048):
</strong><strong>Last sector, +sectors or +size{K,M,G,T,P} (2048-62333951, default
</strong>62333951): 4194304
Created a new partition 1 of type 'Linux' and of size 2 GiB.
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-62333951, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-62333951, default
62333951): 
Command (m for help): p
Disk /dev/sdb: 29.14 GiB, 31293702144 bytes, 61120512 sectors
Disk model: STORAGE DEVICE
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x860657e9

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  4194304  4192257    2G 83 Linux
/dev/sdb2       4196352 61120511 56924160 27.1G 83 Linux
Command (m for help): w
</code></pre>

主要逻辑是对SD卡做了两个分区，一个用于存放Boot（2GB）另一个用作文件系统。

制作文件系统：

```
sudo mkfs.vfat /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
```

#### 第二步：复制镜像到SD卡

复制镜像\<plnx-proj-root>/images/linux 拷贝如下内容到FAT分区：

* BOOT.BIN
* image.ub
* boot.scr

解压缩rootfs.tar.gz 到SD卡的ext4 分区中。

```
mount /dev/sdb2 /mnt/
sudo tar -zxvf rootfs.tar.gz -C /media/emb/2a7f7c53-9b5a-4fee-a345-e540b646edbe/
```

## 通过JTAG启动PetaLinux镜像



## 通过TFTP启动PetaLinux镜像

### 第一步：Windows 搭建TFTP环境

工具下载连接：[https://bitbucket.org/phjounin/tftpd64/wiki/Download%20Tftpd64](https://bitbucket.org/phjounin/tftpd64/wiki/Download%20Tftpd64)

最好直接下载绿色免安装版本：[https://bitbucket.org/phjounin/tftpd64/downloads/tftpd64.464.zip](https://bitbucket.org/phjounin/tftpd64/downloads/tftpd64.464.zip)

启动后，选择Lnux/image镜像目录进行映射：

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

### 第二步：JTAG下载PLM



## 通过QSPI或OSPI启动PetaLinux镜像



## BEAM Tool&#x20;

电路板评估和管理 Board evaluation and management（BEAM） 工具是一款全新的基于系统控制器的工具，可增强 Versal 评估套件用户的开箱即用体验。

BEAM工具目前还处于Beta阶段，使用开发板镜像发现Versal启动异常，可能于BEAM有关。

VCK190 的 BEAM 资料参考地址：[https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/973078551/BEAM+Tool+for+VCK190+Evaluation+Kit](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/973078551/BEAM+Tool+for+VCK190+Evaluation+Kit)

