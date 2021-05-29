---
title:  BearPi-HM Nano 学习记录 - 环境搭建
date: 2020-12-14 17:07:03
abbrlink: BearPi-HM-Nano-0000-0001
author: XiaoPb
img: 
coverImg: /medias/banner/8.jpg
top: true
cover: true
toc: true
mathjax: false
summary: 
  BearPi-HM Nano的Linux环境搭建过程，基于window的linux子系统。
tags:
  - IOT
  - BearPi-HM Nano
  - Linux
  - Python
  - gcc_riscv32
categories:
  - BearPi-HM Nano
password:

---

# BearPi-HM Nano 学习记录 - 环境搭建

## 购买板子获得的学习资料链接

> 1. BearPi-HM Nano的主代码仓库是：
>    https://gitee.com/bearpi/bearpi-hm_nano
>
> 2. BearPi-HM Nano的文档资料在：
>    https://gitee.com/bearpi/bearpi-hm_nano/tree/master/applications/BearPi/BearPi-HM_Nano/docs
>
> 3. BearPi-HM Nano的淘宝链接是：（19.8需要优惠券入口）
>    https://item.taobao.com/item.htm?id=633296694816
>
> 4. BearPi-HM Nano课程免费获取地址：（审核中）
>    https://www.bilibili.com/video/av245535732

## linux 环境搭建

我使用的是`Win10`的`Linux`子系统搭建编译环境，毕竟大部分同学都熟悉的就是win下进行编写代码，那么使用`WSL`也成为了我的一个比较靠前的选项。

好了，既然决定了使用`WSL`，那么是使用`WSL1`还是`WSL2`呢？下面我们来看一下这个的对照表：

| 功能                                           | WSL 1 | WSL 2 |
| :--------------------------------------------- | :---- | :---- |
| Windows 和 Linux 之间的集成                    | ✅     | ✅     |
| 启动时间短                                     | ✅     | ✅     |
| 占用的资源量少                                 | ✅     | ✅     |
| 可以与当前版本的 VMware 和 VirtualBox 一起运行 | ✅     | ✅     |
| 托管 VM                                        | ❌     | ✅     |
| 完整的 Linux 内核                              | ❌     | ✅     |
| 完全的系统调用兼容性                           | ❌     | ✅     |
| 跨 OS 文件系统的性能                           | ✅     | ❌     |

看到这个表，其实我们心里就有一点数了，为什么呢？因为`WSL2`的内核是完整的，这个首先就能避免我们走更多的坑。虽然网上对`WSL2`的资源占用比较大，但是对于目前我们的电脑来说，我个人觉得还是相对不错的，而且微软也提供有限制 `WSL2` 资源的方式。

