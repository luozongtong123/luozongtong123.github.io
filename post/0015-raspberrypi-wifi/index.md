树莓派连接 Wifi，HOW。  
<!--more-->

# 修改 /etc/network/interfaces 文件  
添加或将原来的设置替换成如下所示：  

```
iface wlan0 inet dhcp
wpa_conf /etc/wpa_supplicant/wpa_supplicant.conf
```

# 修改 /etc/wpa_supplicant/wpa_supplicant.conf 文件  
按照如下格式添加无线网络配置：
**最常用的配置。WPA-PSK 加密方式。**

```
network={
ssid="WiFi-name1"
psk="WiFi-password1"
priority=5
}
 
network={
ssid="WiFi-name2"
psk="WiFi-password2"
priority=4
}
```

优先级数字越大优先级越高。

# 启用WiFi    
``` bash
sudo ifup wlan0
```
注意： 如果本来已经启用WiFi，可能需要先停用然后再次启用才可以连接WiFi。

# 停用WiFi  
``` bash
sudo ifdown wlan0
```

# ~~自动重连WiFi~~  
`可能没有作用`
``` bash
git clone https://github.com/zt-luo/WiFi_Check.git  
cd WiFi_Check
chmod +x WiFi_Check
sudo nano /etc/crontab
# 添加如下两行
*/1 * * * * root /home/pi/WiFi_Check/WiFi_Check
*/1 * * * * root /home/pi/WiFi_Check/WiFi_Check1
```

参考：  
[树莓派连接 WiFi（最稳定的方法）](https://www.cmgine.com/archives/11053.html)  
[树莓派 3 代从 0 到连接 WIFI 访问（无显示器）](http://movesan.me/2017/02/07/raspberry/)  


