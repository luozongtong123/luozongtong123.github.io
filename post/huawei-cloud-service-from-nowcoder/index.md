活动链接：[点这里](https://www.nowcoder.com/discuss/306554?type=0&order=0&pos=2&page=1)，具体的购买方式也不多说了，具体[点这里](https://www.nowcoder.com/discuss/330402)。
<!--more-->

# 首次登录
如果在购买的时候没有设置密码，则需要在控制台中把云服务器的密码重置一下。ok，密码拿到了。现在就可以把华为云那个卡卡的网页版的控制态丢开了。直接在我的 Guake 上输入 SSH 指令登录：

``` shell
ssh root@ip
```
OK!

{{% figure class="center" src="/img/huawei-cloud-service-from-nowcoder/hw1.png" alt="SSH登录界面" title="SSH登录界面" %}}

首次登录之后最好添加一个账户，平常的操作还是不要使用 root 账户为好：
``` shell
adduser [username] #添加用户
usermod -aG sudo [username] #给用户添加sudo权限
```

添加 ssh 公钥，免密码登录:  
``` shell
cd ~
mkdir .ssh && cd .ssh
echo [rsa.pub] >> authorized_keys
```

不想记服务器 ip:  
``` shell
# 添加 ip 到 host 文件
# ip  hw
ssh [username]@hw
```

# 我们来康康这个服务器咋样
哦或，竟然自带 htop

{{% figure class="center" src="/img/huawei-cloud-service-from-nowcoder/hw2.png" alt="htop" title="htop" %}}

``` shell
uname -a
```
``` shell
Linux ecs-sn3-medium-2-linux-20191104190410 4.15.0-65-generic #74-Ubuntu SMP Tue Sep 17 17:06:04 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

9块钱的服务器，不能再香了。跑分之前我们先把服务器更新一下：

``` shell
apt update
apt upgrade
```

接下来我们来跑个分测试一下这台"9块钱"的服务器性能咋样。首先使用经典的 `UNIX BENCH` 工具来测试下，测试结果如下：

``` shell
========================================================================
   BYTE UNIX Benchmarks (Version 5.1.3)

   System: ecs-sn3-medium-2-linux-20191104190410: GNU/Linux
   OS: GNU/Linux -- 4.15.0-65-generic -- #74-Ubuntu SMP Tue Sep 17 17:06:04 UTC 2019
   Machine: x86_64 (x86_64)
   Language: en_US.utf8 (charmap="UTF-8", collate="UTF-8")
   CPU 0: Intel(R) Xeon(R) Gold 6161 CPU @ 2.20GHz (4400.0 bogomips)
          x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET
   20:40:44 up 47 min,  2 users,  load average: 0.21, 0.17, 0.13; runlevel 5

------------------------------------------------------------------------
Benchmark Run: Mon Nov 04 2019 20:40:44 - 21:08:48
1 CPU in system; running 1 parallel copy of tests

Dhrystone 2 using register variables       32340436.6 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     4610.6 MWIPS (9.6 s, 7 samples)
Execl Throughput                               4800.2 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        850784.1 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks          227052.1 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks       2472848.7 KBps  (30.0 s, 2 samples)
Pipe Throughput                             1255756.1 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 280341.4 lps   (10.0 s, 7 samples)
Process Creation                              14484.4 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   9158.0 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                   1196.3 lpm   (60.0 s, 2 samples)
System Call Overhead                         890125.7 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   32340436.6   2771.2
Double-Precision Whetstone                       55.0       4610.6    838.3
Execl Throughput                                 43.0       4800.2   1116.3
File Copy 1024 bufsize 2000 maxblocks          3960.0     850784.1   2148.4
File Copy 256 bufsize 500 maxblocks            1655.0     227052.1   1371.9
File Copy 4096 bufsize 8000 maxblocks          5800.0    2472848.7   4263.5
Pipe Throughput                               12440.0    1255756.1   1009.5
Pipe-based Context Switching                   4000.0     280341.4    700.9
Process Creation                                126.0      14484.4   1149.6
Shell Scripts (1 concurrent)                     42.4       9158.0   2159.9
Shell Scripts (8 concurrent)                      6.0       1196.3   1993.8
System Call Overhead                          15000.0     890125.7    593.4
                                                                   ========
System Benchmarks Index Score                                        1420.9

======= Script description and score comparison completed! ======= 
```
``` shell
--------------------------------------------------------------------------
CPU 型号             : Intel(R) Xeon(R) Gold 6161 CPU @ 2.20GHz
CPU 核心数           : 1
CPU 频率             : 2200.000 MHz
总硬盘大小           : 40.9 GB (6.2 GB Used)
总内存大小           : 1993 MB (252 MB Used)
SWAP大小             : 0 MB (0 MB Used)
开机时长             : 0 days, 13 hour 53 min
系统负载             : 0.00, 0.00, 0.00
系统                 : Ubuntu 18.04.3 LTS
架构                 : x86_64 (64 Bit)
内核                 : 4.15.0-65-generic
虚拟化平台           : kvm
--------------------------------------------------------------------------
硬盘I/O (第一次测试) : 199 MB/s
硬盘I/O (第二次测试) : 156 MB/s
硬盘I/O (第三次测试) : 156 MB/s
--------------------------------------------------------------------------
```

# 搞个服务器用来干嘛呢
干嘛具体还没想好--。
博客的话是搭在GiHub上的，其他的网站啥的好像也没时间搞。。。小程序也不会玩。只想到了一个用处，就是作为 VS Code 的远程主机，用来编译调试软件，这样本地主机就可以不那么卡了。。。

# 解决无法直接访问 GitHub 的问题
直接修改 hosts 文件即可。
``` shell
# root 用户
echo "192.30.253.112  github.com" >> /etc/hosts
echo "192.30.253.113  www.github.com" >> /etc/hosts
```


[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