>微软提供了限制 WSL2 资源的方式，参见[https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945](https://docs.microsoft.com/en-us/windows/wsl/release-notes#build-18945)

​		那么我们应该怎么进行`WSL2`的安装呢？这就是我接下来要讲的了。

- 升级 Windows 10 到 2004
- 启用 WSL2 并安装 Linux分发版（[Ubuntu 20.04 LTS](https://www.microsoft.com/store/apps/9n6svws3rx71)）

### 升级 Windows 10 2004

升级到 Windows 10 2004 有多种方法，最靠谱的还是从设置 - 更新和安全里进行 OTA 升级，但是 Windows 的更新是分批推送的，2004 更新有可能等到一两个月后才会出现在你的更新界面中。

另一种更快速的方法是从官方地址下载镜像升级，访问这个地址下载运行就可以升级到最新的版本 [https://www.microsoft.com/software-download/windows10](https://www.microsoft.com/zh-cn/software-download/windows10)，需要注意的是，发布初期可能 bug 含量会高一些，介意的还是再等等。

### 启用 WSL2

> 电脑需要先在BIOS开启CPU虚拟化支持（具体自行查找网站）

升级 `Windows 10` 的过程不会碰到太多问题，升级后还需要进行一些配置才可以使用 `WSL2`，首先要启用 `Windows` 子系统功能：

开启`WSL`和虚拟机平台：

方法一：

- 打开`win10`的`控制面板`

- 然后找到`程序和功能`并点击打开

- 再就是点击页面左上方的`启用或关闭Windows功能`

- 在弹出的界面中找到`适用于Linux的Windows子系统`并勾选

- 找到`虚拟机平台`并勾选

- 点击确定并重启系统

- 使用**管理员**身份打开`powershell`，用下面的命令将`wsl2`设置为默认。

  ```powershell
  wsl --set-default-version 2
  ```

方法二：

- 使用**管理员**权限打开`powershell`

- 启用`适用于 Linux 的 Windows 子系统`可选功能

  ```powershell
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```

- 启用`虚拟机平台`可选组件

  ```powershell
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```

- 重启电脑

- 重新**管理员**身份打开`powershell`，用下面的命令将`wsl2`设置为默认。

  ```powershell
  wsl --set-default-version 2
  ```

> 从 WSL 1 更新到 WSL 2 可能需要几分钟才能完成，具体取决于目标分发版的大小。 如果从 Windows 10 周年更新或创意者更新运行 WSL 1 的旧（历史）安装，可能会遇到更新错误。 按照这些说明[卸载并删除任何旧分发](https://docs.microsoft.com/zh-cn/windows/wsl/install-legacy#uninstallingremoving-the-legacy-distro)。
>
> 如果 `wsl --set-default-version` 结果为无效命令，请输入 `wsl --help`。 如果 `--set-default-version` 未列出，则表示你的 OS 不支持它，你需要更新到版本 1903（内部版本 18362）或更高版本。
>
> 运行命令后如果看到此消息：`WSL 2 requires an update to its kernel component. For information please visit https://aka.ms/wsl2kernel`。 仍需要安装 MSI Linux 内核更新包。

执行完`方法一`或者`方法二`的最后一步时，出现以下信息即是成功开启了`WSL2`：

```powershell
PS C:\WINDOWS\system32> wsl --set-default-version 2
有关与 WSL 2 的主要区别的信息，请访问 https://aka.ms/wsl2
PS C:\WINDOWS\system32>
```

### 安装 linux 分发版

> 虽然可以随意选择Linux分发版，但是我建议使用ubuntu的，网上资料的比较多。如果你熟悉其他的分发版，那也可以使用。

- 打开 [Microsoft Store](https://aka.ms/wslstore)，并选择你偏好的 Linux 分发版。

  ![Microsoft Store 中的 Linux 分发版的视图](https://i.loli.net/2020/12/14/R7KHkOe2L3BuTmc.png)

  单击以下链接会打开每个分发版的 Microsoft Store 页面：

  - [Ubuntu 16.04 LTS](https://www.microsoft.com/store/apps/9pjn388hp8c9)
  - [Ubuntu 18.04 LTS](https://www.microsoft.com/store/apps/9N9TNGVNDL3Q)
  - [Ubuntu 20.04 LTS](https://www.microsoft.com/store/apps/9n6svws3rx71)
  - [openSUSE Leap 15.1](https://www.microsoft.com/store/apps/9NJFZK00FGKV)
  - [SUSE Linux Enterprise Server 12 SP5](https://www.microsoft.com/store/apps/9MZ3D1TRP8T1)
  - [SUSE Linux Enterprise Server 15 SP1](https://www.microsoft.com/store/apps/9PN498VPMF3Z)
  - [Kali Linux](https://www.microsoft.com/store/apps/9PKR34TNCV07)
  - [Debian GNU/Linux](https://www.microsoft.com/store/apps/9MSVKQC78PK6)
  - [Fedora Remix for WSL](https://www.microsoft.com/store/apps/9n6gdm4k2hnc)
  - [Pengwin](https://www.microsoft.com/store/apps/9NV1GV1PXZ6P)
  - [Pengwin Enterprise](https://www.microsoft.com/store/apps/9N8LP0X93VCP)
  - [Alpine WSL](https://www.microsoft.com/store/apps/9p804crf0395)

- 在分发版的页面中，选择“获取”。

![Microsoft Store 中的 Linux 分发版](https://i.loli.net/2020/12/14/Z1ujYa647WRKIHk.png)

安心等待片刻，就安装好了，这时你点击开始，就可以在最近安装看见你安装好的分发版了。

首次启动新安装的 `Linux` 分发版时，将打开一个控制台窗口，系统会要求你等待一分钟或两分钟，以便文件解压缩并存储到电脑上。 未来的所有启动时间应不到一秒。完成之后会让你设置账号和密码。

至此，`WSL2`以及相应的`Linux`系统以及安装完毕了！

在进行下一步之前，需要先把安装好的系统更新一下安装包管理工具相关资源：

1. 更新包索引：`sudo apt update`
2. 升级包：`sudo apt upgrade`

### Linux 构建工具安装配置

> 以下使用的命令均在安装好的`linux`分发版的命令终端运行。

| 开发工具              | 用途              | 获取途径                                                     |
| --------------------- | ----------------- | ------------------------------------------------------------ |
| 交叉编译器gcc_riscv32 | 交叉编译工具      | http://tools.harmonyos.com/mirrors/gcc_riscv32/7.3.0/linux/gcc_riscv32-linux-7.3.0.tar.gz |                                                      |
| gn                    | 产生ninja编译脚本 | http://tools.harmonyos.com/mirrors/gn/1523/linux/gn.1523.tar |
| ninja                 | 执行ninja编译脚本 | http://tools.harmonyos.com/mirrors/ninja/1.9.0/linux/ninja.1.9.0.tar |
| Python3.7+            | 编译构建工具      | https://www.python.org/ftp/python/3.8.5/Python-3.8.5.tgz     |

| 开发工具        | 用途                 | 获取途径               |
| --------------- | -------------------- | ---------------------- |
| SCons3.0.4+     | 编译构建工具         | Linux apt命令安装/更改 |
| bash            | 命令处理器           | Linux apt命令安装/更改 |
| build-essential | 编译依赖的基础软件包 | Linux apt命令安装/更改 |

**将`Linux shell`改为`bash`**

查看`shell`是否为`bash`，在终端运行如下命令

```bash
ls -l /bin/sh
```

如果为显示为`/bin/sh -> bash`则为正常，否则请按以下方式修改：

**方法一**：在终端运行如下命令，然后选择 no。

```bash
sudo dpkg-reconfigure dash
```

**方法二**：先删除sh，再创建软链接。

```bash
rm -rf /bin/sh
sudo ln -s /bin/bash /bin/sh
```

**安装`Python`环境**

1. 输入命令“python3 --version”，查看Python版本号。需使用python3.7以上版本，否则请自行安装相应版本。

> 如果上面安装的分发版本是[Ubuntu 20.04 LTS](https://www.microsoft.com/store/apps/9n6svws3rx71)的话，其已经带了`python3.8`。
>
> **但是没有pip包管理器，需要使用 `sudo apt install python3-pip`命令安装pip3**
>
> 后面需要使用到`python`来操作**编译**，但是我们的系统并没有`python`，只有`python3`，而且我们编译环境需要用到`python3.7+`版本，所以我们把`python3.7+`版本映射一个软连接到`python`方便我们使用：
>
> - 首先使用`cd`指令跳转到`/usr/bin`, `cd /usr/bin`.
>
> - 然后使用`ls`指令查看目录下的`python3.7+`的实际名称,我这里是`python3.8`.
>
>   ```
>    fwupdtpmevlog                        python3                            wslupath
>    g++                                  python3-config                     wslusc
>    g++-9                                python3.8                          wslvar
>    gapplication                         python3.8-config                   wslview
>    gawk                                 ranlib                             www-browser
>   ```
>
> - 然后将`python3.8`映射到`python`，`sudo ln -s /usr/bin/python3.8 python`.

2. 安装`python`模块`setuptools`，运行`sudo pip3 install setuptools`
3. 运行`sudo pip3 install kconfiglib`命令，安装`GUI menuconfig`工具，建议安装`Kconfiglib 13.2.0+`版本。（需`root/sudo`权限安装）
4. 安装`pycryptodome`。运行`sudo pip3 install pycryptodome`命令
5. 安装`six`。运行`sudo pip3 install six --upgrade --ignore-installed six`
6. 安装`ecdsa`。运行`sudo pip3 install ecdsa`

**安装`Scons`**

运行命令：`sudo apt-get install scons -y`进行安装。

输入命令`scons -v`，查看是否安装成功。

**安装编译工具环境**

首先在`win10`通过上面的链接下载`gn`、`ninja`和`gcc\_riscv32`到无中文路径的文件夹下：

我这三个文件下载的路径是`win10`下的`E:\ubuntu\`，其相应的`linux`映射目录在`/mnt/e/ubuntu/`。

下载好这三个文件后，通过`linux`分发版的终端跳转到相应的目录，我这里是跳转到`/mnt/e/ubuntu/`，也就是执行命令：（请按照自己下载的目录跳转）

```bash
cd /mnt/e/ubuntu/
```

解压`gn`工具：

```bash
tar -xvf gn.1523.tar -C ~/
```

解压`ninja`工具：

```bash
tar -xvf ninja.1.9.0.tar -C ~/
```

解压`gcc_riscv32`工具：（WLAN模组类编译工具链）

```bash
tar -xvf gcc_riscv32-linux-7.3.0.tar.gz -C ~/
```

为三个工具设置环境变量：

- 打开`.bashrc`文件：

  ```bash
  sudo vim ~/.bashrc
  ```

- 添加环境变量在文件末尾并保存：

  添加下面三句到文件最末尾，然后保存文件（需要自行查看`vim`的编辑命令，防止误操作）

  ```bash
  export PATH=~/gn:$PATH
  export PATH=~/ninja:$PATH
  export PATH=~/gcc_riscv32/bin:$PATH
  ```

- 生效环境变量

  ```bash
  source ~/.bashrc
  ```

- 测试工具安装是否成功

  使用下面命令测试`gcc_riscv32`工具是否安装成功

  ```bash
  riscv32-unknown-elf-gcc -v
  出现以下内容即是成功
  Using built-in specs.
  COLLECT_GCC=riscv32-unknown-elf-gcc
  COLLECT_LTO_WRAPPER=/home/xiaopb/gcc_riscv32/bin/../libexec/gcc/riscv32-unknown-elf/7.3.0/lto-wrapper
  Target: riscv32-unknown-elf
  Configured with: ../riscv-gcc/configure --prefix=/data/d00434757/tools/riscv32 --target=riscv32-unknown-elf --with-arch=rv32imc --with-abi=ilp32 --disable-__cxa_atexit --disable-libgomp --disable-libmudflap --enable-libssp --disable-libstdcxx-pch --disable-nls --disable-shared --disable-threads --disable-multilib --enable-poison-system-directories --enable-languages=c,c++ --with-gnu-as --with-gnu-ld --with-newlib --with-system-zlib CFLAGS='-fstack-protector-strong -O2 -D_FORTIFY_SOURCE=2 -Wl,-z,relro,-z,now,-z,noexecstack -fPIE' CXXFLAGS='-fstack-protector-strong -O2 -D_FORTIFY_SOURCE=2 -Wl,-z,relro,-z,now,-z,noexecstack -fPIE' LDFLAGS=-Wl,-z,relro,-z,now,-z,noexecstack 'CXXFLAGS_FOR_TARGET=-Os -mcmodel=medlow -Wall -fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -Wtrampolines -fno-short-enums -fno-short-wchar' 'CFLAGS_FOR_TARGET=-Os -mcmodel=medlow -Wall -fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -Wtrampolines -fno-short-enums -fno-short-wchar' --with-headers=/data/d00434757/tools/riscv32/riscv32-unknown-elf/include --with-mpc=/usr/local/mpc-1.1.0 --with-gmp=/usr/local/gmp-6.1.2 --with-mpfr=/usr/local/mpfr-4.0.2
  Thread model: single
  gcc version 7.3.0 (GCC)
  ```

至此`Linux`的环境已经配置好了，那么接下来就是编译测试代码以及下载了。

芯片固件下载软件：[HiBurn.exe](https://github.com/XiaoPb/HiBurn)



## 使用的终端环境

安装后`linux`后开始菜单会有左图终端工具，右图所示的终端工具可在`Microsoft Store`下载：

![应用图标](https://i.loli.net/2020/12/14/nW9HJNs7PgUFtwB.png)

两个终端的界面如下：

![Ubuntu 20.04 LTS](https://i.loli.net/2020/12/14/LxSIEH1fzmkK429.png)

![Windows Terminal](https://i.loli.net/2020/12/14/DYm8xIqVgkotnji.png)

>参考的相关博客和文档：
>
>[面向开发者的 WSL2 安装指南](https://zhuanlan.zhihu.com/p/145488247)
>
>[比较 WSL 1 和 WSL 2](https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions)
>
>[适用于 Linux 的 Windows 子系统安装指南 (Windows 10)](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)













