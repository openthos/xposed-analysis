Welcome to the xposed-analysis wiki!  


# 1. 软硬件环境
- 硬件配置  
    - Master Pro（MP S6-LM01）笔记本
    - CPU: Interl(R) Core(TM) i7-4720HQ CPU @ 2.60GHz
    - Mem: 16G

- 操作系统  
    -Ubuntu15.10 x64
    
# 2. Xposed分析

参考资料：
Xposed官方论坛：[http://forum.xda-developers.com/xposed](http://forum.xda-developers.com/xposed)  
Xposed的作者是rovo89，源码等都位于其github的主页内：[https://github.com/rovo89](https://github.com/rovo89?tab=repositories)  
Xposed官方说明文档：[https://github.com/rovo89/XposedBridge/wiki/Development-tutorial](https://github.com/rovo89/XposedBridge/wiki/Development-tutorial)  
Android Hook框架Xposed原理与源代码分析：[http://blog.csdn.net/wxyyxc1992/article/details/17320911](http://blog.csdn.net/wxyyxc1992/article/details/17320911)  
    
# 3. Xposed编译

## 3.1 Android AOSP 源码编译

源码使用v5.1.1_r24版本。源码的下载不再说明。这里假设您已经下载好了源码。
    
### 3.1.1 编译前的环境准备

可参考安卓源码官网相关信息：[http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html)。但此官网并没有针对Ubuntu15.10x64的说明，故按其说明并不能在Ubuntu15.10x64上正确编译android源码。下面的配置过程是经过验证，可以正确编译源码的配置。  

- 安装JDK
```
sudo apt-get update
sudo apt-get install openjdk-7-jdk
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

- 安装必要的工具包
```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip

sudo apt-get install libc6-dev libncurses5-dev x11proto-core-dev libx11-dev libreadline6-dev libgl1-mesa-dev g++-multilib schedtool tofrodos python-markdown pngquant libxml2-utils xsltproc zlib1g-dev libxext-dev gettext bc mtools lib32z1 lib32ncurses5 python-pip libyaml-dev python-dev

sudo pip install prettytable Mako pyaml dateutils --upgrade
```

### 3.1.2 编译

假设AOSP版本的安卓源码位置为[AOSP]。  

```
cd [AOSP]
source build/envsetup.sh
lunch （这里要选择某一个版本，比如eng、user等）
make -j16
```

## 3.2 Androidx86 源码编译

源码同样使用的是v5.1.1_r24版本。

### 3.2.1 编译前的环境准备

3.1.1节中的配置这里同样需要，若未配置，请按3.1.1节中的配置进行配置。  
另外还需要配置如下工具包：  

```
sudo apt-get install squashfs-tools
```

### 3.2.2 编译

假设x86版本的安卓源码位置为[x86]。  

```
cd [x86]
source build/envsetup.sh
lunch （这里要选择某一个版本，比如eng、user等）
make efi_img -j16
```

## 3.3 Xposed框架编译

通过查找资料和实际的编译实验。目前Xposed框架对x86框架的支持，只能编译32位的版本，并且只能在AOSP版本的android源码下编译通过。androidx86版本的android源码下无法编译通过。这个后面再研究，先说明在AOSP版源码下的编译方法。  

### 3.3.1 编译前的环境准备

假设AOSP版源码所在目录为[aosp]  

- XposedTools
根据第2节中提到的源码网址，下载XposedTools。假设下载后的XposedTools所在目录为[XposedTools]。  

- 安装必要的工具包

```
$perl -MCPAN -e 'install Perl::OSType'
$cpan install Config::IniFiles
$cpan install File::ReadBackwards
$cpan install File::Tail
$perl -MCPAN -e 'install Module::Install'
```

- XposedBridge.jar

根据第2节中提到的网址，下载XposedBridge的jar包。并创建编译时的输出目录，将此jar包复制到输出目录中的java目录下。  

```
mkdir /xposed-out
cd /xposed-out
mkdir java
cd java
cp [download]/XposedBridge[XXX].jar XposedBridge.jar
```

- 准备需要编译的源码

```
cd [aosp]/.repo
mkdir local_manifests
cd local_manifests
ln -s [XposedTools]/local_manifests/xposed_sdk22.xml
cd [aosp]
repo sync --force-sync
```

- 配置build.conf

修改/创建[XposedTools]目录下的build.conf文件，如下：  

```
[General]
outdir = /home/quillon/workspace/xposedou
[Build]
# Please keep the base version number and add your custom suffix
version = 65 (test)
# makeflags = -j4
# Root directories of the AOSP source tree per SDK version
[AospDir]
22 = /home/quillon/workspace/git/androidx86-5.1.1
# SDKs to be used for compiling BusyBox
# Needs https://github.com/rovo89/android_external_busybox
[BusyBox]
```

### 3.3.2 编译

```
cd [XposedTools]
./build.pl -t x86:22
```

# 4. 安装、运行

## 4.1 androidx86 + xposedx86

由于在真机（PC台式机）上，无法安装aosp版本的android系统；而androidx86的系统虽然可以安装到真机上，但是其又没有recovery模式。故这里只能分析xposed的安装包的安装过程，采用手动执行的方式。  

### 4.1.1 安装androidx86

请参考罗浩的关于androidx86的安装文档，这里不再赘述。  

### 4.1.2 安装 xposedx86

分析xposed框架的安装包，根据安装包内的flash-script.sh脚本的安装动作，手动安装xposed。  
需要pc机器上除了被测的android系统外，还要一个linux系统（由于androidx86没有recovery，只能在linux下挂载android分区，然后手动安装。进入linux系统：  
```
su
mount /dev/sda7 /androidmnt
```
然后就是根据flash_script.sh的内容，手动安装xposed相关文件等。  

### 4.1.3 运行

重新启动，进android系统。
在启动过程中卡死在显示android几个大字的界面。看命令行信息如下：  
WARNING: linker: [vdso]: unused DT entry: type 0x6ffffef5 arg 0xec  

估计是aosp和androidx86两个版本的源码还是有一定的差距，aosp版本编译出来的xposed并不能在androidx86版本的系统上运行。  






