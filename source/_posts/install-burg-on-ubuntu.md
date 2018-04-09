---
title: Ubuntu Burg 引导之打开电脑的那一刻
date: 2012-11-14 12:19:20
tags:
mp3: http://link.hhtjim.com/163/33162226.mp3
cover: http://odwjyz4z6.bkt.clouddn.com/005.jpg
---
![enter image description here](http://odwjyz4z6.bkt.clouddn.com/burg-preview.jpg)

[Burg](https://code.google.com/archive/p/burg/downloads) 是一款帮助你在开机时选择操作系统的精简版应用程序，具有一个高度可扩展的菜单系统，可选择文本和图形模式。简而言之，Burg 可广泛定制，有很多免费的 Burg 主题。软件支持各类操作系统，如 Linux、Windows、OSX、Solaris、FreeBSD 等。如果你装有多个操作系统，开机选择系统的同时能根据自定义主题展示，Burg 一定是你不错的选择。下面我在 Ubuntu12.10 下是如何安装 Burg 的，上图便是预览效果。

### 安装 Burg

添加 Burg 源 (vim  /etc/apt/sources.list)
```
deb http://ppa.launchpad.net/bean123ch/burg/ubuntu karmic main
deb-src http://ppa.launchpad.net/bean123ch/burg/ubuntu karmic main
```
更新软件仓库
```
sudo apt-get update
```
导入证书
```
gpg --keyserver subkeys.pgp.net --recv 55708F1EE06803C5
gpg --export --armor 55708F1EE06803C5 | sudo apt-key add -
```
安装 burg 和 burg-themes（推荐用新立德搜索”Burg”并安装，安装过程中会弹出选择框默认即可）
```
sudo apt-get install burg burg-themes burg-emu
```
这样，安装就完全结束了。可以用下面这条命令预览效果，预览状态下不能设置，重启进入启动项后可以按 `F2` 更换主题，`F3` 修改分辨率，更多快捷键请查看帮助。
```
burg-emu
```
如果到这里你还没安装成功，不妨试试[手动安装 Burg](http://fech.in/2012/install-burg-on-ubuntu-manual/)。

### 安装主题

默认的主题除了够用外，没有给人眼前一亮的效果，所以选择一款喜欢喜欢喜欢的主题才能凸显 Burg 的价值，个人比较喜欢 Fortune-BURG 也是我一直在用的一款主题。开始安装，解压主题文件到 `/boot/burg/themes` 目录下，执行下面的 `sudo update-burg` 重新配置 Burg。OK，这个时候就可以在终端输入 `burg-emu` 进行预览，如果希望今后使用一直使用这个主题需要重启设置主题为 Fortune。

在安装过程中，如果出现错误，可以试试手动安装 Burg 的 mbr（主引导记录），执行 `sudo burg-install "(hd0)"` 即可。或执行 `sudo dpkg-reconfigure burg-pc` 重新配置 Burg

以后启动项如果有变动，比如装了新内核删了旧内核，加入新主题等，需执行 `sudo update-burg` 更新 Burg 引导。
