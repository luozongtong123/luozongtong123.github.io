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

## 参考  
[acme.sh - 说明](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)  


