---
layout: post
title: "macOS 使用技巧"
description: "some tips for macOS"
keywords: macOS
category: Programming
tags: [tips, macOS]
---

### 支持ntfs文件系统

印象中因为版权原因，macOS不支持ntfs系统，但是其实只需要打开其隐藏的支持选项即可，

参见 [Enable NTFS File system in Mac OS Mojave](https://devstudioonline.com/article/enable-ntfs-file-system-in-mac-os-mojave)

#### 步骤1 安装brew
略

#### 步骤2 安装FUSE以及ntfs-3g

```sh
brew cask install osxfuse
brew install ntfs-3g
```

#### 步骤3 关闭系统保护

1. 重启macOS
2. 按住 `Command + R`不放，进入Recovery console window
3. 在顶部菜单中找到`Utilities->Terminal`
4. 在terminal中输入如下命令并运行

```sh
csrutil disable
```

#### 步骤4 激活ntfs驱动

```sh
sudo mv /sbin/mount_ntfs /sbin/mount_ntfs.original
sudo ln -s /usr/local/sbin/mount_ntfs /sbin/mount_ntfs
```

如果是当前最新的macOS Catalina，则可以遇到read-only files的问题，
需要额外设置

1. sudo打开`/etc/fstab`
2. 输入`LABEL=YOUR_VOLUME_NAME none ntfs rw,auto,nobrowse`，其中`YOUR_VOLUME_NAME`是volume名字
3. Finder可能找不到volume，则`Finder->Go->Go to Folder`，输入`/volume`

#### 步骤5 开启系统保护

和步骤3基本相同，除了第4步为

```
csrutil enable
```

### 设置app不显示在dock中

某些程序不够完美，比如某vpn，打开之后就不会再频繁使用，却占用宝贵的dock空间，需要设置隐藏

参见 <https://apple.stackexchange.com/questions/207939/how-to-hide-a-specific-active-app-on-os-x-has-to-be-reversible>

隐藏图标

```sh
/usr/libexec/PlistBuddy -c 'Add :LSUIElement bool true' /Applications/[AppName].app/Contents/Info.plist
```

重新显示app
```sh
/usr/libexec/PlistBuddy -c 'Delete :LSUIElement' /Applications/[AppName].app/Contents/Info.plist
```
