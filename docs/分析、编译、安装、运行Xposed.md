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

```sh
$cpan install Perl::OSType
$cpan install Config::IniFiles
$cpan install File::ReadBackwards
$cpan install File::Tail
$cpan install Module::Install
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

# 5. 成功的运行Xposed

目前仅在下面说的版本中运行成功，其他的还没验证，不保证没问题。  

# 5.1 android-arm-5.1.1

利用前面介绍的编译方法，编译出一个基于aosp的arm版本的安卓系统。  

# 5.2 Xposed相关资源

这里没有进行编译，先直接使用xda论坛下载的资源做的实验（估计按前面的办法编译出来的应该一样）。另外，网上下载了一个Flat Style Colored Bars的Xposed模块，用来验证Xposed的有效性。  

# 5.3 emulator

如何安装android studio、sdk、ndk、创建avd、使用emulator等，以及配置相关的环境变量，这里就不详细描述了。请参考相应的资料。  

# 5.4 过程

假设xposed框架所在目录为/xposed/xposed-v79-sdk22-arm/（网上下载的zip包解压所得）。  
假设编译android-arm-5.1.1产生的system.img、userdata.img、ramdisk.img复制到了/tmp/xposedtest/android-arm32/目录。  

- 启动模拟器的方法

```
cd /tmp/xposedtest/android-arm32/
emulator -avd arm32 -system system.img -data userdata.img -ramdisk ramdisk.img
```

其中的arm32是我之前创建的avd。  

- root

按照[手动root emulator](http://blog.csdn.net/feifei454498130/article/details/6537274)中的方法，对emulator的系统进行root。

- 安装XposedInstaller

启动虚拟机，在另外一个shell里面执行：  

```
adb install [xposedinstaller].apk
```

- 安装Xposed框架

关闭emulator。  

 - 修改Xposed框架的安装脚本
 
```
cd /xposed/xposed-v79-sdk22-arm/
cp ./META-INF/com/google/android/flash_script.sh .
chmod 777 flash_script.sh
gedit flash_script.sh
```
修改这个文件，把其中的一些什么版本判断之类的语句删除，只保留文件拷贝，文件属性设置等相关的内容，供后面手动安装xposed框架使用。修改后的文件大致如下：  

```
grep_prop() {
  REGEX="s/^$1=//p"
  shift
  FILES=$@
  if [ -z "$FILES" ]; then
    FILES='/system/build.prop'
  fi
  cat $FILES 2>/dev/null | sed -n $REGEX | head -n 1
}

android_version() {
  case $1 in
    15) echo '4.0 / SDK'$1;;
    16) echo '4.1 / SDK'$1;;
    17) echo '4.2 / SDK'$1;;
    18) echo '4.3 / SDK'$1;;
    19) echo '4.4 / SDK'$1;;
    21) echo '5.0 / SDK'$1;;
    22) echo '5.1 / SDK'$1;;
    23) echo '6.0 / SDK'$1;;
    *)  echo 'SDK'$1;;
  esac
}

cp_perm() {
  cp -f $1 $2 || exit 1
  set_perm $2 $3 $4 $5 $6
}

set_perm() {
  chown $2:$3 $1 || exit 1
  chmod $4 $1 || exit 1
  if [ "$5" ]; then
    chcon $5 $1 2>/dev/null
  else
    chcon 'u:object_r:system_file:s0' $1 2>/dev/null
  fi
}

install_nobackup() {
  cp_perm ./$1 $1 $2 $3 $4 $5
}

install_and_link() {
  TARGET=$1
  XPOSED="${1}_xposed"
  BACKUP="${1}_original"
  if [ ! -f ./$XPOSED ]; then
    return
  fi
  cp_perm ./$XPOSED $XPOSED $2 $3 $4 $5
  if [ ! -f $BACKUP ]; then
    mv $TARGET $BACKUP || exit 1
    ln -s $XPOSED $TARGET || exit 1
    chcon -h 'u:object_r:system_file:s0' $TARGET 2>/dev/null
  fi
}

