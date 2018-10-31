
最近买了个斐讯的路由器，总结一下刷机过程。

<!--more-->

斐讯路由器 K2， 首先需要刷 bootloader ——不死 [Breed](http://www.right.com.cn/forum/thread-161906-1-1.html) 。刷入的方法可以用 [Breed Web 助手](https://weibo.com/itmfb?reason=&retcode=)通用版。刷入bootloader相当于完成一半，接下来就是进入bootloader web 控制台，方法是先拔掉电源，在断电状态下，先按住路由器上的重置按钮不放，插上电源，持续按住10秒左右，在松开按钮，等待1分钟。此时通过电脑浏览器访问192.168.1.1就能进入 breed Web 控制台界面。

进入控制台后第一步是进行固件备份，备份的内容包括 EEPROM、固件、编程器固件，这三部分备份好了就可以放心刷入固件了，以后若想用回官方系统就可以通过备份的内容恢复。

开刷前，需要首先恢复出厂设置，固件类型选择 Config 区（公版），接着就可以进入固件更新，选择固件并点击上传。确认固件MD5后就可以执行刷入了。目前使用的固件是LEDE，其下载地址如下：[连接](https://downloads.lede-project.org/releases/17.01.4/targets/ramips/mt7620/lede-17.01.4-ramips-mt7620-psg1218-squashfs-sysupgrade.bin)（psg1218 17.01.04），其他 [Release](https://downloads.lede-project.org/releases/)。

接下来就是进行路由器设置，首先第一步需要先让路由器联网，方法是将学校的网线连接到WAN口，电脑通过LAN口或者WIFI与路由器相连，在电脑上登陆web认证。
然后安装 mentohust 实现校园网自动认证（伪？？），可以直接安装 luci-i18n-mentohust-zh-cn opkg会自动处理依赖问题。关于 mentohust 的设置，填写用户名和密码，更改网卡名为eth0，DHCP方式为 1（二次认证），其他默认即可。

利用NAT6，让内网用户可以连接 IPv6。首先确认路由器可以获得 IPv6 公网地址（2开头），ping ipv6.google.com 进一步确认。然后安装 ip6tables 和 nat6 。在 网络>接口>LAN>DHCP服务器>IPv6设置>勾选总是广播默认路由。编写如下脚本自动设置：
```bash
#!/bin/sh
# filename: /etc/hotplug.d/iface/90-ipv6
# please make sure this file has permission 644

# check if it is the intf which has a public ipv6 address like "2001:da8:100d:aaaa:485c::1/64"
interface_public="wan6"
[ "$INTERFACE" = "$interface_public" ] || exit 0

res=`ip -6 route | grep "default from"`
gateway=`echo $res | awk '{print $5}'`
interface=`echo $res | awk '{print $7}'`

if [ "$ACTION" = ifup ]; then
    ip -6 r add default via $gateway dev $interface
    if !(ip6tables-save -t nat | grep -q "v6NAT"); then
        ip6tables -t nat -A POSTROUTING -o $interface -m comment --comment "v6NAT" -j MASQUERADE
    fi
else
    ip6tables -t nat -D POSTROUTING -o $interface -m comment --comment "v6NAT" -j MASQUERADE
    ip -6 r del default via $gateway dev $interface
fi
```
变量 interface_pulbic 是带有公网 IPV6 地址的接口地址。加上相应的执行权限：

```bash
chmod 644 /etc/hotplug.d/iface/90-ipv6
```

参考：

[LEDE 下的 ipv6 NAT6](https://lixingcong.github.io/2017/04/24/ipv6-nat-lede/)

[LEDE 内网转发 IPv6](https://i-meto.com/lede-ipv6/amp/)

[LEDE 配置 IPv6 NAT](https://blog.blacate.me/2017/04/09/ipv6-nat-on-openert-lede/)



