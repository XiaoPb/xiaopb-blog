---
title:  BearPi-HM Nano 学习记录 - 例程尝新
date: 2020-12-16 22:20:03
abbrlink: BearPi-HM-Nano-0000-0002
author: XiaoPb
img: 
coverImg: /medias/banner/7.jpg
top: true
cover: true
toc: true
mathjax: false
summary: 
  BearPi-HM Nano源码例程的编译、下载以及效果展示。
tags:
  - IOT
  - BearPi-HM Nano
  - Linux
  - git
categories:
  - BearPi-HM Nano
password:

---

# BearPi-HM Nano 学习记录 - 例程尝新

## 前言

完成了上一篇的[BearPi-HM Nano 学习记录 - 环境搭建](http://xiaopb.xyz/posts/bearpi-hm-nano-0000-0001/)学习笔记的内容，我们已经完成了编译环境的搭建。那么，接下来就该尝试编译现有的例程进行点灯测试了。那么，我们开始吧！！！

## 下载源码

因为官方提供的源码是在`Gitee`平台的仓库，其仓库地址为[BearPi-HM Nano的主代码仓库](https://gitee.com/bearpi/bearpi-hm_nano)，为了不必要的麻烦，我们可以使用`Git`直接拉取完整源码到本地。

### Git 安装/更新

在上一篇学习笔记中，我们安装好了`Ubuntu版本的Linux`，经我测试，`Ubuntu 20.04 LTS`自带有`Git客户端`，但是版本可能会比较旧了，所以我们先更新一下`Git`的版本吧，防止出现问题。

```bash
sudo apt-get remove git

sudo add-apt-repository ppa:git-core/ppa

sudo apt-get update

sudo apt-get install git
```

第一个指令出现下面文字时要输入`Y`并按回车确认，然后等待执行完成：

```bash
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  git-man libcurl3-gnutls liberror-perl
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  git ubuntu-server
0 upgraded, 0 newly installed, 2 to remove and 133 not upgraded.
After this operation, 36.6 MB disk space will be freed.
Do you want to continue? [Y/n]
```

第二个指令后出现下面文字时要按回车确认，然后等待一段时间即可：

```bash
 The most current stable version of Git for Ubuntu.

For release candidates, go to https://launchpad.net/~git-core/+archive/candidate .
 More info: https://launchpad.net/~git-core/+archive/ubuntu/ppa
Press [ENTER] to continue or Ctrl-c to cancel adding it.
```

第四个指令后出现下面文字时要输入`Y`并按回车确认，然后等待执行完成即完成更新：

```bash
2 upgraded, 0 newly installed, 0 to remove and 133 not upgraded.
Need to get 7135 kB of archives.
After this operation, 1665 kB disk space will be freed.
Do you want to continue? [Y/n] 
```

接下来我们输入指令`git --version`即可查看更新后的`Git`版本了。

```bash
xiaopb@XiaoPb:~$ git --version
git version 2.29.2
```



### 利用Git拉取源码

更新好Git之后就可以开始拉取源码了。

首先，我们先用`cd`指令跳转到我们要保存的目录，为了方便`win`下编辑代码，我们可以把代码下载到`/mnt/*`挂载的目录下，这些目录都是和`Win`共用的目录，我这里使用的是`win`下的E盘的`ubuntu`文件夹（`E:\ubuntu`）, 对应的`Linux`下路径为`/mnt/e/ubuntu`, 所以我使用的跳转指令为：`cd /mnt/e/ubuntu`，大家调到自己选择的目录即可。

然后使用指令`git clone https://gitee.com/bearpi/bearpi-hm_nano`即可拉取源码到本地目录中来，需要等待一些时间，网络好的话会比较快。

下载完即可在`Linux`和`Win10`下看得到下载下来的源码目录了：

![linux 下源码目录](https://i.loli.net/2020/12/17/pKbNFmH25fgjP3i.png)

![Win10 下源码目录](https://i.loli.net/2020/12/17/34MgvyrA2BETSzu.png)

## 编译代码

### 检查源码目录下的 build.py

首先检查一下源码目录下的`build.py`是不是软链接，检查办法，在`Win10`下查看该文件的属性，若属性如下图所示即可：

![build.py的属性](https://i.loli.net/2020/12/17/vc5KZgl1y3boiSe.png)

若**目标不是**`*\bearpi-hm_nano\build\lite\build.py`，其中`*`是上面你拉取源码的相应位置，则需要进行以下操作：

- 打开`Linux终端`，使用`cd`指令跳转到源码目录中去（例如：`cd /mnt/e/ubuntu/bearpi-hm_nano`）。

- 查看目录下是否存在`build.py`文件，`ls`。

  ```bash
  xiaopb@XiaoPb:/mnt/e/ubuntu/bearpi-hm_nano$ ls
  LICENSE       base      bundle.json  foundation  prebuilts    utils
  README.md     build     domains      kernel      test         vendor
  applications  build.py  drivers      out         third_party
  ```

	- 存在则要先删除原本的`build.py`文件，`rm -f build.py`。

- 重新产生软连接，`ln -s ./build/lite/build.py ./build.py`。

### 检查源码目录下的 .sh 文件

由于是在`gitee`上拉取的源码，并且我们是下载到`Linux`和`Win10`共用的文件夹内，到达本地的时候，可能会存在文件的换行符被修改的情况。

在源码目录下使用`vscode`打开整个目录，若没有安装`vscode`，请移步[VSCode官网](https://code.visualstudio.com/)下载安装包进行安装使用，安装教程可参考[VSCode详细安装教程](https://www.cnblogs.com/csji/p/13558221.html)。

![右键->通过Code打开](https://i.loli.net/2020/12/17/YmMwP4ZCpFBGf32.png)

打开目录树`/vendor/hisi/hi3861/hi3861`

![Bearpi-hm_nano 目录](https://i.loli.net/2020/12/17/JQlL72BHDRYr6GV.png)

分别点击 `build_patch.sh`, `build.sh`, `hm_build.sh`这三个文件，查看`vscode`的右下角是否是`LF`，若不是`LF`(可能是`CRLF`)就点击红框位置，在弹出的选择行尾序列的选择中选中`LF`，然后保存。

![文件换行标志(红框)](https://i.loli.net/2020/12/17/njDR6dsXyKB7Pum.png)

![选择行尾序列](https://i.loli.net/2020/12/17/gHEbztrj1oVUx8J.png)

- 以上的三个文件是我遇到的需要修改的`sh`文件，若是编译的时候遇到如下错误：

  ```bash
  [221/222] ACTION //vendor/hisi/hi3861/hi3861:run_wifiiot_scons(//build/lite/toolchain:linux_x86_64_riscv32_gcc)
  FAILED: obj/vendor/hisi/hi3861/hi3861/run_wifiiot_scons_build_ext_components.txt
  python ../../build/lite/build_ext_components.py --path=../../vendor/hisi/hi3861/hi3861 --command=sh\ hm_build.sh
  /mnt/e/ubuntu/bearpi-hm_nano/vendor/hisi/hi3861/hi3861/build_patch.sh: line 18: set: -: invalid option
  set: usage: set [-abefhkmnptuvxBCHP] [-o option-name] [--] [arg ...]
  Traceback (most recent call last):
    File "../../build/lite/build_ext_components.py", line 64, in <module>
      sys.exit(main())
    File "../../build/lite/build_ext_components.py", line 58, in main
      cmd_exec(args.command)
    File "../../build/lite/build_ext_components.py", line 32, in cmd_exec
      raise Exception("{} failed, return code is {}".format(cmd, ret_code))
  Exception: ['sh', 'hm_build.sh'] failed, return code is 2
  ninja: build stopped: subcommand failed.
  you can check build log in /mnt/e/ubuntu/bearpi-hm_nano/out/BearPi-HM_Nano/build.log
  /home/xiaopb/ninja/ninja -w dupbuild=warn -C /mnt/e/ubuntu/bearpi-hm_nano/out/BearPi-HM_Nano failed, return code is 1
  ```

- 可以看出错误出在`--command=sh\ hm_build.sh`，也就是`hm_build.sh`文件，则直接找到该文件进行修改行尾序列即可。

### 查看/选择开启的例程

在上面的基础下，打开目录树下的路径`applications\BearPi\BearPi-HM_Nano\sample`，点击`BUILD.gn`

然后将`"B1_basic_led_blink:led_example"`前面的`#`去掉，即可开启该例程的编译，想体验其他例程也是如此操作。

![例程目录](https://i.loli.net/2020/12/17/5OPznf9dTIklmVJ.png)

本次仅测试点灯例程，所以把其他例程全都用`#`屏蔽掉，仅将`"B1_basic_led_blink:led_example"`前面的`#`去掉，如下图所示。

![仅打开B1例程时的 BUILD.gn 文件](https://i.loli.net/2020/12/17/kz6oNgJUO9VF4pd.png)

到此，编译点灯例程前的准备完成了！那让我们开始编译吧！！！



### 编译源码

首先检查一下`Linux`终端下的操作目录是否在源码目录下，自己下载的地方应该知道吧，嘻嘻！

编译起来很简单，只需要运行一条指令即可：`python build.py BearPi-HM_Nano`

接下来就等他编译完成咯。

```bash
xiaopb@XiaoPb:/mnt/e/ubuntu/bearpi-hm_nano$ python build.py BearPi-HM_Nano

=== start build ===

Done. Made 63 targets from 56 files in 949ms
ninja: Entering directory `/mnt/e/ubuntu/bearpi-hm_nano/out/BearPi-HM_Nano'
[1/221] cross compiler obj/base/hiviewdfx/utils/lite/hiview_config.o
[2/221] cross compiler obj/base/hiviewdfx/frameworks/hilog_lite/mini/hiview_log_limit.o
···
··· 此处略过一堆输出
···
[section_count=0x1]
[section0_compress=0x1][section0_offset=0x3c0][section0_len=0x69749]
[section1_compress=0x0][section1_offset=0x0][section1_len=0x0]
-------------output/bin/Hi3861_wifiiot_app_ota.bin image info print end--------------

< ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ >
                              BUILD SUCCESS
< ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ >

See build log from: /mnt/e/ubuntu/bearpi-hm_nano/vendor/hisi/hi3861/hi3861/build/build_tmp/logs/build_kernel.log
[221/221] STAMP obj/vendor/hisi/hi3861/hi3861/run_wifiiot_scons.stamp
ohos BearPi-HM_Nano build success!
```

看到出现`BUILD SUCCESS`，那就是编译成功了。

那么让我们下载到板子上看一下运行效果吧！



## 下载固件到板子

- 首先将板子通过`Type-C`线连接至电脑，然后打开[HiBurn.exe](https://github.com/XiaoPb/HiBurn), 查看`COM`端口，如果存在多个`COM`口，需要你去确认一下哪个是连接到板子上的。

![HiBurn界面](https://i.loli.net/2020/12/17/sjxHVflXMIbQoAu.png)

- 然后点开设置，选择`Com setting`，设置波特率为`3000000`（这样下载速度比较快啦），然后点确定。

![Com setting](https://i.loli.net/2020/12/17/OzdygKVqIX72u61.png)

- 然后点击`Select file`，在`.\bearpi-hm_nano\out\BearPi-HM_Nano`下选择`_allinone.bin`做后缀的文件(`Hi3861_wifiiot_app_allinone.bin`)，然后点击打开。

- 将`Auto buru`选中。

  ![选中 Auto buru](https://i.loli.net/2020/12/17/4OYn5rsJHePkuDq.png)

- 然后点击`Connect`，板子按下`RESET`复位按键，等待下载完成后点击`Disconnect`。

  ```
  Connecting...
  Ready to load at 0x10A000
  CCStartBurn
  total size:0x3C00
  loady succ
  Entry loader
  ============================================
  
  erase flash 0x0 0x200000
  Ready for download
  CCCStartBurn
  total size:0xB4450
  
  Execution Successful
  ============================================
  
  erase flash 0x1FA000 0x6000
  Ready for download
  CCCtotal size:0x6000
  
  Execution Successful
  ============================================
  ```

- 出现三次`============================================`后即是下载成功，成功后要手动点击`Disconnect`来断开连接，断开连接前不能按板子上面的复位按键。

- 断开连接后，按下板子上的`RESET`复位按键，即可看到`LED`闪烁的效果了。

- 如果在按下复位前，打开串口助手，连接上该板子的串口，即可收到以下初始化输出信息。

  ![串口助手设置](https://i.loli.net/2020/12/17/peC9uvJGE5l8ZYU.png)

  ```bash
  ready to OS start
  sdk ver:Hi3861V100R001C00SPC025 2020-09-03 1
  8:10:00
  FileSystem mount ok.
  wifi init success!
  
  00 00:00:00 0 196 D 0/HIVIEW: hilog init success.
  00 00:00:00 0 196 D 0/HIVIEW: log limit init success.
  00 00:00:00 0 196 I 1/SAMGR: Bootstrap core services(count:3).
  00 00:00:00 0 196 I 1/SAMGR: Init service:0x4ae938 TaskPool:0xfa224
  00 00:00:00 0 196 I 1/SAMGR: Init service:0x4ae944 TaskPool:0xfa894
  00 00:00:00 0 196 I 1/SAMGR: Init service:0x4aea6c TaskPool:0xfaa54
  00 00:00:00 0 228 I 1/SAMGR: Init service 0x4ae944 <time: 0ms> success!
  00 00:00:00 0 128 I 1/SAMGR: Init service 0x4ae938 <time: 0ms> success!
  00 00:00:00 0 72 D 0/HIVIEW: hiview init success.
  00 00:00:00 0 72 I 1/SAMGR: Init service 0x4aea6c <time: 0ms> success!
  00 00:00:00 0 72 I 1/SAMGR: Initialized all core system services!
  00 00:00:00 0 128 I 1/SAMGR: Bootstrap system and application services(count:0).
  00 00:00:00 0 128 I 1/SAMGR: Initialized all system and application services!
  00 00:00:00 0 128 I 1/SAMGR: Bootstrap dynamic registered services(count:0).
  ```

  至此，例程的源码编译下载已经成功了。我们看下效果吧！

![板子效果](https://i.loli.net/2020/12/17/E4WFBeGatULTkIl.gif)