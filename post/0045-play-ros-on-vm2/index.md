[在虚拟机上使用 ROS](/post/0044-play-ros-on-vm/) 中记录了虚拟机上 ROS 的配置，文章最后简单试了下在 Gazebo 中仿真，可以在截图上看到仿真的帧率只有可怜的 1.58FPS。本篇文章尝试在上文的基础上开启 Unity 3D 的加速支持。
<!--more-->

# 检查是否已经开启了加速  

``` bash
sudo apt install nux-tools
/usr/lib/nux/unity_support_test -p
```

如果如下图所示，Unity 3D supported 后面显示的是那个可爱的绿色 <font color="green">yes</font> ，那么恭喜你，可以关掉网页了。如果是讨厌的红色 <font color="red">no</font>，那么就继续往下看吧。😏

{{% figure class="center" src="/img/0045-play-ros-on-vm2/unity_support_test.png" alt="unity_support_test" title="unity_support_test" %}}  

# 准备哈系统  

``` bash
sudo apt update && sudo apt upgrade
sudo apt install build-essential module-assistant dkms
sudo m-a prepare
```

# 重新安装哈 VirtualBox 的扩展功能  

挂载扩展功能的 ISO，具体可参见[上篇文章](/post/0045-play-ros-on-vm2/)。重新安装下扩展功能。

``` bash
./autorun
```

# 在 VirtualBox 中开启 3D 加速  

首先将虚拟机关机。然后进行如下设置：

{{% figure class="center" src="/img/0045-play-ros-on-vm2/3d-acceleration.png" alt="3d-acceleration" title="3d-acceleration" %}}

# 最后再进入系统测试下  

{{% figure class="center" src="/img/0045-play-ros-on-vm2/unity_support_test.png" alt="unity_support_test" title="unity_support_test" %}}  

唉？怎么又是这张图?🤔  

{{% figure class="center" src="/img/0045-play-ros-on-vm2/gazebo.png" alt="gazebo" title="gazebo" %}}  

虽然帧率仍然并不是很高而且帧率波动还比较大但是总的来说勉强够用。  

大功告成！收工。


# 参考  

[How to Speed up Ubuntu 16.04 and 17.04 in VirtualBox
](https://www.linuxbabe.com/virtualbox/speed-up-ubuntu-virtualbox)  
[Can't run /usr/lib/nux/unity_support_test -p to see if unity is supported](https://ubuntuforums.org/showthread.php?t=1856762)  


