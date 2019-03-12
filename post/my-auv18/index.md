去年去波尔图参加了一个 AUV 的学术会议，回来之后一直也没有写点总结，趁着这次参加学术沙龙做分享，就写点总结吧。  
<!--more-->

{{% figure class="center" src="/img/my-auv18/auv-18-head.png" alt="AUV 2018" title="AUV 2018" %}}

# 会议简介  

会议的全名是 2018 IEEE OES Autonomous Underwater Vehicle Symposium。会议的主办方是 IEEE Oceanic Engineering Society (IEEE OES) ，承办方是波尔图大学。会议接收的主题包括一下几个方面：  

- Vehicle Design
- Vehicle Navigation
- Sensor Fusion、Vehicle Control
- Vehicle Planning and Execution Control
- Multi Vehicle Systems、Vehicle Applications
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

{{% figure class="center" src="/img/my-auv18/AUV.png" alt="ARMs-I" title="ARMs-I" %}}  

主要尝试去解决的一个问题是水下采集数据的效率问题。主要的想法是通过大量廉价的 AUV 集群去采集数据，为了解决 AUV 之间的通信问题，使用了无线 Mesh 网作为通信手段。


