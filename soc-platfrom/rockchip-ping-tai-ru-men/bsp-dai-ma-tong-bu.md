# BSP代码同步

## 现状——问题

我们对于Soc器件的开发基于原厂（Rockchip）某一release版本的Linux SDK，Rockchip的SDK对外发布使用Repo+Git的方式对外发布，客户基于SDK开发时需要注意保留rockchip的git及repo提交记录，SDK同步版本记录等，方便rockchip进行问题确认。

当前我司在进行Soc使用开发产品迭代产品种类多，迭代周期长（2年）

在此期间出现多次因厂商修复Bug而产品未及时合并导致产品出现严重问题。对于ARM Soc平台维护来讲，需要核心解决如下问题：

* 及时接收原厂的SDK同步及更新，并及时在内部进行同步更新。
* 及时接收原厂Patch并在内部主动合并同步。

### 不可控因素

因为国内厂商自身的问题，Uboot 和 Linux Kernel 喜欢 “深度定制” 并为积极向主线上提交代码，而是被动的接收主线代码的更新及同步合并（每次选择一个Release 长期支持分支进行维护）

RK会在不同的Soc器件上Release 不同的 Kernel 版本

## 同步策略

rockhip采用Google的Repo来管理SDK的发布，在内部通过linux版本+\<rkr数字>进行内部小版本的迭代。

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

SDK的大版本通过更新repo xml来进行迭代更新。

对我们来说，我们需要建立一个Soc BSP基线版本，其主要包括与Soc平台相关的部分代码。如Uboot、Linux Kernel。

对于Soc平台基线，需要作为平台产出，平台基线需要保证同步原厂所有的git提交记录及版本管理。

基线版本管理方式使用Repo工具进行管理：为了兼容与原厂SDK的同步能力。

## 同步方法&方案

由于公司制度及国内大环境因素，公司内部代码需要使用自建仓库进行管理，而非使用公共仓库，内部自建仓库出于安全策略，无法直接与rockchip的代码保持同步。

所以需要引入“中转服务器”，大概框架如下：

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

中转服务器建立原厂代码服务器和公司内部代码服务器同步桥梁。由于repo管理git仓库，需要保证每一个仓库都记录原厂Release信息的单独分支，该分支仅更新原厂合并提交代码。

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

我们内部的基线为master分支，该分支作为产品使用基线，原则如下：

* 从原厂Release分支拉出来。
* 各个产品基于此分支拉取产品分支代码，产品后几个版本需要进行代码合并到基线版本。
* 原厂Release分支定期与原厂保持同步（需要专门在中转服务器中维护）

产品上需要解决的原厂已经解决的问题，或者Patch，通过cherry-pick合入，在最后全部rebase到基线分支中。

## 如何实施同步

