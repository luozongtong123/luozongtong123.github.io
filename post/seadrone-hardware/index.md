ROV 因为固件升级出错需要拆机恢复固件，趁此机会把硬件结构摸了一下。  
<!--more-->

## 一、结构

通过拆解发现，SeaDrone结构较为简单，拆卸方便。密闭舱的拆卸只需拆下一个螺栓。密封舱的盖子和机体的密封通过两个密封圈来实现。小密封圈容易在拆卸的时候与盖子粘到一起，拆卸时要注意。大密封圈在机体密封接触面的外侧。

{{% figure class="center" src="/img/seadrone-hardware/1-1.jpg" alt="整体视图" title="整体视图" %}}   

{{% figure class="center" src="/img/seadrone-hardware/1-2.jpg" alt="密封圈" title="密封圈" %}}   



## 二、主控

主控板是Raspberry Pi 3 Model B，内部运行Raspbian Linux 。其登陆用户名和密码为：username: pi ，password: seadrone。通过SSH登陆之后可以获得root 权限。进而可以安装软件编写运行自己的软件，可以用这种方式通过以太网与上位机通信，不依赖ROV固件。目前，树莓派上还有三个空余的USB口（可接HUB扩展，串口转USB等方式扩展通讯接口）。

{{% figure class="center" src="/img/seadrone-hardware/2-1.jpg" alt="树莓派" title="树莓派" %}}   

{{% figure class="center" src="/img/seadrone-hardware/2-2.jpg" alt="树莓派" title="树莓派" %}}   



## 三、电机驱动及传感器

### 1. 电机驱动

共有5块电机驱动，电机驱动的主控为TI的TMS320系列32位DSP。电机驱动与主控板的通讯方式为串口（两线，RX和TX），电平电压3.3v，波特率921600。官方上的[Github](https://github.com/orobotix/SeaDrone-Smart-Thruster-Library)上有个开源的电机驱动api，用来通过树莓派控制电机和获取各种相关信息（转速、电流、电压、温度、警报）。可以通过一个串口同时控制最多16个电机。四位拨码开关可以设置电机驱动的ID，0x00 – 0x0F，共16个。发送信息时带一个ID，只有对应ID的驱动才会响应对应的信息。  

{{% figure class="center" src="/img/seadrone-hardware/3-1-1.jpg" alt="电机驱动" title="电机驱动" %}}   

{{% figure class="center" src="/img/seadrone-hardware/3-1-2.jpg" alt="电机驱动拨码" title="电机驱动拨码" %}}   

### 2. IMU

IMU使用的是MPU9250，与树莓派的通信接口应该是I2C。MPU9250包含三轴加速度计、三轴陀螺仪、三轴磁力计，可以实现AHRS。

{{% figure class="center" src="/img/seadrone-hardware/3-2.jpg" alt="IMU" title="IMU" %}}   



### 3. 摄像头

摄像头包括两部分：一是图像数据的传输，通过USB接口直接与树莓派相连；二是摄像头的转动，应该是类似一个舵机，使用PWM的方式控制。

{{% figure class="center" src="/img/seadrone-hardware/3-3.jpg" alt="摄像头" title="摄像头" %}}   



### 4. 磁开关

磁开关控制电源的方式没有关注，下次拆解可以关注一下。

{{% figure class="center" src="/img/seadrone-hardware/3-4.jpg" alt="磁开关" title="磁开关" %}}   



### 5. 深度传感器

深度传感器是一个压力传感器，通过穿舱口直接与水接触。通信方式I2C。芯片型号为MS5837-30BA，最高压力为30bar（300m），分辨率0.2bar（2mm）。供电电压3.3v-5.5v。四线两根供电两根通信（SDA、SCL）。价格参考淘宝：100元人民币。

{{% figure class="center" src="/img/seadrone-hardware/3-5.jpg" alt="深度传感器" title="深度传感器" %}}   



### 6. 温度传感器

未找到。



## 四、通舱口

机体上大概预留了5个可以实现通舱的口，与商家沟通后确认可以提供配套电缆实现自由扩展。



{{% figure class="center" src="/img/seadrone-hardware/4.jpg" alt="通舱螺栓" title="通舱螺栓" %}}   



## 五、以太网通信的实现

树莓派中以太网经过载波通信模块的转换，变为两线制信号，通过电缆传到岸上控制盒，岸上控制盒再经过载波通信模块由两线制信号转换为以太网信号。以太网再通过无线路由器的交换机功能与热点功能组成局域网。所有的通信便通过这个局域网进行。   

岸上缆绳控制盒由三部分组成：载波通信转换模块、Buck升压电路、路由器。路由器供电电压为12V，Buck电路将5v充电宝电压升为12v后为路由器供电。  

{{% figure class="center" src="/img/seadrone-hardware/5-1.jpg" alt="载波通信模块" title="载波通信模块" %}}   

{{% figure class="center" src="/img/seadrone-hardware/5-2.jpg" alt="岸上控制盒" title="岸上控制盒" %}}   

控制数据通过UDP通信传输。   

视频数据的传输：树莓派安装了一个名为[mjpg-streamer](https://github.com/jacksonliam/mjpg-streamer) 的插件来实现。相当于在树莓派上建立了一个HTTP服务器，用来传输视频信号。   

基于这个局域网，我们可以编写自己的程序与岸上设备通信，无需依赖ROV固件程序，也不会与原有的程序发生冲突。



## 六、供电方案

电池电压29v左右，8x3.7=29.6，可能是16节18650两两并联最后串联组成电池组。目前已知的内部电压至少有5v和3.3v。

{{% figure class="center" src="/img/seadrone-hardware/6.jpg" alt="电池端子" title="电池端子" %}}   





[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

