---
title: 《操作系统实践》实验指南 2
date: 2022-09-20 08:59:44
tags: OS
categories: 课程笔记
---

# 文章目录

[1 环境配置与 "hello OS！"](https://qing-lky.github.io/2022/09/13/OS-LAB/)

**2 从实模式到保护模式**

# 2 从实模式到保护模式

(更新中)

## 2.1 Freedos 初体验

### 2.1.1 Why DOS？

DOS 是磁盘操作系统（Disk Operating System）的简称，是早期个人计算机上的一类操作系统。

在上一章中，课本里提到：

> 还有一个可选的方法能够帮助调试，做法也很简单，就是把 "org 07c00h" 这一行改成 "org 0100h" 就可以编译成一个 .COM 文件让它在 DOS 下运行了。

通过 DOS，可以方便地进行引导程序的执行和调试。

在之前的实验中，我们都是将引导程序直接写入引导扇区中。但是引导扇区的大小是有限的（512），随着程序的增大，问题会逐渐体现。为了解决这个问题，可以写一个“真正的”引导扇区来读取我们的程序，使我们的程序像一个真正的内核那样被运行。但这有较高的难度。

再加上：

> 很多保护模式的教程都是基于 DOS 来讲的，如果读者在本书中有些东西没有搞明白，可以同时参考其它教程。

在这一章的实验中，我们选择 bochs + Freedos 的方式来进行。 

### 2.1.2 Freedos 的正确下载姿势与解读

虽然书上只有一句轻描淡写的“从官网上下载”，不过我连续下错了两次，所以还是记录下如何获取到正确的、可以用教材中的配置方法启动的映像。

事实上，在各种渠道下下载的盘，只要配置正确，都可以在 bochs 中启动。不过，作为一个初学者，我们可以先尝试教材中介绍的方法。

下载的入口在[Bochs 官网](https://bochs.sourceforge.io/)的侧边栏 Get Bochs 下的 [Disk Image](https://bochs.sourceforge.io/diskimages.html)。我们要下的是 freedos 的映像。 

我下载下来时它的名字叫："freedos-img.tar.gz"，里面有四个文件 "a/b/c.img" 和 "bochsrc"。只有在这里下载的压缩包里面才会有课本中提到的 "a.img"。

*如果你不想关心别的事情了，那可以跳到 2.1.3 了。后面的内容只是记录我在找到这个书上指定的东西的过程中发现的一些趣事。*

首先，事实上，你也可以在其它地方找到 freedos。比如 [freedos 自己的官网](https://freedos.org/download/)，和我们[下载 bochs 源码的地方附近](https://sourceforge.net/projects/bochs/files/)。

freedos 官网上可以找到能用的 FreeDOS 1.3 Floppy Edition。根据它的 readme.txt，我们也许可以用 144m/x86boot.img 作为启动盘来装比较高版本的 freedos。~~（xygg 试过，跑起来很帅）~~不过我没有去尝试，不清楚配置的细节上有没有特殊的讲究。而且我们的实验对 freedos 的版本其实没有多大的讲究。

至于 sourceforge.net 上提供的那个，则是个 hard disk，配置起来和书上提供的有很大出入（当然它有给出示例 bochsrc，所以你想跑那个也不是不行）。

在 bochs.sourceforge.io 上下载的这个，虽然描述说的是 "10-meg hard disk image which boots into FreeDOS"，但其实这个 hard disk image 指的是压缩包里的 c.img，另外两个软盘 a.img 和 b.img 虽然在示例中没有作为启动盘使用，但它们也是启动盘。

我直接把 `boot: c` 改成 a 和 b 试了一下。a.img 是可以正常且完整启动的。b.img 可以看得出进入了引导程序，但是内核或 FAT 的加载似乎出现了什么问题。我暂时没有去细究这个 b。但是书本上教的把 a.img 复制出来改成 freedos.img 当启动盘用确实是可行的。

P.S. 事实上，通过下面的指令，我们也可以看出它是一个启动盘：

```sh
hexdump -C a.img | head -50
```

你可以在其中 0200 前（也就是 511 和 512 字节处）观察到 55 和 aa。结合上节课的知识，这个 a.img 会被识别为启动盘。

事实上，我们也可以考虑把 a.img 和 b.img 的前 512 个字节搞出来反汇编一下观察下差别，来满足我们的好奇心。这里暂时留个坑罢。

### 2.1.3 在 Bochs 中运行 Freedos

前面我们下载到了 freedos 映像的压缩文件。把其中的 a.img 弄出来作为启动盘，然后创建一个新的空白软盘 pm.img 来存放我们希望 guest os 能访问的文件。

为方便辨认，我把 a.img 改名成了 freedos.img。

```sh
tar -zxvf freedos-img.tar.gz
cp freedos-img/a.img ./freedos.img
bximage -func=create -fd=1.44M -q pm.img
vim bochsrc
```

把 bochsrc 修改为下面的内容：（其实就是上章用的那些修改了一下）

```sh
###############################################################
# Configuration file for Bochs
###############################################################

# how much memory the emulated machine will have
megs: 32

# filename of ROM images
romimage: file=/usr/local/share/bochs/BIOS-bochs-latest
vgaromimage: file=/usr/share/vgabios/vgabios.bin

# what disk images will be used
floppya: 1_44=freedos.img, status=inserted
floppyb: 1_44=pm.img, status=inserted

# choose the boot disk.
boot: a

# where do we send log messages?
# log: bochsout.txt

# disable the mouse
mouse: enabled=0

# enable key mapping, using US layout as default.
# keyboard_mapping: enabled=1, map=/usr/share/bochs/keymaps/x11-pc-us.map

display_library: sdl
```

接下来，运行 bochs，成功启动的话可以看到 freedos 的命令行界面。

{% asset_img after_boot.png %}

那个 format 是我自己打进去的，为了确认键盘能不能用。

如果你遇到的报错是 no boot device，请检查你的启动盘有没有设置对。

不过也有非常多人遇到了进入 bochs 后 freedos 内键盘无法使用的问题，这个我也不清楚怎么解决。（指的是完全打不了字或一打字 bochs 就报错。我们后面用的那个示例 .com 是个死循环，运行后是关不掉也用不了键盘的，只能关掉 bochs）

### 2.1.4 软盘格式化与挂载

在 freedos 下，你可以运行下面的指令来格式化 B 盘（也就是 pm.img）。

```bat
format B:\
```

不同于前面直接将二进制代码写入扇区，在 dos 下运行 .com 需要借助 freedos 的文件系统，所以我们需要先把 pm.img 格式化。如果不这么做的话，在 mount 时会因为无法识别文件系统类型而报错。

其实，你也可以不用到 freedos 下进行格式化，直接在 linux 下执行下面的指令也是可以的。

```sh
mkfs.fat pm.img
```

被上述指令格式化的软盘是可以被 freedos 使用的。

P.S. 格式化指根据用户选定的文件系统（如FAT12、FAT16、FAT32、NTFS、EXT2、EXT3等），在磁盘的特定区域写入特定数据，以达到初始化磁盘或磁盘分区、清除原磁盘或磁盘分区中所有文件的一个操作。一个什么都没有的空白软盘和格式化后的软盘是不一样的。

使用下面指令可以挂载，和取消挂载。

```sh
# 挂载前需要新建文件夹
sudo mkdir /mnt/floppy
sudo mount -o loop pm.img /mnt/floppy
# 可以像正常文件系统那样访问读写这个文件夹
sudo cp a.txt /mnt/floppy
# 取消挂载
sudo umount /mnt/floppy
```

P.S. 什么是 loop device？它是一种“伪设备”。我们的 pm.img 上虽然有一个文件系统，但它毕竟是虚拟的，不是一个真正的硬件，不能直接访问。所以我们可以用 -o loop（也可以指定具体的 loop device），这样 pm.img 就会与这个伪设备关联，然后我们就得到了一个可挂载的设备。这样我们就可以通过挂载和访问这个设备来访问这个虚拟的文件系统了。

（上面的这段参考了[这篇 CSDN 博客](https://blog.csdn.net/weixin_30832351/article/details/98198529)和[维基百科](https://zh.wikipedia.org/wiki/Loop%E8%AE%BE%E5%A4%87)）

不管你是不是挂载到 /mnt/xxx 下，被你挂载的文件夹都会被加上权限限制（毕竟是在访问一个设备），所以 mount 以及对文件系统的操作都需要在 root 或 sudo 下进行。

书上的做法是在 cp 完 .com 文件后取消挂载。其实不取消也行，没必要每次都挂载一下。

### 2.1.5 运行随书示例