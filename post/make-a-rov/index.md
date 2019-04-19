一步一步做自己的 ROV。
<!--more-->

# 一、概述

注：ROV 的结构设计和密封舱的设计是一个复杂的问题，本文将直接略过这个问题的讨论，直接给出如下的 ROV。本文将着重与 ROV 舱内的电子元件的组装以及程序的设置等。

<img src="/img/make-a-rov/轴测图.jpg" width="60%" height="60%">

<img src="/img/make-a-rov/俯视图.jpg" width="60%" height="60%">

<img src="/img/make-a-rov/主视图.jpg" width="60%" height="60%">

## 1、推进器配置

如上面给出的图所示，ROV 一共有 8 个推进器，其中 4 个推进器在水平面上呈对角布置，可以为 ROV 在水平面上的各种运动（包括，平移和原地转动）提供推力。这四个推进器标号为 1-4 。还有四个推进器布置在垂直方向，这四个推进器除了可以为 ROV 提供上浮和下潜的推力外还可以使 ROV 进行俯仰和横滚运动。这四个推进器标号 5-8 。应当注意到，为了防止 ROV 由于推进器的转动而产生的反向的转矩对 ROV 的姿态造成影响，推进器是正反浆对称布置的。其中绿色的为 CCW （逆时针）浆，蓝色的为 CW （顺时针）桨。

<img src="/img/make-a-rov/frame-numberings.jpg" width="60%" height="60%">

## 2、控制系统

