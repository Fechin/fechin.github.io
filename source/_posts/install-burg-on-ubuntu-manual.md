---
title: Ubuntu 12.10 手动安装 Burg 引导
date: 2012-12-16 12:46:30
categories:
    - Linux
tags:
    - Burg
    - 引导
    - Ubuntu
mp3: https://link.hhtjim.com/163/3374005.mp3
cover: /static/images/cover/install-burg-on-ubuntu-manual.jpeg
---

一个月前在本本上装过 Burg，也是之前唯一一次在 Ubuntu12.10 安装成功，当时写了一篇文章叫 [Ubuntu Burg 引导之打开电脑的那一刻](https://fech.in/2012/install-burg-on-ubuntu/)，之后多次重装，再也没有成功过，更新源错误提示

> 无法下载“W: 无法下载 http://ppa.launchpad.net/n-muench/burg/ubuntu/dists/quantal/main/source/Sources 404 Not Found”

或是用 `sudo apt-get install burg burg-themes burg-emu` 安装时提示各种找不到。burg 那个性化的设置实在迷人，今天又开始折腾了，百度 Google 找教程无果，看来只能靠自己了。有时候为了实现我们想要的效果就要不择手段，生命不息，折腾不止！既然无法自动更新，那我手动。

### 开始安装
从 Burg 的源站下载需要的文件，我用的 Ubuntu64 位系统，需要下面这些文件
> burg-common_1.98+20100623-1_amd64
> burg-emu_1.98+20100623-1_amd64
> burg-pc_1.98+20100623-1_amd64
> burg-themes-common_1.98+20100623-1_all
> burg-themes_1.98+20100623-1_all
> burg_1.98+20100623-1_amd64

记住了，按顺序执行
```
sudo dpkg -i '/home/thinkcu/Downloads/burg-common_1.98+20100623-1_amd64.deb'
sudo dpkg -i '/home/thinkcu/Downloads/burg-emu_1.98+20100623-1_amd64.deb'
安装 burg 的时有一个跟 libsdl1.2debian 的依赖关系，直接在终端修复依赖关系：
sudo apt-get -f instal
```
![enter image description here](/static/images/install-burg-on-ubuntu-manual1.jpg)

安装第二步， 执行 `sudo dpkg -i '/home/thinkcu/Downloads/burg-pc_1.98+20100623-1_amd64.deb'` ，会弹出软件包设置的对话框，统统默认，按 Tab 健选定确定，回车即可！
![enter image description here](/static/images/install-burg-on-ubuntu-manual2.jpg)
![enter image description here](/static/images/install-burg-on-ubuntu-manual3.jpg)

安装第三步，同样按顺序执行
```
sudo dpkg -i '/home/thinkcu/Downloads/burg-themes-common_1.98+20100623-1_all.deb'
sudo dpkg -i '/home/thinkcu/Downloads/burg-themes_1.98+20100623-1_all.deb'
sudo dpkg -i '/home/thinkcu/Downloads/burg_1.98+20100623-1_amd64.deb'
```
最后执行 `sudo burg-install /dev/sda` 或者 `sudo burg-install "(hd0)"` 设置手动安装 Burg 的 mbr（主引导记录）

![enter image description here](/static/images/install-burg-on-ubuntu-manual4.jpg)

至此，大功告成，接下来可以用 `burg-emu` 查看当前配置，设置主题可以看前一个教程！刚开始我是在自己电脑上安装的，安装成功之后再给同学安装了一个，所有图片从同学小张那里截取的。