install_overwrite() {
  TARGET=$1
  if [ ! -f ./$TARGET ]; then
    return
  fi
  BACKUP="${1}.orig"
  NO_ORIG="${1}.no_orig"
  if [ ! -f $TARGET ]; then
    touch $NO_ORIG || exit 1
    set_perm $NO_ORIG 0 0 600
  elif [ -f $BACKUP ]; then
    rm -f $TARGET
    gzip $BACKUP || exit 1
    set_perm "${BACKUP}.gz" 0 0 600
  elif [ ! -f "${BACKUP}.gz" -a ! -f $NO_ORIG ]; then
    mv $TARGET $BACKUP || exit 1
    gzip $BACKUP || exit 1
    set_perm "${BACKUP}.gz" 0 0 600
  fi
  cp_perm ./$TARGET $TARGET $2 $3 $4 $5
}

echo "******************************"
echo "Xposed framework installer zip"
echo "******************************"


echo "- Placing files"
install_nobackup /system/xposed.prop                      0    0 0644
install_nobackup /system/framework/XposedBridge.jar       0    0 0644

install_and_link /system/bin/app_process32                0 2000 0755 u:object_r:zygote_exec:s0
install_overwrite /system/bin/dex2oat                     0 2000 0755 u:object_r:dex2oat_exec:s0
install_overwrite /system/bin/oatdump                     0 2000 0755
install_overwrite /system/bin/patchoat                    0 2000 0755 u:object_r:dex2oat_exec:s0
install_overwrite /system/lib/libart.so                   0    0 0644
install_overwrite /system/lib/libart-compiler.so          0    0 0644
install_overwrite /system/lib/libart-disassembler.so      0    0 0644
install_overwrite /system/lib/libsigchain.so              0    0 0644
install_overwrite /system/lib/libxposed_art.so            0    0 0644
if [ $IS64BIT ]; then
  install_and_link /system/bin/app_process64              0 2000 0755 u:object_r:zygote_exec:s0
  install_overwrite /system/lib64/libart.so               0    0 0644
  install_overwrite /system/lib64/libart-compiler.so      0    0 0644
  install_overwrite /system/lib64/libart-disassembler.so  0    0 0644
  install_overwrite /system/lib64/libsigchain.so          0    0 0644
  install_overwrite /system/lib64/libxposed_art.so        0    0 0644
fi

if [ "$API" -ge "22" ]; then
  find /system /vendor -type f -name '*.odex.gz' 2>/dev/null | while read f; do mv "$f" "$f.xposed"; done
fi

echo "- Done"
exit 0
```

 - 安装

```
cd /
mkdir system
cd /tmp/xposedtest/android-arm32/
sudo mount system.img /system
cd /xposed/xposed-v79-sdk22-arm/
./flash_script.sh
```

- 启动带Xposed安卓系统

```
cd /tmp/xposedtest/android-arm32/
emulator -avd arm32 -system system.img -data userdata.img -ramdisk ramdisk.img
```

启动的过程中，会发现卡死在显示android字样的启动界面上。  
这时在新启动一个shell，执行：  
```
adb devices
```

若发现emulator是off状态，那等一会再执行此命令，直到emulator是正常的启动状态。  
这时若执行：  
```
adb logcat
```
会发现xposed被selinux拦截了。  
下面的命令可以暂时关闭selinux（本次系统关闭，重启系统后会自动恢复），目前还没找永久关闭的方法。  
```
setenforce 0
```

执行完这个命令后，系统会继续启动，耐心等待，直到系统启动完成。  

- 安装Xposed模块

```
adb install [Flat Style Colored Bars].apk
```

- 启用Xposed模块

在模拟器中点击XposedInstaller，选择模块，勾选[Flat Style Colored Bars]，启用他。然后重新启动模拟器（注意启动过程中还是需要关闭selinux）。  

- 验证Xposed的有效性

重启后，再次在模拟器中点击XposedInstaller，选择模块，点击[Flat Style Colored Bars]，会进入其配置界面，修改一些配置，观察界面上其相应的变化。  

比如，修改battery，假设原来通知栏上的电池图标是竖着的电池状。这里可以把其修改为横着的电池状，修改后，几秒后，电池就会变为横着的。  



