---
layout: post
title: Mac OS X 开启原生自带虚拟内存盘（Ramdisk）
tag: OSX
date: 2017-04-26 10:26:00.000000000 +10:00
---

虚拟内存盘是通过软件将一部分内存（RAM）模拟为硬盘来使用的一种技术。

由于内存有高达数 GB 每秒的速度，模拟成硬盘在适当情景下使用，会极大的增强系统性能，并且起到保护硬盘和隐私的作用。

Mac OS X 是 Unix 类型系统，原生就支持用命令行创建Ramdisk。所以可以省去了买 iRamdisk、tmpDisk 这类鸡肋软件。

如果细心按照本教程一步步的模仿，那么很容易就可以创建一个开机自动创建的 Ramdisk

1、 打开 Dashbord，找到实用工具中的脚本编辑器（10.10中叫这个名字）。输入以下内容：

```
do shell script "
if ! test -e /Volumes/\"Ramdisk\" ; then
diskutil erasevolume HFS+ \"RamDisk\" `hdiutil attach -nomount ram://2097152`
fi
mkdir /Volumes/Ramdisk/TempDownloads
mkdir -p /Volumes/Ramdisk/Library/Developer/Xcode/DerivedData
mkdir -p /Volumes/Ramdisk/Library/Developer/CoreSimulator/Devices
mkdir -p /Volumes/Ramdisk/Library/Caches/Google
mkdir -p /Volumes/Ramdisk/Library/Caches/com.apple.Safari/fsCachedData
mkdir -p /Volumes/Ramdisk/Library/Caches/com.netease.163music
mkdir -p /Volumes/Ramdisk/Library/Caches/Firefox
"
```

可以创建一个1GB 的虚拟内存盘。其中数字2097152 =1024*1024*1GB*2（最后的那个2是必须乘上的）。实际创建出来的不是1GB 整数，而是1.07GB 左右。可能是系统的对磁盘大小计算方法问题，但是这个公式肯定是正确的。（如果是2GB 那么就是1024*1024*2GB*2=4194304，把这个数字体换上即可。）

2、 将上述脚本保存为 app 格式，即可成为一个可执行文件。

3、 打开系统与偏好设置-用户和群组-登录项选项卡

4、 点击左下角小锁头解锁后按下加号加入刚刚我们编写的脚本程序，
即可实现脚本的开机自动启动，并且创建一个1GB 的虚拟磁盘。

这个脚本除了创建磁盘，还在磁盘中创建了几个文件夹：

根目录有TempDownloads和一个 Library 文件夹。
Library 文件夹下有 Caches 文件夹和 Developer 文件夹。（Caches 放 Chrome 和 Safari 的浏览器缓存，Developer 放 Xcode 的临时编译空间文件）
5、 接下来把浏览器缓存和 Xcode 临时编译空间在内存盘上创建一个替身（就是一个软链接，类似 Windows 的快捷方式）。

首先确保退出 Safari、Chrome、Xcode 这三个程序，然后再终端输入（一条一条输入）

```
rm -rf ~/Library/Caches/Google
rm -rf ~/Library/Caches/com.apple.Safari
rm -rf ~/Library/Developer/Xcode/DerivedData
```

上面三条命令删除了磁盘中三个程序的临时缓存文件夹，放心，100%安全。只要你打开程序，这三个文件夹还会自动生成。

6、 在执行下面三个命令前，必须把我们的脚本执行起来，在桌面上或者 finder 中能打开我们的内存盘，看到里面有我们创建的目录，才能正常执行下面的程序：

```
ln -s /Volumes/Ramdisk/Library/Caches/Google ~/Library/Caches/Google
ln -s /Volumes/Ramdisk/Library/Caches/com.apple.Safari~/Library/Caches/com.apple.Safari
ln -s /Volumes/Ramdisk/Library/Developer/Xcode/ ~/Library/Developer/Xcode/DerivedData
```

7、 这样，打开 Safari 或者 Chrome或者 Xcode 依次测试，就可以看到在内存盘的相关目录中产生了他们的缓存文件。同时如果我们打开这几个程序原本的目录，会发现他们会自动跳转到内存盘的相应目录下，在 SSD 中不再会产生任何垃圾。于此大功告成。以上所有命令都只需要输入一次，以后开机自动可用。

同时建议关闭休眠模式，Mac 默认的休眠模式是3，是一种混合休眠模式，在合盖保存工作到内存时，同时为了防止断电工作丢失，也保存一份到本地磁盘。对于笔记本来说，这个模式实属多余，电量不足还是选择关机吧。模式3意味着将要把内存中的工作保存到磁盘的休眠镜像文件中，这又会造成大量读写。

为了兼容起见，建议直接把模式改为0，就是只启用睡眠模式。（关闭盖子工作只保存到内存），关闭休眠：

```
sudo pmset -a hibernatemode 0
```

为了防止该文件再重启后重新生成。（不建议删除，没必要）

```
sudo touch /private/var/vm/sleepimage 
sudo chmod 000 /private/var/vm/sleepimage
```

其他相关命令：
pmset -g | grep hibernate （查看当前的hibernate模式）
ls -lh /private/var/vm/sleepimage （查看sleepimage文件大小）
今后如果需要打开hibernate模式，再将该值设为默认的就可以了：

sudo pmset -a hibernatemode 3 （设置hibernatemode为默认值3）
sudo rm /private/var/vm/sleepimage
几个小提示和说明：

Xcode 编译程序会生成大量的中间文件，一般数百兆甚至更多，放到内存盘中很有必要。

如果要还原三个程序的原本缓存位置，只需再次输入那三条 rm -rf 开头的命令即可。

如果内存盘满了，可以手动清理，或者重启后自动清空。

不建议把整个~/Library/Caches/都链接到内存盘中，因为有一个 com.apple.helped文件夹，只要使用 Xcode 之类的程序，会自动下载帮助文档多达数百兆。每次清理后都会重新下载生成，会严重消耗资源。倒不如让他在硬盘里躺着。

除了这三个程序，其他程序的缓存一般都很小，用不着管他。清除了反而影响程序的启动加载时间。只需要每隔半个月用 cleanMyMac 清理一次即可。（放过那个com.apple.helped，那个文件只要大的不是很夸张多达数 GB，就半年清理一次吧）。

链接到 Ramdisk 的目录，清理软件是不会帮忙清理的，只能手动清理，或者重启自动丢失。