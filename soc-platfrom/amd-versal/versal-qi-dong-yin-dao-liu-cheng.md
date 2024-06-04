# Versal启动引导流程

versal不同于MPSoC，Versal内部有一个专门的PLM控制器，Versal初次上电第一个运行的程序为PLM（除BootROM）还需要配置NoC，DDR，PL（可选）。然后为Arm Trusted Firmware，然后才到Uboot。

到达Uboot后，此时就可以选择不同的启动方式（自己选择），不同于硬件，硬件选择的是PLM的引导方式，可以从不同的存储器中引导PLM。而当硬件固定后，PLM中所需加载的Noc、DDR等均固定下来不需要修改。

本章节启动方式及工具完全参考UG1144（V2023.2）

## Config U-Boot Boot Script (boot.scr)



## EMMC分区划分（SD卡同样适用）

由于fdisk默认会按照MBR分区表来划分，MBR分区表只能划分4个分区，无法满足我们当前的需求。

<table><thead><tr><th width="123">分区名称</th><th width="125">分区类型</th><th>分区介绍</th><th>分区大小</th><th>分区编号</th></tr></thead><tbody><tr><td>Uboot</td><td>FAT32</td><td>存放BOOT.BIN（无FPGA固件）</td><td>64M</td><td>1</td></tr><tr><td>sysconfig</td><td>ext4</td><td>sysconfig环境变量</td><td>128M</td><td>5</td></tr><tr><td>env</td><td>RAW</td><td>Uboot环境变量</td><td>4M</td><td>6</td></tr><tr><td>misc</td><td>RAW</td><td>AB分区信息</td><td>4M</td><td>7</td></tr><tr><td>boot_a</td><td>FAT32</td><td>Image boot.scr</td><td>128M</td><td>8</td></tr><tr><td>boot_b</td><td>FAT32</td><td>Image boot.scr</td><td>128M</td><td>9</td></tr><tr><td>rootfs_a</td><td>ext4</td><td>rootfs文件系统</td><td>2G</td><td>10</td></tr><tr><td>rootfs_b</td><td>ext4</td><td>rootfs文件系统</td><td>2G</td><td>11</td></tr><tr><td>userdata</td><td>ext4</td><td>用户数据</td><td>~</td><td>12</td></tr></tbody></table>

{% hint style="warning" %}
GPT分区能划出来，但是GPT分区引导不起来镜像，SD卡上板之后不识别。
{% endhint %}

需要基于MBR分区创建扩展分区，操作方法为：

<pre class="language-bash"><code class="lang-bash"><strong># n p 1 &#x3C;enter> &#x3C;enter> +64M n e 2 &#x3C;enter> &#x3C;enter> &#x3C;enter> w &#x3C;enter>
</strong><strong>echo -e "n\np\n1\n\n+64M\nn\ne\n2\n\n\nw\n" | sudo  fdisk /dev/sdb
</strong>#echo -e "n\np\n1\n\n+$partation_size\nn\ne\n2\n\n\nw\n" | fdisk /dev/$MMC_DEIVCE_NAME

# n l &#x3C;enter> &#x3C;enter> +4M &#x3C;enter> w &#x3C;enter>
echo -e "n\nl\n\n+4M\nw\n" | sudo  fdisk /dev/sdb
#echo -e "n\nl\n\n+$partation_size\nw\n" | fdisk /dev/$MMC_DEIVCE_NAME

# n &#x3C;enter> l &#x3C;enter> &#x3C;enter> &#x3C;enter> w &#x3C;enter>
echo -e "n\nl\n\n\nw\n" | sudo  fdisk /dev/sdb
</code></pre>

* 刷写方案一：针对EMMC分区需要在板端识别并完成。（需要进RamDisk分区镜像系统启动对emmc进行分区及下载刷写）
* 刷写方案二：打包生成SD image镜像（wic镜像），使用脚本打出来完成。

{% hint style="danger" %}
扩展镜像分区必须保证第一个分区为FAT32镜像格式
{% endhint %}

## 启动引导流程

