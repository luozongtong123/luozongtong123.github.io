Caddy 号称最容易上手的 Web Server，他有一个非常显著的特点 —— 默认开启 HTTPS，自动使用 [Let’s Encrypt](https://letsencrypt.org/) 申请证书并续约。  
<!--more-->

Caddy 和我们常用的 Nginx、Apache 等 Web 服务器相比，最大的特点就是部署简单，它拥有基本的 apache 或者 nginx 有的 web server 模块，同时还有一些很有特色的功能，比如: HTTP/2、Automatic HTTPS、Multi-core、Websockets、Markdown、IPv6 等等。  

## 安装  

使用官方脚本安装：

``` bash
sudo curl https://getcaddy.com | bash -s personal http.git,dns
```

## 配置  

### 添加用户  

添加 www-data 用户组与用户便于管理 www 文件。

``` bash
sudo groupadd www-data
sudo useradd -g www-data www-data
sudo usermod -a -G root www-data
```

### 必要的文件夹  
``` bash
sudo mkdir /etc/caddy
sudo touch /etc/caddy/Caddyfile
sudo chown -R root:www-data /etc/caddy

sudo mkdir /etc/ssl/caddy
sudo chown -R root:www-data /etc/ssl/caddy
sudo chmod 0770 /etc/ssl/caddy

sudo touch /var/log/caddy.log
sudo chown root:www-data /var/log/caddy.log
sudo chmod 770 /var/log/caddy.log


# 克隆博客  
sudo mkdir /var/www
sudo chown www-data:www-data /var/www
sudo git clone https://github.com/zt-luo/zt-luo.github.io.git /var/www/blog
sudo chown -R www-data:www-data /var/www/blog
```

### 修改 Candy 配置文件  

`nano /etc/caddy/Caddyfile`  

```
www.ztluo.dev {
  gzip
  tls me@ztluo.dev
  root /var/www/blog
  git {
    repo https://github.com/zt-luo/zt-luo.github.io.git
    hook /webhook "blog/webhook2github"
  log /var/log/caddy.log
}
```

### 配置开机自动启动  

``` bash
sudo curl -s https://raw.githubusercontent.com/mholt/caddy/master/dist/init/linux-systemd/caddy.service -o /etc/systemd/system/caddy.service   # 从 github 下载 systemd 配置文件
sudo systemctl daemon-reload        # 重新加载 systemd 配置
sudo systemctl enable caddy.service # 设置 caddy 服务自启动
sudo systemctl status caddy.service # 查看 caddy 状态
```

### 启动 Caddy  

``` bash
# 启动
sudo systemctl start caddy.service

# 查看状态
sudo systemctl status caddy.service
```

如果看到 `caddy[xxxx]: Serving HTTPS on port 443` 说明成功启动，如果没有启动则根据错误提示查找问题。

## 参考  

[Caddy 最容易上手的 Web Server - 自动化 HTTPS 一分钟部署网站 \ 网盘](https://wzfou.com/caddy/)  
[systemd Service Unit for Caddy](https://github.com/caddyserver/caddy/tree/master/dist/init/linux-systemd)  

