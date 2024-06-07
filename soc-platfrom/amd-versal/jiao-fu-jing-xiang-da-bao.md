# 交付镜像打包

## BSP镜像及SDK镜像打包

BSP镜像及SDK镜像均使用Yocto Project生成。

BSP镜像包包含如下镜像内容：

* Uboot镜像：uboot.elf; 包含uboot.elf的启动镜像BOOT.BIN
* Kernel 镜像：Image.elf; image.ub(实际名称为fitimage.img)
* 文件系统：

BSP镜像包通过Xilinx的的plnx-deploy生成。

当前Xilinx的Image生成产物比较多，BSP将其全部生成将其交付业务。
