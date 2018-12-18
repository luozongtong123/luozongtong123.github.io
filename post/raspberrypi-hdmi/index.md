记录一下解决树莓派连接显示器黑屏的问题。  
<!--more-->

**树莓派版本：**3  
**显示器：**  4:3的长城显示器，通过 HDMI 转 VGA 的转接头连接。  
**问题描述**：树莓派通过 HDMI 接口连接显示器。只有第一次成功显示，之后再也无法显示。第一次显示的时候也有部分屏幕无法显示。  
**解决方法：**  
**修改 SD 卡 boot 分区的 config.txt 文件。**  
将 SD 卡插入到电脑中，首先备份 config.txt 文件，然后使用编辑器打开，这里不推荐记事本，可以使用 notepad++ 打开。然后加入以下几个选项：  

```
hdmi_force_hotplug=1
config_hdmi_boost=4
hdmi_group=2
hdmi_mode=35
hdmi_drive=2
hdmi_ignore_edid=0xa5000080
disable_overscan=1      
```

这里我的显示器分辨率是 1280*1024，所以修改为 hdmi_mode=35，如果你的显示器分辨率不同，请查找需要修改的值。


参考：  
[解决树莓派使用 HDMI-VGA 转换器黑屏的方案](http://blog.lxx1.com/1706)  
[CONFIG.TXT](https://www.raspberrypi.org/documentation/configuration/config-txt/README.md)  
[VIDEO OPTIONS IN CONFIG.TXT](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md)  

