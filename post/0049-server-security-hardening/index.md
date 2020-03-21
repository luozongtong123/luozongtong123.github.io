前几天把博客迁移到了服务器上，这相当于把服务器完全暴露在了公开的互联网上了，为了保证服务器的安全，一些安全加固的措施是非常有必要的。  
<!--more-->

## 创建非 root 用户  

首先第一件事就是要创建非 root 用户，使用 root 用户进行日常的操作是非常危险的。

``` bash
sudo useradd -m -r -s /bin/bash user
sudo passwd user

# 为用户添加 sudo 权限
sudo usermod -aG sudo user 
# 或
sudo usermod -aG wheel user 
# 或
sudo tee /etc/sudoers.d/user <<< 'user ALL=(ALL) ALL'
sudo chmod 440 /etc/sudoers.d/user

# 切换到 user
su -l user
```

## 防止密码被爆破  

为了防止密码被爆破限制登录密码错误 3 次后锁定账户。

``` bash
sudo nano /etc/pam.d/login
sudo nano /etc/pam.d/remote
sudo nano /etc/pam.d/sshd
# 在 #%PAM-1.0 下添加下面一行
auth required pam_tally2.so deny=3 unlock_time=300 even_deny_root root_unlock_time=600
```

## 远程登录加固  

使用 `sudo nano /etc/ssh/sshd_config` 编辑 sshd 配置文件。

使用私钥远程登录服务器：  

``` bash
# 生成私钥
ssh_keygen
# 添加公钥  
echo <主机 id_rsa.pub> >> .ssh/authorized_keys
# 修改权限  
chmod 600 .ssh/authorized_keys
```

1. 修改默认端口号：默认端口号 22，默认的端口号会导致过多登录尝试，将其改为不常用的端口，选项名称为 `Port`；
2. 禁止空口令登录：`PermitEmptyPasswords no`；
3. 禁用密码登录：`PasswordAuthentication no`，要注意一定要在确认可以通过私钥登录后才修改此项配置，否则会导致无法远程登录。

## 创建服务账户  

每个服务使用单独自己的账户，这样就不会因为某个服务遭到攻破而导致 root 权限被拿到。

创建带有 home 目录的 acme 帐号：

``` bash
sudo useradd -m -r -s /bin/bash acme
sudo su -l acme
```

创建不带有 home 目录的 acme 帐号：
``` bash
sudo useradd -r -s /bin/bash acme
sudo su -l acme
```

## 配置防火墙  

本文以 ufw 为例配置防火墙，ufw 是一个很好用的防火墙配置命令，可以简化操作，减少错误的发生。  

``` bash
# 安装
# Ubuntu
sudo apt install ufw
# CentOS
sudo yum install epel-release
sudo yum install ufw
```

执行如下命令即可成功配置防火墙。注意，如果 ssh 端口不是 22 那么第一行需要调整（将 ssh 改为端口号）。

``` bash
# 关闭防火墙
sudo ufw disable
# 防火墙默认阻止
sudo ufw default deny
# 创建 允许某些服务/端口 的规则
sudo ufw allow ssh
sudo ufw allow https
sudo ufw allow http
sudo ufw allow 8888
# 创建 组织某些服务/端口 的规则
sudo ufw deny http
sudo ufw deny 8888
# 启动防火墙
sudo ufw enable
# 查看防火墙状态
sudo ufw status
sudo ufw status verbose
# 删除某个防火墙规则
sudo ufw delete allow http
# 开启 log
sudo ufw logging low
```