控制系统采用开源项目 [ArduSub](https://www.ardusub.com/) 的方案。其控制核心是 Pixhawk ，配合树莓派完成控制数据和视频数据的传输。系统的硬件框图如下所示：

![hardware-diagram](/img/make-a-rov/hardware-diagram.png)

# 二、供电方案

## 1、供电电池

推进器使用的是 ROVMAKER 家的[动力总成](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.72f42e8dvns4rT&id=553255419051&_u=n1krgas19c88)。该推进器的推荐的供电电压是 6S。前面给出的 ROV 的结构和密封舱制作完成后，我们对它进行了浮力测定，最终测得浮力为 7341g。为了避免 ROV 制成后安装过多的配重块，选用了两块 16000mah 的航模电池，每块电池重约 2kg 。

## 2、DC-DC 降压模块

使用多个 [DC-DC 降压模块](https://detail.tmall.com/item.htm?spm=a1z0d.6639537.1997196601.44.251174844ayKY5&id=558664731724)，为除了推进器外的所有其他部件供电。该模块的输出电压默认可调。在模块背面切断 ADJ 的那段铜箔并用焊锡连接其他标有不同电压的焊盘即可快速获得不同的常用电压等级。该模块还有一个使能端口，高电平有效，默认内部上拉，若想断掉这一路供电只需要给一个低电平即可。

使用到的电压包括一下几个部分：

* 5.0V：树莓派供电；
* 5.3V：飞控 MAIN 、AUX 输出端口供电；
* 12.V：电力线载波通信模块供电；
* 9.0V：LED灯带供电；
* 12.V：LED高亮补光灯供电；
* 7.4V：舱外水下舵机供电；
* 5.0V：继电器控制端供电；


## 3、水密接插件

水密接插件使用 8 芯的[水密接插件](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.49ab2e8dbytuPm&id=570101069120&_u=n1krgas16f03)，该接插件的单芯额定电流为5A，接插件的芯线定义（从母头方向看）如下图所示：

<img src="/img/make-a-rov/waterproof_plug.jpg" width="30%" height="30%">

* 线号 1：1#电池充电，正极；
* 线号 2：2#电池充电，正极；
* 线号 3：载波通信 L 线（白色）；
* 线号 4：开关 1 ；
* 线号 5：载波通信 N 线（黑色）；
* 线号 6：2#电池充电，负极；
* 线号 7：1#电池充电，负极；
* 线号 8：开关2；

**ROV 上电：**

线缆盘插头使用了线号 3、5 和 4、8 的芯线，线号 3、5 连接通信线缆，线号 4、8 短接。在 ROV 内部，线号4、8 连接到继电器控制端，当线缆盘插头插到 ROV 上时，插头将线号 4、8 短接，接通继电器， ROV 上电。

**ROV 充电：**

由于我们使用的是没有保护板的航模锂电池，所以不能直接对锂电池充电，必须借助平衡充电器。我们的方案是为航模电池安装[锂电池保护板](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.49ab2e8dbytuPm&id=563723443594&_u=n1krgas12c24)。直接将两块锂电池的平衡充电头并联后连接到充电保护板，保护板的 P+ 和 P- 引出，作为充电接口。线号 1、2 并联，连接到 P+；线号 6、7 并联，连接到 P- 。（并联的目的是增强过流能力）

这样就可以直接使用两线的[充电器](https://item.taobao.com/item.htm?spm=a1z2k.11010449.931864.16.4f24509de8yP9Y&scm=1007.13982.82927.0&id=571402510404&last_time=1534992375)对 ROV 进行充电了。

充电时注意：应该先将 ROV 与充电器连接后再将充电器连接 220V 电源；充电器和 ROV 连接后充电器会有一个灯变亮；充电器连接 220V 电源后会有两个红灯亮，充电完成后一个红灯会变绿。


## 4、电流计

ROV上安装的[电流计](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.49ab2e8dbytuPm&id=520582169727&_u=n1krgas1ee11)可以测量电池电压和通过的电流大小。同时电流计的接口还可以同时为飞控提供电源供电。电流计和飞控 MAIN 、AUX 输出端口供电组成双路供电，使飞控运行更加可靠。需要注意的的是电流计安装在继电器之后。

# 三、推进器连接

## 1、推进器介绍

推进器使用的是 ROVMAKER 家的[动力总成](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.72f42e8dvns4rT&id=553255419051&_u=n1krgas19c88)。该推进器使用的是开放式的电机，电子绕组做好防水防腐蚀处理后可以直接与水接触，没有密封舱，不存在密封问题，使用的深度可达几百米。电调集成在了推进器上。电调灌封在一个密闭的容器中，从容器中引出三条线，颜色分别为红色、黑色、黄色。红色为推进器供电正极，黑色为推进器供电负极，黄色为推进器控制转速的 PWM 的信号线。

推进器的性能参数介绍如下：

* 电调最大电流：30A
* 电压：3S~6S（推荐6S）
* KV值：350
* 电调 PWM 范围：1000（翻转最高）-2000（正转最高） 中位是1500
* 频率：50Hz


## 2、推进器测试

推进器安装前可能需要对推进器进行测试以确定推进器可以正常工作。测试工具是[舵机测试仪](https://item.taobao.com/item.htm?spm=a1z10.5-c.w4002-17373907211.61.28d740abz1qAF0&id=547409965763)。推进器和舵机测试仪连接好电源线（注意共地）和 PWM 信号线。线路连接好后还无法直接控制推进器转动，需要进行性初始化。初始化方法如下：

1. 接线，上电 哔-哔-哔 三声，表示电调开机正常；
2. 给电调 2000 或 1000 最高转速信号， 哔 一声；
3. 给电调 1500 停转信号， 哔 一声；
4. 初始化完成后就可以开始调速控制

## 3、推进器的接线

由于推进器较多，为了方便进行推进器的拆装的理线，我们使用接线端子连接推进器。在接线端子上将所有推进器的正负极并联到一起，所有的 PWM 信号线标号（标号参考前文推进器配置）并引出。推进器的 PWM 信号线安按照 1-8 的顺序连接到飞控的 Main output 1-8。

# 四、各种设备与飞控的连接

## 1、飞控简介

飞控是整个系统的控制核心。其所有端口的 [Pinouts](https://docs.px4.io/en/flight_controller/pixhawk.html) 如下图所示：

![pixhawk_connectors](/img/make-a-rov/pixhawk_connectors.png)


## 2、Main output 和 Auxiliary output 的连接

Main output（主通道） 的连接前文已经有所叙述，即编号为 1-8 的推进器的 PWM 信号线分别与 主通道的 1-8 连接。

舱内摄像头俯仰云台的舵机的三根线（舵机供电由连接到主通道的 DC-DC降压模块负责）连接到 Auxiliary output（辅助通道）的 1号通道。舱外机械爪防水舵机的 PWM 信号线连接到辅助通道的 2 号通道，该舵机由独立的 DC-DC 降压模块供电。漏水传感器的数字输出端口与辅助通道的 3 号通道连接。

[漏水传感器](https://item.taobao.com/item.htm?spm=a1z0k.6846577.0.0.55c67c29FccocN&id=545593288189&_u=t2dmg8j26111)的探头和数据采集板相连，数据采集板的供电直接由辅助通道的电源提供。实际使用的效果来看，这款漏水传感器的灵敏度高，没有误报，使用效果很好。

高亮 LED 补光灯供电 DC-DC 降压模块的使能端口与辅助通道的 4 号通道相连，LED 灯带供电 DC-DC 降压模块的使能端口与辅助通道的 5 号通道相连，舱外两个内窥镜摄像头的 LED 灯与辅助通道的 6 号通道相连。


## 3、其他设备的连接

深度传感器与飞控的 I2C 接口相连；电流计与 POWER 端口相连；蜂鸣器和 BUZZER 端口相连；按钮和 SWITCH 相连；GPS 模块与 GPS 端口相连。

GPS 端口的[线号](https://docs.px4.io/en/flight_controller/pixhawk.html)定义如下：

<img src="/img/make-a-rov/GPS_port.jpg" width="80%" height="80%">

注：线的颜色是官方标注，实际的线的颜色可能会有不同。在使用时可以将飞控上印刷的文字摆正，从左到右依次为 1-6。

我们使用的 GPS 模块没有罗盘功能，只需要连接供电以及 RX、TX。

# 五、各种设备和树莓派的连接

树莓派的供电是通过 DC-DC 降压模块完成的，降压模块得到的 5V 电压直接和树莓派的 5V 供电相连。

<img src="/img/make-a-rov/pi_pinouts.png" width="80%" height="80%">

飞控通过 USB 接口直接和树莓派相连；一个舱内摄像头和两个舱外摄像头连接到 USB 接口；电力线载波模块通过以太网线和树莓派相连。

# 六、电力线载波通信测试

选用的[电力线载波通信模块](https://item.taobao.com/item.htm?spm=a1z10.5-c.w4002-17373907211.27.6fc422e6NUC6ET&id=538928813306)使用非常方便，不需要做复杂的初始化设置。两块电力线载波通信模块的接线方法如下图所示。

<img src="/img/make-a-rov/PLC.jpg" width="80%" height="80%">

电力线载波通信模块的测试方法很简单，把两块模块当作一段网线，直接将这段网线接入路由器测试网络是否连通即可。

#七、线缆盘

线缆盘使用 220V 交流电线缆盘改造而成。原缆盘的面板拆除不用，自行设计并 3D 打印了用于安装开关等元件的面板。面板上包含开关、缆盘内电量指示、线缆盘充电口、网线插口。内部集成有 4 节 18650 锂电池，充放电保护板以及电力线载波通信模块。

另外，线缆盘配有专用的 4S 锂电池充电器。

# 八、各软件的安装

软件包括三部分，首先是飞控的固件 [ArduSub](https://github.com/ArduPilot/ardupilot)，地面站软件 [QGroundControl](https://github.com/bluerobotics/qgroundcontrol)，树莓派上运行的辅助程序 [Companion](https://github.com/bluerobotics/companion)。下载链接分别列出：QGroundControl [Windows](https://s3.amazonaws.com/downloads.bluerobotics.com/QGC/QGroundControl-installer.exe )、[Mac](https://s3.amazonaws.com/downloads.bluerobotics.com/QGC/QGroundControl.dmg)、[Linux](https://s3.amazonaws.com/downloads.bluerobotics.com/QGC/QGroundControl.AppImage)；ArduSub Firmware Files [ArduSub V3.5](http://firmware.ardupilot.org/Sub/stable/PX4/ArduSub-v2.px4)；Raspberry Pi Images [Latest Ardusub-Raspbian Image](https://s3.amazonaws.com/downloads.bluerobotics.com/Pi/stable/ardusub-raspbian.img.zip)。

各软件的安装步骤可以参考[这里](https://www.ardusub.com/getting-started/installation.html)。

# 九、视频流调试

机械爪安装在 ROV 底部，舱内摄像头无法观察到 ROV 底部情况，ArduSub 官方没有多摄像头支持，为了观察到 ROV 底部情况，增加独立的视频流传输系统。视频传输软件用的是 [mjpg-streamer](https://github.com/jacksonliam/mjpg-streamer)，摄像头使用的是提供 mjpg 格式的摄像头。

树莓派上安装  mjpg-streamer 的步骤如下：

```bash
# 首先 SSH 登陆到树莓派
# 用户名：pi 密码：companion

#将代码库克隆到本地
git clone https://github.com/jacksonliam/mjpg-streamer.git
#安装依赖
sudo apt-get install cmake libjpeg8-dev
# 编译代码
cd mjpg-streamer-experimental
make
sudo make install
# 查看摄像头的设备符 
#一般来讲舱内摄像头会占用两个摄像头设备符，video0 是mjpg格式的，video1 是h.264格式的
ls /dev
#启动视频流传输
cd mjpg_streamer
mjpg_streamer -i 'input_uvc.so -d /dev/video2 -r 1280x720' -o './output_http.so -p 8092 -w ./www'
# 在浏览器上输入如下网址，即可以观看视频流
# http://192.168.2.2:8092/?action=stream

#为了方便启动视频流，在用户的 home 目录建立脚本，启动视频流时只需运行此脚本
cd
touch start_video_streaming.sh
nano start_video_streaming.sh
#将如下内容复制到 nano 中，Ctrl + x 保存
cd mjpg_streamer
mjpg_streamer -i 'input_uvc.so -d /dev/video2 -r 1280x720' -o './output_http.so -p 8092 -w ./www'
#增加执行权限
chmod u+x start_video_streaming.sh
#此后，如果需要启动视频流则只需要登陆到树莓派并执行如下命令
./start_video_streaming.sh
#也可以将脚本设置为开机自动执行
#修改/etc/rc.local文件
#在 exit 0 之前添加如下命令
./home/pi/start_video_streaming.sh &

```

# 十、QGroundControl 的设置

## 1、加速度计校准

![calibrate_accelermeter](/img/make-a-rov/calibrate_accelermeter.jpg)

在如上图的界面中点击 Accelrmeter 并在 Autopilot Orientation 下拉选项中选择 None，确认后按照指示将飞控按照不同的方向摆放，直到校准完成。

##2、罗盘校准

![calibrate_compass](/img/make-a-rov/calibrate_compass.jpg)

在如上图的界面中点击 Accelrmeter 并在 Autopilot Orientation 下拉选项中选择 None，确认后按照指示将飞控随机地转动，直到校准完成。

加速度计和罗盘校准完成后可以进行 Level Horizon 校准，只需要将飞控水平放置，电机按钮并保持一段时间的水平放置即可。


## 3、深度传感器的校准

由于压力传感器的安装位置以及由于传感器的误差，当 ROV 浮在水面上时，深度计的度数可能不为零。此时便可以进行深度传感器的零位校准。当 ROV 浮在水面上时，单击 Calibrate Pressure ，确认，等待片刻便可完成压力传感器的零位校准。

## 4、推进器及推进器布置设置

![frame](/img/make-a-rov/frame.jpg)

推进器框架选择 Vectored-6DOF。




[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

