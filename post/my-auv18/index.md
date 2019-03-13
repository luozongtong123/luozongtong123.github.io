去年去波尔图参加了一个 AUV 的学术会议，回来之后一直也没有写点总结，趁着这次参加学术沙龙做分享，就写点总结吧。  
<!--more-->

{{% figure class="center" src="/img/my-auv18/auv-18-head.png" alt="AUV 2018" title="AUV 2018" %}}

# 会议简介  

会议的全名是 2018 IEEE OES Autonomous Underwater Vehicle Symposium。会议的主办方是 IEEE Oceanic Engineering Society (IEEE OES) ，承办方是波尔图大学。会议接收的主题包括一下几个方面：  

- Vehicle Design
- Vehicle Navigation
- Sensor Fusion
- Vehicle Control
- Vehicle Planning and Execution Control
- Multi Vehicle Systems
- Vehicle Applications
- Open Source Robotics  

主要面向水下自主航行器相关的内容。本来是每两年举办一次，由于会议的影响力变大了现在改成一年办一次名字也改了，这个我会在最后再说一下。  

{{% figure class="center" src="/img/my-auv18/meeting-room.jpg" alt="主会场" title="主会场" %}}

学生参会的话主要包含两种类型，一种是根据会议主题投稿论文摘要，然后做大会口头报告；还有一种是投稿海报，根据会议提供的题目提交一个提案，然后做海报展示。

# 主题

