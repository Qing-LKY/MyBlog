---
title: Linux 学习备忘录
date: 2022-09-27 10:28:36
tags: Linux
categories: 课程笔记
---

## 前言

因为最近装新环境的频率有点高，我发现我经常动不动就忘记指令什么的，整天百度，浪费时间，所以开个帖备查算了。

## 解压、压缩命令

| 格式 | 加压 | 解压 | 备注 |
| --- | --- | --- | --- |
| .tar | `tar cvf x.tar myDir/` | `tar xvf x.tar` | 此命令只打包不压缩。打包时也可以直接指定文件而不是目录。解包效果类似“解压到当前文件夹” |
| .tar.gz | `tar zcvf x.tar.gz myDir/` | `tar zxvf x.tar.gz` | c 是 create，x 是 extract |
| .gz | `gzip myfile` | `gzip -d x.gz` | 单个文件的压缩 |
| .xz | NULL | `xz -d a.xz` | NULL |
| .tar.xz | NULL | `tar Jxvf a.tar.gz` | NULL |
| .zip | NULL | `unzip a.zip` | 等价于 `unzip -d ./ a.zip` |

## Git

删除所有没有被 track 的文件：`git clean -f`。（查看这步操作会删掉什么：`git clean -n`）

将被 track 的文件恢复到上次 commit 的状态：`git reset --hard HEAD`。（不带 --hard 可以查看会修改什么）

将修改提交到上次 commit 里：`git commit --amend`。

fetch 之后需要手动合并：`git merge master origin/master`。

强制推送覆盖远端分支：`git push -f`。

## SSH

生成可用于 Github 的 ssh key：`ssh-keygen -t ecdsa -b 521 -C "your_email@example.com"`。

## 查找文件

使用 whereis: `whereis bash`。

使用 find: `find ~/ -name "x.txt"`。
