---
title: VMware LVM 磁盘扩容
date: 2022-10-14 16:41:12
tags: Linux
categories: 课程笔记
---

## 前言

最近装了点东西，磁盘空间有点吃不消了。于是想着给虚拟机扩展磁盘。我使用的 VMware® Workstation 16 Pro。使用的 Linux 镜像是 ubuntu-20.04.5-live-server-amd64。

由于之前安装系统的时候没怎么细看，直接一路默认点过去了。我发现使用的似乎是 LVM。

所以鼓捣一下，扩了下容，记录下做法。

## Part 1

输入 `df -h`，可以查看文件系统磁盘空间的使用情况。我发现，我的 /dev/mapper/ubuntu--vg-ubuntu--lv 也就是挂载在 `/` 的已经满了。 用了 8 个 G。但实际上我不止给我的虚拟机分配了这么点，我分了 20 G。应该是什么地方没设置好。

当我查看硬盘状况 `sudo fdisk -l` 时，发现 /dev/sda3 有 18 个 G。也就是说其实我还有 10 个 G 没有用上。

所以去查了一下，翻到了不少博客。

首先，输入 `sudo vgdisplay`，发现果然还有很大的 Free Size。

于是，运行下面的指令扩展逻辑卷的大小：

```bash
lvextend -L +10G /dev/mapper/ubuntu--vg-ubuntu--lv
```

具体扩大几 G 取决于 Free Size 的大小，10 G 对我来说是可行的。

然后把文件系统的大小也调整一下：

```bash
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

## Part 2

后来，我发现原本分配的这 20 个 G 也不是很够用。于是我又在 VMware 里通过“虚拟机--设置--硬盘--扩展”，给它分配到了 40 G。

再次输入 `sudo fdisk /dev/sda -l` 时，我发现，它的大小已经变大，但是三个分区的大小没变，并且出现了 "GPT PMBR size mismatch (41943039 != 83886079) will be corrected by write." 的提示。

于是我直接 `sudo fdisk /dev/sda`。输入 d 删掉了原本 18 G 的分区 3，然后输出 n 重新建了分区 3，建分区的过程保持默认，它会自动帮我分配剩余的所有空间到这个分区里。在提示是否删除原有标签的时候选择了 No。

但是此时，pv 卷的大小还没有变，只是物理盘大小大了。用下面的指令就可以把 pv 卷自动调整。

```bash
# 查看自己的物理卷名字 我的叫 /dev/sda3
sudo pvdisplay
# 调整段的大小
sudo pvresize /dev/sda3
```

再次 `sudo pvdisplay`。发现 pv 卷的大小对了。

再次 `sudo vgdisplay`。发现 Free Size 又有了。

再重复 Part 1 的步骤就行了。
