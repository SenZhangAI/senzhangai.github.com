---
layout: post
title: "安装 arch linux"
description: "install arch linux"
keywords: arch
category:
tags: []
---

## 前言

装一次Arch 不容易，需要查好多资料，所以记录下来以免下次又得重新找。
据大多数的信息都可以在Arch的Wiki <https://wiki.archlinux.org/index.php/Installation_guide> 上找到。

也可参考 <http://blog.csdn.net/huangfuran/article/details/73733400>

## 制作U盘启动

先修知识点： 通常主流启动方案有两种：

1. BIOS（预启动环境，引导程序） + MBR（文件系统）
2. UEFI（预启动环境，引导程序） + GPT(文件系统)

其中方案2 相对方案1 更优，所以选 UEFI + GPT 方案。详见<https://www.zhihu.com/question/28471913>

Windows下可用的制作U盘启动的程序可以选择

## 基本的系统安装

### 分区
我的理解是分区可以将文件映射到硬盘不同区域，这一方面可以有不同的文件系统格式，例如NTFS或者

首先可以通过`fdisk -l` 或者 `lsblk -f`（推荐） 或者 `parted -l` 查看分区信息。

编辑分区方案可用`fdisk`或`parted`，parted相对fdisk更方便一点，但我觉得fdisk也足够好用。

分区需要注意的是对于 UEFI + GPT 方案，需要有单独的一个分区存储引导系统，大小推荐大于550M，文件格式`EFI system`。

