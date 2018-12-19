
ArduSub 官方的程序无法直接给电机发送指令，这个功能需要自己实现。为了实现这个功能首先需要搭建 Pixhawk 的编译环境。
下面将简要介绍一下环境的搭建步骤。

<!--more-->

## 一、前言

Pixhawk1（px4_v2） 飞控使用了两片单片机。Pixhawk 其实是 PX4FMU 与 PX4IO 两块板子的组合，其中PX4FMU 使用的单片机是 [STM32F427](http://www.st.com/web/en/catalog/mmc/FM141/SC1169/SS1577/LN1789) ，PX4IO 使用的单片机是 STM32F100。PX4FMU  负责复杂的浮点运算，PX4IO 负责将 PX4FMU 计算得到的控制量输出为 PWM 或 其他信号。

官方支持的是 gcc-arm-none-eabi 编译器，使用的编译管理系统的是 [Waf](https://waf.io/)，Waf 依赖 Python，mavlink 的代码的生成也依赖 Python ，除此以外 Pixhawk1 上运行着 [NuttX](http://nuttx.org/) RTOS，Pixhawk1 固件的生成依赖 [PX4Firmware](https://github.com/PX4/Firmware)。

官方支持的编译环境包括一下几个：[Linux/Ubuntu](http://ardupilot.org/dev/docs/building-setup-linux.html)、[Windows](http://ardupilot.org/dev/docs/building-setup-windows.html)、[Windows10 WSL](http://ardupilot.org/dev/docs/building-setup-windows10.html)、[Windows Cygwin](http://ardupilot.org/dev/docs/building-setup-windows-cygwin.html)、[MacOSX](http://ardupilot.org/dev/docs/building-setup-mac.html)。目前使用的 Windows10 系统，电脑性能较差虚拟机暂不考虑，双系统切换麻烦 暂不考虑。官方提供的 Windows 编译环境使用的是一个较老版本的 msys 性能较差 不考虑。Cygwin 包的配置需要一个一个的安装非常费劲同时性能也很差 不考虑。那么现在只剩下一个选项 Windows10 WSL。之前也使用过一段时间的 Windows10 WSL 但是后来还是转投了 msys2 的阵营，主要原因是最初 WSL 与 Windows10 文件系统的交互较差，只能从 WSL 中访问 Windows10 的文件，无法从 Windows10 中访问 WSL 里的文件，有时这是一个非常恼人的特性。最近发现可以直接从 Windows10 中访问 WSL 的文件系统，我又欢快地从 msys2 转投了 WSl 阵营。另外，提一下完整编译的时间，使用 msys 的情况下我的电脑需要一个半小时，使用 WSL 可以缩短为半个小时，可以说性能提高很明显了。

官方提供的配置教程其实足够详细了，但是教程中只提到了使用 Ubuntu 16.04 LTS 的情况下的配置， ~~没有提 Ubuntu 18.04 LTS 的配置，也~~没有提 nijia 的安装。

## 二、编译环境配置

### 1、启用 WSL

这一步很简单，直接上图。

{{% figure class="center" src="/img/build-environment-for-ardusub/enable-wsl.jpg" alt="enable WSL" title="enable WSL" %}}

### 2、安装 Ubuntu 应用

~~如果为了省时省力可以直接安装 [Ubuntu 16.04 LTS](https://www.microsoft.com/store/productId/9PJN388HP8C9) ，现在是2018 年下半年了，为了使用新版本的程序我选择 Ubuntu 18.04 LTS 。安装时可以直接点击上面链接也可以打开 Microsoft Store 搜索 Ubuntu 选择合适的版本下载即可。~~

Ubuntu 18.04 LTS 上自带的是版本号 6.3 的 gcc-arm-none-eabi ，编译环境只支持 4.9 和 5.4。ARM 官方 4.9 和 5.4 只有 32位版本的，自行编译太过麻烦。

注：商店中没有标明版本号的 Ubuntu 应用默认安装的是最新的 LTS 版本，如果时之前安装的Ubuntu 应用，现在有了新的 LTS 版本，可以通过 do-release-upgrade 命令升级 LTS版本号。Ubuntu LTS 的服务支持的时间是 5 年。



### 3、初始化 Ubuntu

打开安装的 Ubuntu 应用，或者当系统中只安装了一个 WSL 应用时，可以直接输入 bash 启动那个唯一的 WSL。需要注意的是 *inux 系统输入密码时不会显示已输入的字符的数量。

{{% figure class="center" src="/img/build-environment-for-ardusub/wsl-init.png" alt="WSL init" title="WSL init" %}}


初始化完成后，为了提高安装软件包的速度，需要将 Ubuntu 的软件源替换成国内速度较快的软件源，这里推荐使用网易的源。

``` bash
# 用 nano 打开镜像源列表文件
sudo nano /etc/apt/sources.list 
# 注释掉所有官方的源
# 复制对应版本号的源到文件的最下方
# Ctrl + x 关闭并保存文件
# 更新软件列表、更新软件
sudo apt-get update
sudo apt-get upgrade
```

Ubuntu 16.04 LTS 源：

``` bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse

```

~~Ubuntu 18.04 LTS 源：~~

``` bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
```



### 4、克隆 Ardupilot 代码库到本地

``` bash
git clone https://github.com/ardupilot/ardupilot.git
cd ardupilot
git submodule init
git submodule update --recursive
```



### 5、进行环境设置

首先运行脚本，准备好大部分的环境。

``` bash
./Tools/scripts/install-prereqs-ubuntu.sh # 安装软件时将需要账号密码
# 若是 WSL 默认的纯净环境这一步需要下载接近 300M 的软件包，安装后占用 1.3G 的空间
nano ~/.profile 
# 删除下面这行，如果有的话， ctrl + x 保存关闭
# export PATH=/opt/gcc-arm-none-eabi-4_9-2015q3/bin:$PATH
```

脚本下载的是32位的 gcc-arm-none-eabi ，版本号 4_9-2015q3，解压的目录位置是 `/opt/gcc-arm-none-eabi-4_9-2015q3/` ，我们的 WSL 是64位的，需要下载 64 位的 gcc-arm-none-eabi ，解压的文件可以直接删除。

``` bash
sudo rm -rf /opt/gcc-arm-none-eabi-4_9-2015q3
sudo apt-get install gcc-arm-none-eabi
source ~/.profile
```



接下来安装 Ninja 构建系统。 Ninja 比 Make 更快，并且 PX4 的CMake 生成器官方支持 Ninja。不幸的是，Ubuntu 目前只支持一个非常过时的版本（Ubuntu 16.04 LTS）。下载二进制文件并添加到系统路径来安装最新版本的 [Ninja](https://github.com/martine/ninja)：

``` bash
mkdir -p $HOME/ninja
cd $HOME/ninja
wget https://github.com/martine/ninja/releases/download/v1.8.2/ninja-linux.zip 
# 最新版本是 v1.8.2，
unzip ninja-linux.zip
rm ninja-linux.zip
exportline="export PATH=$HOME/ninja:\$PATH"
if grep -Fxq "$exportline" ~/.profile; then echo nothing to do ; else echo $exportline >> ~/.profile; fi
. ~/.profile
```



### 6、进行编译

``` bash
git fetch --tags
git checkout ArduSub-stable
git submodule update --recursive
./waf configure --board px4-v2
./waf build sub
# 编译并下载程序到飞控，注意 WSL 不支持这么做
./waf --upload sub
```

### 7、下载程序  

下载程序需要使用 QGroundControl 或者 ArduSub Companion Web Interface。操作方法见如下两图。

{{% figure class="center" src="/img/build-environment-for-ardusub/firmware-setup.png" alt="QGroundControl Firmware Setup" title="QGroundControl Firmware Setup" %}}

{{% figure class="center" src="/img/build-environment-for-ardusub/pixhawk-firmware-update.png" alt="ArduSub Companion Web Interface Pixhawk Firmware Update" title="ArduSub Companion Web Interface Pixhawk Firmware Update" %}}


