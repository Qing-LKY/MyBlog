---
title: 如何在 Linux 下嗯造一个实验用的磁盘映像
date: 2022-09-28 18:11:46
tags: 
- FAT32
- 软件安全
categories: 课程笔记
---

## 前言

事情是这样的。我们软安有个实验，需要一个 FAT32 的磁盘或 U 盘来研究文件系统和磁盘的结构。

但是非常尴尬的是，我压根没有这样的条件。我的两个盘都是 NTFS 的，手头也没有 U 盘。

所以我决定在 Linux 下，使用虚拟磁盘映像来做这个实验。

下面我将会嗯造一个实验用的磁盘映像。为了达到研究的目的，它会有分区，有 FAT32 文件系统，并且各个分区会有一些各种各样的文件。

## 创建虚拟磁盘

其实就是需要个满足要求大小的文件。

### 方法一：使用 bximage

在 OS 实验中应该用过这个玩意的。fd 还是 hd 都无所谓，没指定 type 就是默认 flat，出来的就会是 128M 的“空”文件。

```bash
sudo apt install bximage
bximage -func=create -fd=128M -q myDisk.img
```

### 方法二：使用 dd 和 /dev/zero

同样也是 OS 实验用过的东西。只不过这次的输入来源是 /dev/zero 这个特殊的设备。它会不停的输出 0 (NULL)。

```bash
# 下面的指令会往 myDisk.img 里填充 128Mb 的 0
dd if=/dev/zero of=myDisk.img bs=1M count=128
```

## 磁盘分区

此时的磁盘还没有分区，我们需要对它进行分区。

```bash
sudo fdisk myDisk.img
```

执行上面的指令后，会进入 fdisk 提供的交互界面。它的使用大致如下：

- 输入 m 可以查看所有能用的指令。
- 输入 l 可以查看所有支持的文件类型的 16 进制码。可以看到我们需要的 FAT32 类型码是 b。
- 输入 p 可以查看当前已有的分区状况。现在还没有任何分区。
- 输入 n 创建一个新的分区。进入分区创建的交互：
  输入 p/e 来创建主分区或扩展分区。这里我们输入 p。
  输入一个数字作为分区号。这里我们保持默认（分区 1）。
  输入一个数字表示第一扇区的起始扇区。这里我们保持默认。
  输入一个数字来表示第一扇区的最后扇区。这里我们输入一个比较靠中间的数（因为我想多造点分区）。
  此时我们就造出了一个分区。分区的默认类型是 linux。
  重复上述过程，多造一两个分区出来。
- 输入 t 来修改分区的类型。将创建的分区修改位 FAT32（b）类型。具体操作跟着交互提示走，此处不再赘述。
- 输入 O 可以导出我们当前的配置。输入 L 可以载入一个配置。
- 输入 w 来将修改写入虚拟硬盘。并退出 fdisk。

简单的规划一下：我们的盘是 128 Mb。注意 FAT32 对簇的数量是有要求的。它要求至少要有 65525 个簇。我们的扇区大小是 512 b。大致算一下，一个分区至少要有 32 Mb。

（我暂时还没有理解 65525 的逻辑所在。[情报来源是这个](https://www.keil.com/pack/doc/mw/FileSystem/html/fat_fs.html)。）

你可以把已经算好的配置用 O 导出了。你可以这样做：

```bash
# 如果你忘了用 O。你也可以这样导出：
sudo sfdisk -d myDisk.img > disksrc
# 这样可以用之前的分区配置来分区：
sudo sfdisk myDisk.img < disksrc
```

下面是我的 diskrc，如果不想手动分区，也可以直接用我的导入：

```text
label: dos
label-id: 0x50380754
device: myDisk.img
unit: sectors

myDisk.img1 : start=        2048, size=       67953, type=b
myDisk.img2 : start=       71680, size=       70001, type=b
myDisk.img3 : start=      143360, size=      118784, type=b
```

分区我是随便加的。如果你希望多分几个区甚至扩展分区来研究，最好把磁盘大小拉大一点，比如拉到 256 Mb。

需要注意，在 fdisk 下用 O 指令导出的 disksrc 是只读的。如果需要改动，需要使用 `chmod +w disksrc` 来添加可写权限。使用 `sfdisk -d` 则不会，因为它只负责导出到标准输出。

## 使用 loop 设备

我们的分区还没有被格式化。我们需要对每个分区分别格式化，而不是对整个磁盘格式化。所以我们需要建立 loop 设备到分区的映射，单独对每个分区进行格式化。

这里有几篇参考的文章，其中这篇问答含有完整的三种做法：[Mount single partition from image of entire disk device](https://askubuntu.com/questions/69363/mount-single-partition-from-image-of-entire-disk-device)。

我的 Ubuntu 版本较高，可以使用最简单的方法二（需要 16.05 +）：

```bash
# 使用这个指令后，myDisk 的各个分区会被映射到 /dev/loop* 上。例如 /dev/loop3p1：
sudo losetup -Pf myDisk.img
# 使用下面的指令可以取消 loop3 的映射（所有的分区）：
sudo losetup -d /dev/loop3
```

其它两种方法，有需要的读者可自行到上面的链接中阅读。方法一是手动计算偏移。方法三是使用了一个工具。

## 分区格式化

上面我们已经将各个分区弄到了 loop 设备上，接下来我们可以进行格式化了：

```bash
mkfs.fat -F32 /dev/loop3p1
```

其它分区的格式化方法类比一下即可。

## 分区挂载和加入文件

需要先 losetup。如果你没有 losetup 的话，参考前面。

使用 mount 把 /dev/loop3p1 之类的挂载到某个目录上就行了。例如：

```bash
# 挂载
sudo mount /dev/loop3p1 /mnt/myhd
# 取消
sudo umount /dev/loop3p1
```

需要注意你可能不止要 `umount`，还需要 `losetup -d`。取决于你的个人需要。

## 信息检查

你可以用下面的指令来查看你挂上去的这个磁盘的相关信息：

```bash
# 下面的指令可以查看挂载点的信息：
mount
mount | grep "/mnt/myhd"
# 下面的指令可以查看：
# 使用了哪个 loop 设备、文件系统类型、磁盘大小、已使用大小、挂载位置
df -hT
df -hT | grep "/mnt/myhd"
# 下面的指令可以查看更具体的磁盘、设备信息：
sudo fdisk -l
```

## 懒人总结
