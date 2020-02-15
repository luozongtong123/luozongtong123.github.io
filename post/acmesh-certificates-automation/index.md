`acme.sh` 实现了 `acme` 协议，可以从 [Let’s Encrypt](https://letsencrypt.org/) 生成免费的证书。
<!--more-->

## 安装  

为了使用 standalone 模式，需要首先安装 socat 工具。在 standalone 模式下 acme.sh 会在 80 端口上创建一个 web 服务器进行验证，进而完成证书的签发。

``` bash
yay socat
# or 
yum install socat
```

``` bash
git clone https://github.com/acmesh-official/acme.sh.git ~/acme.sh
cd acme.sh
./acme.sh --install --accountemail "me@ztluo.dev"
```

## 证书签发  

standalone 模式下签发证书。
``` bash
acme.sh --issue --domain blog.ztluo.dev --standalone
```

http 方式，依赖已有的 Caddy 服务器。
``` bash
acme.sh --issue --domain blog.ztluo.dev --webroot /var/www/
```

## 证书安装 

``` bash
mkdir -p /etc/ssl/acmesh/blog.ztluo.dev
acme.sh --installcert  --domain blog.ztluo.dev \
        --key-file /etc/ssl/acmesh/blog.ztluo.dev/blog.ztluo.dev.key \
        --fullchain-file /etc/ssl/acmesh/blog.ztluo.dev/blog.ztluo.dev.crt \
        --reloadcmd "service nginx force-reload"
```

## 自动更新  

``` bash
acme.sh --upgrade --auto-upgrade
```

## DNS CAA  

CAA 记录可以告诉浏览器域名的证书是由谁签发的进而防止证书伪造。具体 DNS 记录的构造可以使用 [CAA Record Helper](https://sslmate.com/caa/)。填入域名后选择 “Auto-Generate”，即可以根据当前域名对应的网站自动生成相应的策略。

## 参考  
[acme.sh - 说明](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)  
[文档 - Let's Encrypt - 免费的 SSL_TLS 证书](https://letsencrypt.org/zh-cn/docs/)  
[超快速 DNS CAA 设定 Step by Step](https://cjk.aiao.today/dns-caa-setting-step-by-step/)  

