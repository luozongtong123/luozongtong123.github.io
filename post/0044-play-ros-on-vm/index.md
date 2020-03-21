最近想研究一下 `uuv_simulator` 怎么使用，可能的话作为我毕业论文的一部分。`uuv_simulator` 是基于 ROS 的， ROS 官方支持的系统是 Ubuntu ，而我机器上跑得是 Manjaro，如果在新分区或着是在新硬盘上安装 Ubuntu 则需要高一整套的学习和工具链，思来想去还是安装虚拟机吧。于是就有了这篇博文。

<!--more-->

# 虚拟机管理软件  

刚开始接触 Linux 的时候就是从虚拟机开始玩的，那个时候也是用的这台电脑，当时还是 4G 内存和机械硬盘， Windows 上再开个虚拟机整得比较卡，体验很差。现在已经内存升级到 8G + 固态硬盘，同时也是在 Manjaro 下开虚拟机，想来应该不会太卡。虚拟机管理软件的话其实很容易选择。在 Windows 平台下可能还有 VMware 的选择，但是在 Linux 下 [VirtualBox](https://www.virtualbox.org/) 是最好的选择。下载安装就直接在用 `yay` 了 (还好不用忍受 VirtualBox 官网那乌龟爬的速度)。  

``` bash
yay virtualbox
```

# 客户端 Linux 发行版选择  

`uuv_simulator` 本身支持 `kinetic` `lunar` `melodic` 三个 ROS 的发行版本。按照我的喜好的话当然选择最新的 `melodic` 用。`melodic` 支持 Ubuntu 18.04 版本，18.04 其实已经不太新了，我使用的 Guake 在 Manjaro 上都更新到 3.6 了，而在 18.04 上才到 3.0，连开机自启动都没有。话说回来，使用 `melodic` 最好还是用 18.04，长期支持版更靠谱一些。  

Ubuntu 默认的桌面要臃肿一些，为了虚拟机不那么卡，同时也为了和我的 Manjaro 的使用习惯保持一致，我选择使用以 Xfce 作为默认桌面系统的 [Xubuntu](https://xubuntu.org/)。  

安装的话也比较简单，直接在 VirtualBox 中新建虚拟机以及相应的虚拟硬盘，然后把下载好的 Xubuntu 镜像挂载就可以。然后顺着安装向导一步步安装即可。  

# 系统配置  

稍微等一会系统就安装好了，进入系统的第一件事是安装 VirtualBox 的增强功能，安装了增强功能就可以让虚拟机的分辨率跟随窗口大小，并可以共享剪贴板。VirtualBox 中直接下载经常下载失败，速度也是及其感人，在此提供一个下载好的包，直接下载好添加到光驱文件即可，[下载连接](/zip/0044-play-ros-on-vm/VBoxGuestAdditions_6.0.14.7z)，版本号是 6.0.14，如果想用其他版本的就自己到官网上下载把，官网[下载链接](https://download.virtualbox.org/virtualbox)。  

安装增强功能之前需要安装编译的工具，直接通过一行命令搞定: `sudo apt install build-essential`。啊，还有件事情忘了，如果 Ubuntu 默认的源速度不快的话可以到 `Settings > Software & Update > Download from > Other...` 中选择你想要的源，我的话直接选择了 `Select Best Server` ，系统帮我选择了南京邮电大学的源。  

# 安装 ROS 及 `uuv_simulator`  

## 勾选 "restricted" "universe" and "multiverse"  

位置是 `Settings > Software & Update` 。  

## 选择 ROS 源  

官方源：  
``` bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```
国内源：  
``` bash
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
# 或者
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
```

## 设置密钥  

``` bash
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
# 或者
curl -sSL 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0xC1CF6E31E6BADE8868B172B4F42ED6FBAB17C654' | sudo apt-key add -
```

## 安装  

```bash
sudo apt update
sudo apt install ros-melodic-desktop-full
sudo apt install ros-melodic-uuv-simulator

# 环境变量
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

# Quick start  

```bash
roslaunch uuv_gazebo_worlds empty_underwater_world.launch
roslaunch uuv_descriptions upload_rexrov.launch mode:=default x:=0 y:=0 z:=-20 namespace:=rexrov
```

简单启动试了一下。  

{{% figure class="center" src="/img/0044-play-ros-on-vm/gazebo.png" alt="gazebo" title="gazebo" %}}

有一个报错如下：  

``` bash
libcurl: (51) SSL: no alternative certificate subject name matches target host name 'api.ignitionfuel.org'
```

将 `~/.ignition/fuel/config.yaml` 中 `url: https://api.ignitionfuel.org` 替换成 `url: https://api.ignitionrobotics.org` 即可。