系统启动引导与分区划分密不可分，Xilinx器件使用EMMC或者SE卡启动默认会从第一个分区（FAT32）查找BOOT.BIN，其本质是在文件中校验文件头，通过dump BOOT.BIN发现，其具有标准且固定的二进制头，如下：

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

BOOT.BIN 使用 Xilinx 的 BootGen工具生成，该工具

BOOT.BIN中会打包如下固件：

* PDI文件（从硬件设计导入）
* plm.elf
* psmfw.elf
* system-default.dtb
* bl32.elf
* u-boot.elf

Soc上电第一个启动的镜像为该BOOT.BIN，该镜像中仅含有一个UBOOT，不含FSBL及SSBL两种Uboot.

Uboot启动后读取misc分区中的AB系统标志和信息，在该步骤中可以判断是否需要进行生产烧录，如EMMC中无分区表、无AB系统标志，则需要进行生产烧录，具体生产烧录流程见下节。

如能正常读取到分区信息，且能读取到AB系统标志，按照misc分区标志，从A/B分区加载Kernel和文件系统。

加载Kernel方法为使用Uboot命令行fatload从EMMC（或者SD卡）中读取Image镜像文件到DDR中，然后通过boot命令从DDR中启动kernel，使用fatload读取FAT32分区中镜像：

<pre class="language-bash"><code class="lang-bash"><strong>fatload &#x3C;interface> [&#x3C;dev[:part]> [&#x3C;addr> [&#x3C;filename> [bytes [pos]]]]]
</strong></code></pre>

* `<interface>`: 设备接口类型，例如 `mmc`（SD 卡）、`usb`（USB 设备）、`ide`（硬盘）等。
* `<dev[:part]>`: 设备号和可选的分区号。例如，`0:1` 表示设备 0 的第一个分区。
* `<addr>`: 内存地址，文件将被加载到此地址。
* `<filename>`: 要加载的文件名。
* `[bytes]`: 要加载的文件大小。如果为 0 或省略，则加载整个文件。
* `[pos]`: 文件中的起始位置。如果省略，则从文件开始处加载。

<pre class="language-bash"><code class="lang-bash">fatload mmc 0:1 0x800000 boot.bin
fatload usb 0:1 0x800000 uImage
fatload mmc 0:1 0x800000 kernel.img 0x2000 0x1000
fatload mmc 0:1 0x800000 rootfs.img 0 0x4000

<strong>Versal> fatload mmc 0:1 0x00200000 Image
</strong>22606336 bytes read in 1538 ms (14 MiB/s)

Versal> fatload mmc 0:1 0x04000000 ramdisk.cpio.gz.u-boot

Versal> fdt print /cpus

Versal> fdt addr 0x1000

Versal> booti 0x00200000 - 0x00001000

Versal> booti 0x00200000 0x04000000 0x00001000
</code></pre>

设备初次上电通过QSPI引导

### 问题：

{% hint style="info" %}
加载Kernel & 设备树 后系统启动Panic，找不到文件系统
{% endhint %}

```sh
[    2.500269] /dev/root: Can't open blockdev
[    2.504368] VFS: Cannot open root device "(null)" or unknown-block(0,0): error -6
```

分析：

```
现在使用官方的image镜像烧录内核 command line打印：root=/dev/ram0
[    0.000000] Kernel command line: console=ttyAMA0 earlycon=pl011,mmio32,0xFF000000,115200n8 clk_ignore_unused root=/dev/ram0 rw init_fatal_sh=1
```

解决方案一：

Uboot加载RAMFS到DDR，通过booti启动镜像，ramfs启动会挂载根文件系统（指定的分区）默认从ramfs启动不会导致内核panic，RAMFS的Init进程来负责挂载文件系统。

解决方案二：

Uboot bootargs 中增加关键词：rootwait 等待分区准备好后再加载分区。

## 生产烧录方案

生产需要使用生产工具烧录QSPI FLASH，FLASH烧录镜像需要打包生成。

目前能打出FLASH镜像的工具（MCS）镜像的工具只有Petalinux，Yocto中无法生成FLASH镜像文件。

打包方案待补充。

## 镜像分区打包方案

由于启动镜像烧录方式和COEX ARM不同，故打包也需要重新适配。

