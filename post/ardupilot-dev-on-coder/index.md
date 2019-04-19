本地 WSL 编译太慢了，同时又由于 WSL 和原本文件系统之间的一些问题，在 Windows 上的 VSCode 中的代码提醒无法正常工作，这对理解代码非常不友好。所以就向尝试在 Coder 上进行 Ardupilot 的开发。  
<!--more-->
前期尝试使用 Coder 的时候把两个账号里的容器都搞得不纯净了，没法测试安装依赖的步骤是否正确。还有因为 Coder 提供的硬盘空间实在是太小了，用默认的安装依赖的脚本直接把硬盘撑爆了。还好今天 Coder 上线了重置容器的功能，终于可以从零开始配置环境了。  

开始之前现需要按照 [Coder online IDE!](/post/coder/) 中配置 locale 的方法配置好 locale。

配置编译依赖:
``` bash
apt install gcc-arm-none-eabi
apt install python-pip cmake ninja-build

pip2 install --upgrade pip
pip install future lxml pymavlink MAVProxy
```

开始编译:tada:  
``` bash
git clone https://github.com/zt-luo/ardupilot.git ardupilot --recursive
cd ardupilot
git checkout sub-lab

./waf configure --board Pixhawk1
./waf sub
```
编译的时候容易因为网络问题没有响应,多等一会就好啦.  



[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