方法参见 [安装 archlinux 之使用 EFI/GPT](https://www.cnblogs.com/congbo/archive/2012/09/20/2688795.html)

其他分区的分区方案，参考<http://blog.csdn.net/huangxiang360729/article/details/52639673>
以及Arch Wiki <https://wiki.archlinux.org/index.php/Partitioning>
以及 <https://www.zhihu.com/question/22453727/answer/22358304>

我的方案如下(为什么1T的硬盘我要纠结这久，才分100G?)，到时看使用情况，看哪个分区吃紧。

| **dir**     | **dev**    | **size**                                                    | **format**      |
|:------------|:-----------|:------------------------------------------------------------|:----------------|
| `/boot/efi` | `dev/sdx1` | 600M（推荐大于550M）                                        | EFI系统（vfat） |
| `/`         | `dev/sdx2` | 20G(包含usr情况下要大一点，如果是服务器usr单独分区可以较小) | ext4            |
| `/home`     | `dev/sdx3` | 60G（无明确推荐，根据用户情况）                             | ext4            |
| `/var`      | `dev/sdx4` | 15G（推荐8-12GB）                                           | ext4            |

我觉得内存够大，不想要swap分区，否则2G应该ok。

### 挂载分区

先格式化`mkfs.ext4 /dev/sdx2`，然后`mount dev/sdx2 /mnt` 将sdx2挂载到根目录，`mount dev/sdx1 /mnt/boot/efi` 挂载到boot，其他类似

### 安装Arch

通过源加载文件到 `/mnt` 来安装，所以第一件事情是找一个比较快的源。

参考 <https://wiki.archlinux.org/index.php/Mirrors>

```bash
$ cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
$ # 排序
$ rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
$ # 更新源
$ pacman -Syyu
```

然后用Arch的安装脚本安装`pacstrap /mnt base`，源很快，安装非常顺序，只要一两分钟。
我除了装`base` 也装了`base-devel`，因为看到里面很多多很常用。

### 自动生成fstab

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```
### Chroot

```bash
$ arch-chroot /mnt
```
### Timezone

```bash
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ # 例如:
$ ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ hwclock --systohc
```

### 键盘与字体

`/etc/vconsole.conf` 添加 `KEYMAP=us`, 也可以不设置，默认为us

关于字体参考：
<http://www.cnblogs.com/xlmeng1988/archive/2013/01/16/locale.html>

```bash
$ vi /etc/locale.conf    # 添加一行LANG=en_US.UTF-8
$ vi /etc/locale.gen     # 把en_US.UTF-8 UTf-8,zh_CN.GBK GBK,zh_CN.UTF-8 UTF-8,zh_CN GB2312前面的注释去掉
$ locale-gen             # 更新语言环境
$ locale                 # 查看是否有问题
```

### 主机名

```bash
$ vi /etc/hostname  添加主机名 Arch
```

### 网络配置
首先改/etc/hosts文件

添加 `127.0.1.1 Arch.localdomain Arch`

然后

```bash
# 有线连接：

$ systemctl start dhcpcd    # 连接
$ systemctl enable dhcpcd   # 以后自动连接

# 无线连接：

$ pacman -S iw wpa_supplicant dialog
$ wifi-menu    # 连接
```
更多详细的网络配置参见：

参见 <https://wiki.archlinux.org/index.php/Network_configuration#Set_the_hostname>

`lspci -v` 查看Ethernet controller
其中会获得 `Kernel driver in use: haha` `Kernel modules: haha`
然后`dmesg | grep haha` 会得到dev的初始化信息，

#### 无线网卡
无线网卡为TPLINK的USB无线网，
参考 <https://wiki.archlinux.org/index.php/Wireless_network_configuration>

因为是USB网卡，不是主板PCI网卡，所以`lspci`没用，用`dmesg | grep usbcore`得知网卡为`r8188eu`，应该是Realtek的，
在 <https://wireless.wiki.kernel.org/en/users/drivers/rtl819x>
中找到其驱动为`rtl8xxxu`
但是坑爹的是Archwiki中说有问题，用第三方的
<https://github.com/lwfinger/rtl8188eu>

首先是内核版本不一致，尝试`# mkinitcpio -p linux`

make 失败，提示`No rule to make target 'modules'`
查看issue，原因是没有`kernel headers`

`sudo ip link set wxxxxxx up`, 终于安装成功

```bash
$ sudo pacman -Syu
$ sudo pacman -S linux-headers
```

### grub

```bash
# BIOS 系统：

$ pacman -S grub os-prober
$ grub-install --target=i386-pc /dev/<目标磁盘>
$ grub-mkconfig -o /boot/grub/grub.cfg

# UEFI 系统：

$ pacman -S dosfstools grub efibootmgr
$ grub-install --target=x86_64-efi --efi-directory=<EFI 分区挂载点> --bootloader-id=grub
$ grub-mkconfig -o /boot/grub/grub.cfg
```

`grub -install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --boot-directory=/boot`

注意其中`x86_64-efi`是64位，`i386-efi`是32位，
efibootmgr是生成`.efi`，GRUB安装脚本需要的启动项(stub entries),
dosfstools可能不需要。

grub-mkconfig -o /boot/grub/grub.cfg

然后 `umount \mnt && reboot` 即可

### Win10 Arch 双系统

如果是bios + MBR 方案，可以参考[这篇文章](https://www.jianshu.com/p/27ea66e1838c)，步骤基本没问题。

原理是在Win系统的引导程序中添加选项，跳到Linux的引导程序，在ArchWiki中提到单独划分一个Window可识别的分区，但如上文所示，可以通过`ntfs-3g`支持，因此不需要单独划拨一个后期用不上的分区。

注意还需要处理Windows与Linux时间不一致的问题。

## 图形界面

我尝试在grub之前安装图形界面，虽然可以成功，但鼠标用不了，这是因为安装环境中没有鼠标驱动，在新Arch系统中装了鼠标驱动没用，而安装grub，启动新系统后能生效。

首先得了解 X, X11(X11R6), Xorg，Xfree86 , gnome, kde, Xfce之类都是些什么，
简单来说 X 是一种图形协议标准，X11R6 是X协议下的当前的最重要的版本， X Protocol version 11 Release 6

Xorg Xfree86, Xnest 都是对X协议的实现，苹果的OS X操作系统也是基于X协议，并被认为是最好的实现，将其实施于系统内核，性能更优。
可以在WINDOWS上有X服务器运行，这样你可以在linux系统上运行一个X应用程序然后在另一台windows系统上显示
Xorg 是在Xfree基础上衍生的版本，因为相对Xfree的许可更自由，很多系统例如Ubuntu、gentoo等都用Xorg替换Xfree

X实现了图形显示，但没有窗口管理 WM(Windows Manager)，所以如果没有WM可以显示图形，但不能最大化最小化移动等，
常见WM包括 gnome、kde，Xfce等，但其能力范围大于WM，为桌面环境

一篇比较好的介绍链接(linux图形界面基本知识（X、X11、Xfree86、Xorg、GNOME、KDE之间的关系）)

<http://blog.csdn.net/zhangxinrun/article/details/7332049>

所以X服务器实现，比如选Xorg，桌面环境我试试Xfce4，听说很轻量。

### Xorg安装

参考：

<https://wiki.archlinux.org/index.php/Xorg>

首先安装显卡驱动，先用`lspci`查看自己是什么显卡。

然后`pacman -Ss xf86-video` 查看有哪些包可以装，
我的是AMD卡，所以`pacman -S xf86-video-ati`或者应该也可`xf86-video-amdgpu`

可能还需要 `pacman -S mesa`  (3D支持),暂时不装

然后安装xorg，
简单的直接`pacman -S xorg`都装上，但作为不折腾不舒服的人，
我决定先只装`xorg-xinit`以及`xorg-server`

然后用 `startx` 测试一下是否安装成功，提示没有xterm，正好我缺一个终端模拟器，
先装一个xterm，`pacman -S xterm`，以后看用gnome-terminal还是啥。

退出后又提示没有xclock，`pacman -S xorg-xclock`

可以进入界面后用`pkill X`退出

然后，我装了一下`xorg-twm` 这是最乞丐版的Windows manager， 相对于gnome之类的简直不能看，
不过还蛮好玩的，startx后可以移动图形了，鼠标也可以用。

### Xfce4

<https://wiki.archlinux.org/index.php/Xfce>


提示有16个extra要装，我原本想只装部分，继续折腾，但看官方貌似意思都装，就都装吧。

然后`startxfce4`即可看到进入了。

### 另一个有趣的窗口管理i3wm
TODO

#### 登陆管理

安装 Xfce4 的登陆管理器， 看有人推荐slim，lxdm，gdm

参见Wiki <https://wiki.archlinux.org/index.php/Display_manager>

其实最好是自动登陆，所以酷不酷炫也就无所谓了，不过我先试一试直接在Xinitrc中执行`exec startxfce4`

### 其他安装

```bash
$ pacman -S vim neovim  openssh git curl wget yaourt lantern
```

#### 解决乱码
firefox 打开发现中文部分字体是乱码，
按照 <https://wiki.archlinux.org/index.php/Firefox>所述 中文语言包 firefox-i18n-zh-cn也没用，猜测是字体问题
不得不说arch真是文档齐全
<https://wiki.archlinux.org/index.php/Arch_Linux_Localization_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>

安装字体 `adobe-source-han-sans-cn-fonts` 以及 `adobe-source-han-serif-cn-fonts` 后
配置字体(这一步似乎不需要，因为已经存在此链接了) `sudo ln -s /etc/fonts/conf.avail/64-language-selector-prefer.conf /etc/fonts/conf.d/64-language-selector-prefer.conf`
重建字体cache `fc-cache -fv`

即可解决firefox字体乱码问题，就是有些丑。

##### 配置系统语言环境

参考[Linux的locale, LC_ALL 和LANG](http://blog.csdn.net/nicolase/article/details/42499521)
以及[linux下locale中的各环境变量的含义](https://www.cnblogs.com/yinheyi/p/7247295.html)
还有这一篇非常的详尽[Linux中LANG,LC_ALL,local详解](http://blog.csdn.net/z4213489/article/details/7937894)

了解locale，LC_CTYPE之类是什么

配置`~/.xinitrc ~/.xprofile`, 然后启动时输入`startx` 即可执行此脚本，如果直接`startxfce4`则不会执行此脚本

`.xinitrc` 为startx命令启动时执行的脚本，而`.xprofile`为用gdm等启动器进入用户界面时执行的脚本。

```bash
export LANG=en_US.UTF-8
export LANGUAGE=zh_CN:en_US
export LC_CTYPE=zh_CN.UTF-8

exec startxfce4
```

另外，belleve设计的iosevka字体非常好，所以`yaourt -S ttf-isosevka`安装字体，如果出错，多半是locale的问题

locale命令 查看是否有问题

### 用户及密码
passwd   添加root用户的密码

#### 用户、组与权限
装Arch顺便也探索一下Linux对权限控制的管理机制，假装自己是个运维。

组有个特别的wheel组，参见 [Linux中的wheel用户是什么](http://blog.csdn.net/cbbbc/article/details/51712797)

并用sudo管理，临时提权

这里有篇文章对于sudo的权限管理讲解很细致, [sudo权限集中管理+日志审计实战 ](http://blog.51cto.com/youdong/1719639)

##### 配置wheel的权限，让不属于wheel的用户无法使用`su`

修改 /etc/pam.d/su 文件，找到`#auth required pam_wheel.so use_uid`这
一行，将行首的“#”去掉
修改 /etc/login.defs 文件，在最后一行增加“SU_WHEEL_ONLY yes”语句。

##### 添加用户

```bash
$ useradd -m -g users -s /usr/bin/bash <用户名>
$ # 该命令创建一个名为 <用户名> 的用户，指定登陆 shell 为 bash，所属主用户组 users，用户文件夹位于 /home/<用户名>。
$ passwd <用户名>   # 设置密码
```

所以执行

```bash
useradd -m -g wheel -s /usr/bin/zsh sen
passws sen
```

##### 配置sudo

`pacman -S sudo`

`visudo` 或者 编辑 `/etc/sudoers`

因为将用户作为wheel组，简单起见，只需要注释掉如下中的NOPASSWD行

```bash
# Uncomment to allow people in group wheel to run all commands
# %wheel        ALL=(ALL)       ALL
# Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
```

