前几天发现了 斐讯 N1 这个好东西，买了一个拿来玩玩。记录一下折腾过程。
<!--more-->
# 刷机  

首先需要需要刷 webpad 的固件。  
https://www.mivm.cn/phicomm-n1-linux/  
https://www.zrj96.com/post-1159.html  


## 替换 dtb 解决负载不正常的问题  

替换 5.0.0 dtb  

[dtb 链接](https://github.com/yangxuan8282/phicomm-n1/issues/15#issuecomment-473663722)  
[直接下载](/zip/0032-phicomm-n1/meson-gxl-s905d-phicomm-n1.7z)

# Armbian 相关  

## 启用 bbr  

[参考](https://sb.sb/blog/debian-ubuntu-tcp-bbr/)


## 解决 sudo 慢的问题

``` bash
# 查看 hostname
hostname
# 将 hostname 添加到 host 的第一和第二行
sudo nano /etc/hosts
```

## NTP 时间自动更新

``` bash
sudo apt install ntpdate
sudo crontab -e
# 添加如下一行
00 10 * * * /usr/sbin/ntpdate -u cn.pool.ntp.org > /dev/null 2>&1
```


## oh-my-zsh

``` bash
# 安装
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# 更换主题
nano ~/.zshrc
# 修改
ZSH_THEME="ys"
source ~/.zshrc

# 添加插件
...
```

# 启用测试版软件仓库  

https://linux.cn/article-3288-1.html  

