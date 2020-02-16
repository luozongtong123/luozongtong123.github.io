`acme.sh` 实现了 `acme` 协议，可以从 [Let’s Encrypt](https://letsencrypt.org/) 生成免费的证书。
<!--more-->

## 准备用户和文件夹
创建 certusers 用户组，该用户组成员可以访问证书文件：

``` bash
sudo groupadd certusers
sudo useradd -m -r -s /bin/bash -g certusers acme
sudo mkdir -p /usr/local/etc/certfiles/acmesh
sudo chown -R acme:certusers /usr/local/etc/certfiles/acmesh/
```

## 安装  

为了使用 standalone 模式，需要首先安装 socat 工具。在 standalone 模式下 acme.sh 会在 80 端口上创建一个 web 服务器进行验证，进而完成证书的签发。

``` bash
yay socat
# or 
yum install socat cronie
```

切换到 acme 账户：
``` bash
sudo su -l acme
```

``` bash
git clone https://github.com/acmesh-official/acme.sh.git
cd acme.sh
./acme.sh --install --accountemail "me@ztluo.dev"
```

## 证书签发  

standalone 模式下签发证书。

``` bash
# 首先在有 root 权限的账户下赋予 socat 监听 80 端口的权限
sudo setcap CAP_NET_BIND_SERVICE=+eip /usr/bin/socat
```

``` bash
acme.sh --issue --domain blog.ztluo.dev --standalone
```

## 证书安装 

``` bash
mkdir -p /usr/local/etc/certfiles/acmesh/blog.ztluo.dev
acme.sh --installcert  --domain blog.ztluo.dev \
        --key-file /usr/local/etc/certfiles/acmesh/blog.ztluo.dev/blog.ztluo.dev.key \
        --fullchain-file /usr/local/etc/certfiles/acmesh/blog.ztluo.dev/blog.ztluo.dev.crt \
        --reloadcmd "service nginx force-reload"
# 允许同组用户读取私钥
sudo chmod g+r /usr/local/etc/certfiles/acmesh/blog.ztluo.dev/blog.ztluo.dev.key
```

## 自动更新  

``` bash
acme.sh --upgrade --auto-upgrade
```

## DNS CAA  

CAA 记录可以防止伪造证书，其基本原理是当用户在证书签发机构为其域名申请证书时，证书签发机构会检查相应域名的 CAA 记录，如果记录中不信任当前证书签发机构则拒绝签发证书。具体 DNS 记录的构造可以使用 [CAA Record Helper](https://sslmate.com/caa/)。填入域名后选择 “Auto-Generate”，即可以根据当前域名对应的网站自动生成相应的策略。

## 参考  
[acme.sh - 说明](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)  
[文档 - Let's Encrypt - 免费的 SSL_TLS 证书](https://letsencrypt.org/zh-cn/docs/)  
[超快速 DNS CAA 设定 Step by Step](https://cjk.aiao.today/dns-caa-setting-step-by-step/)  

