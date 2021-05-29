---
title:  JetsonNano 开箱设置
date: 2019-10-30 12:30:03
abbrlink: JetsonNano-0000-0001
author: XiaoPb
img: 
coverImg: /medias/banner/1.jpg
top: true
cover: true
toc: true
mathjax: false
summary: 
  JetsonNano 的首次开机设置以及换国内源。
tags:
  - JetsonNano
  - linux
  - 开机设置
categories:
  - JetsonNano
password:
---

# JetsonNano 开箱设置

## 一. 刷机

​	参考官网教程和网上教程。

## 二. 换源

1. 备份`sources.list`文件，`  sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak`。
2. 打开`sources.list`文件，` sudo vim /etc/apt/sources.list`。
3. 复制下面的内容到`sources.list`，并保存退出（`Esc` -> `:wq` -> `Enter`）。

    ```powershell
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
    ```

4. 更新apt，`sudo apt-get update`。

5. 推荐进行系统全面更新一次，`sudo apt full-upgrade `。

6. 安装`pip3`，`sudo apt-get install python3-pip`.

7. 给pip3换源，`mkdir ~/.pip`->`vim ~/.pip/pip.conf`,

   ```powershell
   [global]
   timeout = 6000
   index-url = https://pypi.tuna.tsinghua.edu.cn/simple/
   trusted-host = pypi.tuna.tsinghua.edu.cn
   ```

   ( pip和pip3的操作方法相同 )

   



