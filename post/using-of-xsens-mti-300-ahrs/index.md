
今天实验室来交流的泰国老师给我讲解了一下 Xsens MTi-300 AHRS 的使用，现在稍稍总结一下。

<!--more-->

## 一、型号分类

根据官方选型手册，Xsens MTi  系列传感器共有三个系列：

1.MTi 1-series

-  MTi-1 IMU 
-  MTi-2 VRU
-  MTi-3 AHRS

2.MTi 10-series

-  MTi-10 IMU 
-  MTi-20 VRU
-  MTi-30 AHRS

3.MTi 100-series

-  MTi-100 IMU 
-  MTi-200 VRU
-  MTi-300 AHRS
-  MTi-G-710 GNSS/INS

其性能随着数字的增多而提高，MTi 100-series系列性能最高。其中1开头的传感器仅仅包含惯性测量单元（Inertial measurement unit  -- **IMU**），提供三轴加速度数据，并不包含三轴姿态角的解算；2开头的传感器在1开头的基础上增加三轴姿态角的解算，称为垂直参考单元（Vertical Reference Unit -- **VRU**），之所以称为垂直参考单元是因为提供的偏航角是相对的并不是绝对的；3开头的传感器在2开头的传感器中增加磁力计，组成航向参考系统（Heading Reference System -- **AHRS**）。

我们目前使用的型号是 **MTi-300 AHRS**。



## 二、相关软件

### 1、MT Manager

该软件提供传感器的设置、数据的可视化及记录等功能；

### 2、Magnetic Field Mapper

该软件提供3开头的传感器的周围磁场的测算，并可以把测算的结果写入到传感器里进而提高传感器的准确度；

### 3、Firmware Updater

该软件可以给传感器估计升级。



要注意，各种相关的软件最好使用最新版，因为新版更新会提供更好的性能以及修复Bug等。

要**尤为注意**的是要保持传感器固件的版本保持最新，因为固件中包含地球磁场的数据，地区磁场数据每年都在变化，所以若想要获得最佳的效果最好使用最新的地球磁场数据。





## 三、滤波配置

官方为 MTi-300 AHRS 系列传感器提供了五种滤波配置，包括：general、high_mag_dep、dynamic、low_mag_dep、vru_general。

其中 vru_general 相当于去掉了磁力计的作用，偏航角变成相对的（取上电时的位置为0），同样其结果不受周围磁场的影响。而 high_mag_dep 则是在计算偏航角时相对惯性测量单元的数据更加依赖磁场的测量数据。

总得来说，在决定使用何种配置时需要考虑周围磁场的情况，若周围磁场不是特别强并且不会发生变化时可以选择较为依赖磁力计的配置文件，否则则需要使用 vru_general 配置。



原文参考：

The **general** filter profile is the default setting. It assumes moderate dynamics and a homogenous
magnetic field. External magnetic distortions are considered relatively short (up to ~20 seconds).
Typical applications include camera tracking (e.g. TV camera’s), remotely operated robotic arms on
ROV’s etc

The **high_mag_dep** filter profile assumes homogenous magnetic field and an excellent Magnetic Field
Mapping. This filter profile heavily relies on the magnetometer for heading. Dynamics of the motion arerelatively slow. Typical applications are navigation of ROV’s or the control of small unmanned
helicopters.

The **dynamic** filter profile assumes jerky motions. However, the assumption is also made that there is no GNSS available and/or that the velocity is not very high. In these conditions a 100-series MTi may be a better choice. The dynamic filter profile uses the magnetometer for stabilization of the heading,and assumes very short magnetic distortions. Typical applications are where the MTi is mounted on persons or hand-held (e.g. HMD, sports attributes etc.).

The **low_mag_dep** filter profile assumes that the dynamics is relatively low and that there are long-
lasting external magnetic distortions. Also use this filter profile when it is difficult to do a very good
Magnetic Field Mapping (MFM). The use of the low_mag_dep filter profile can be useful to limit drift in
heading whilst not being in a homogenous magnetic field. Typical applications are large vessels and
unmanned ground vehicles in buildings.

The **vru_general** filter profile assumes moderate dynamics in a field where the magnetic field cannot
be trusted at all and benefits from the Active Heading Stabilization feature. It is also possible to use
this filter profile in situations where an alternative source of yaw is available. Yaw from the VRU is
unreferenced; note however, that because of the working principle of the VRU, the drift in yaw will be
much lower than when gyroscope signals would be integrated. Typical applications are stabilized
antenna platforms mounted on cars of ships and pipeline inspection tools. This filter profile is the onlyone available for the MTi-20 VRU.

Every application is different and although example applications are listed above, results may vary
from setup to setup. It is recommended to reprocess recorded data with different filter profiles in MT
Manager to determine the best results in your specific application.



## 四、校准惯导


### 更改**滤波设置**： 

使用 **USB** 线将传感器与电脑相连后，打开 MT Manager 软件，搜索到设备后，打开 MT Setting 界面，选择 Device Setings 选项卡，在 Filter Setting中选择滤波配置，右上角 Write To MT 后便可生效。

### **误差估算**：

当传感器偏航角数据发生明显漂移时需要进行这一操作。在 Gyro Bias Estimate 中可以看到有一个输入框和按钮。输入框设置误差估算的时间，应该保证其大小不小于 6 秒，在点击按钮时要确认在点击按钮后在误差估算的时间内传感器要**保持静止**。

### **当前位置**设定：

由于传感器需要根据当前位置确定所处地球磁场的数据，所以需要提供准确的经纬度数据。在 Position 中填入当前的经纬度，条件允许时同时可以提供海拔数据。



其他需要 **注意**的地方：当选择使用 vru_general 滤波配置时**要**勾选 Active Heading Stabilization（AHS），同时保证在不使用 vru_general 滤波配置时**不要**勾选 AHS。