我参加的是海报那类，即 [Student Poster Competition](https://auv2018.lsts.pt/node/17)。  

海报的主题：  

> **AUV CONCEPTUAL DESIGN CHALLENGE**  
> "AUV system for data collection in the water column"  

要解决的问题：  

> **data collection in the water column.**  

除了给定了题目以外还有一些硬性要求，比如：  

- **Endurance**: for over 9 hours;  
- **Launch and recovery**: from shore or from a ship (without the support of a workboat);  
- **Data transfer**: wireless;  
- **Maximum time between deployments**: 6 hours;  
- **Motion patterns**: profiling the water column between the surface and 100 meters along predefined paths;  
- **Sensor payload**: CTD, turbidity and fluorometer;  
- **Command and control interface**: cell-phone based;  
- **Low cost**.    

# 海报内容  

**Design of an AUV System Based on Wireless Mesh Network for Data Collection in the Water Column**


## CONCEPT  

### Background

主要尝试去解决的一个问题是水下采集数据的效率问题。主要的想法是通过大量廉价的 AUV 集群去采集数据，为了解决 AUV 之间的通信问题，使用了无线 Mesh 网作为通信手段。

### Approach  

{{% figure class="center" src="/img/my-auv18/AUV.png" alt="Fig.1 ARMs-I" title="Fig.1 ARMs-I" %}}  

设计了如 Fig.1 所示的 AUV，搭载有两个侧推两个垂推一个主推，没有舵，推进器是整体暴露在水中的，整个航行器没有动密封，密封结构简单可靠。航行器可以在海面上与临近的航行器建立无线 Mesh 网。所有的航行器都通过这个网络与其他的航行器通信，互通信息。  

{{% figure class="center" src="/img/my-auv18/network_area.png" alt="Fig.2 Network area" title="Fig.2 Network area" %}}

如 Fig.2 所示，将待采集数据的区域划分成等大小方块，方块中心有一个圆。方块区域是每个 AUV 的作业区域，圆形区域是 AUV 可以建立通信的区域。在原型区域内，1-2，1-4 都在通信范围内。之所以在作业区域内又划分了一个圆形区域是为了在保证通信的前提下尽可能不把作业区域划分的过小，减少 AUV 因为更换作业区域导致的效率降低。  

{{% figure class="center" src="/img/my-auv18/taskv.png" alt="Fig.3 A typical scenario" title="Fig.3 A typical scenario" %}}  

一个典型的工作流程如 Fig. 3 所示。首先，所有 AUV 先到第一个指定位置，到位后进行数据采集，采集完成后回到开始的位置，建立网络，交换信息，然后移动到下一个区域，重复这一步直至完成数据采集。  

**特色：**  

- **引入了无线 Mesh 网，赋予了 AUV 网络通信的能力，让 AUV 间的协作成为可能。**  
- **提出的工作流程能够有效提高水下数据采集作业的效率。**  
- **设计的 AUV 制作成本低，可以大量应用。**  

##  DESIGN SOLUTION  

### Hull Structures  

船壳的结构主要是参考了 ISiMI<sup>1</sup> 的设计。

{{% figure class="center" src="/img/my-auv18/dimensions_of_AUV2.png" alt="Dimensions of ARMs-I" title="Dimensions of ARMs-I" %}}  

| Parameters | Value | Unit | Description              |
| :--------: | :---: | :--: | :----------------------- |
|     a      |  200  |  mm  | Nose section             |
|     b      |  600  |  mm  | Mid-section              |
|     c      |  400  |  mm  | Tail sectionTail section |
|     d      |  170  |  mm  | Diameter                 |
|     e      |  60   |  mm  | Main thruster            |

### Power Consumption  

首先根据推力公式计算出所需要的推进器的推力。推力公式如下：  

$$ F_d = \dfrac{1}{2} \rho v^{2} C_d A $$

计算结果如下表：  

| Speed          | 1m/s   | 1.5m/s | 1.8m/s | 2m/s   | 2.5m/s  |
| -------------- | ------ | ------ | ------ | ------ | ------- |
| $$ F_d(N)$$    | 2.2698 | 5.1071 | 7.3541 | 9.0792 | 14.1867 |
| $$ F_d(lfb) $$ | 0.5101 | 1.1481 | 1.6533 | 2.0411 | 3.1893  |

结合主推进器的效率和功率图，确定航行器的最大速度为 2m/s 持续运行速度为 1.5m/s。

{{% figure class="center" src="/img/my-auv18/T200_Efficiency.png" alt="T200 Efficiency vs Thrust" title="T200 Efficiency vs Thrust" %}}

{{% figure class="center" src="/img/my-auv18/T200_Power.png" alt="T200 Power vs Thrust" title="T200 Power vs Thrust" %}}

### Communication  

通信方式 ESP Mesh。ESP Mesh 是一种 WiFi Mesh 的实现。  

## SCHEMATICS  

{{% figure class="center" src="/img/my-auv18/AUV_Hardware_architecture.png" alt="Hardware architecture" title="Hardware architecture" %}}

{{% figure class="center" src="/img/my-auv18/Network_topology.png" alt="Network topology" title="Network topology" %}}

## 海报

{{% figure class="center" src="/img/my-auv18/AUV2018-poster.png" alt="Poster" title="Poster" %}}

# 会场  

{{% figure class="center" src="/img/my-auv18/Rectory's-Façade .png" alt="Rectory's Façade" title="Rectory's Façade" %}}

{{% figure class="center" src="/img/my-auv18/Rectory's-Archways.jpg" alt="Rectory's Archways" title="Rectory's Archways" %}}

{{% figure class="center" src="/img/my-auv18/Interior-of-The-Noble-Hall.png" alt="Interiorof The Noble Hall" title="Interior of The Noble Hall" %}}

{{% figure class="center" src="/img/my-auv18/talk.jpg" alt="Talk" title="Talk" %}}

{{% figure class="center" src="/img/my-auv18/coffee-break.jpg" alt="Coffee break" title="Coffee break" %}}

{{% figure class="center" src="/img/my-auv18/lunch.jpg" alt="Lunch" title="Lunch" %}}

{{% figure class="center" src="/img/my-auv18/dinner.jpg" alt="Dinner" title="Dinner" %}}

{{% figure class="center" src="/img/my-auv18/award.jpg" alt="Award" title="Award" %}}

{{% figure class="center" src="/img/my-auv18/AUV18-group-photo.jpg" alt="Group photo" title="Group photo" %}}


