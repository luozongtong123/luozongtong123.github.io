
<!--more-->

# 烧录镜像  
拿到树莓派后第一步是将系统镜像烧录到存储卡。镜像下载：[地址](https://www.raspberrypi.org/downloads/raspbian/)。这里选择没有桌面的 Raspbian Lite。烧录工具使用 [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)。

# 开启 SSH  
Raspbian 默认关闭 SSH，在第一次开机前需要设置为开启。方法为：在烧录完成的内存卡的 boot 分区的根目录下创建一个名为 ssh 的文件。

{{% admonition warning %}}
这个名为 `ssh` 的文件不应含任何扩展名。
{{% /admonition %}}

# 连接显示器  
如果想直接连接显示器使用，则需要对显示器相关的设置进行更改。具体方法见：[树莓派连接显示器的问题](/post/raspberrypi-hdmi/)

# 修改主机名  
方法一：直接修改文件
$ sudo nano /etc/hostname   #将 raspberrypi 修改为合适的名称
$ sudo nano /etc/hosts           #将 raspberrypi 修改为合适的名称  

方法二：
raspi-config -> 2 Network Options -> N1 Hostname 

# 修改软件源并更新软件  

``` bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak  #备份为 sources.list.bak
sudo nano /etc/apt/sources.list 

在编辑界面用 # 注释掉所有行，添加如下两行
deb https://mirrors.aliyun.com/raspbian/raspbian/ stretch main contrib non-free rpi 
#deb-src https://mirrors.aliyun.com/raspbian/raspbian/ stretch main contrib non-free rpi 

sudo apt-get update
sudo apt-get upgrade
```

# 使用 raspi-config  
raspi-config 是 Raspberry PI 官方 Raspbian 镜像自带的一个系统配置工具。它可以用来进行更改密码等设置。  
参考：[树莓派官方设置工具 raspi-config](http://www.shumeipaiba.com/wanpai/jiaocheng/3.html)

# 连接 WiFi  
这里有两种方法：  
方法一：直接修改文件，具体操作参考：[树莓派连接 WiFi](/post/raspberrypi-wifi/)  
方法二：借助 raspi-config 进行设置。  
raspi-config :arrow_right: `2 Network Options` :arrow_right: `N2 Wi-fi` ，输入SSID和密码即可。  



[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

