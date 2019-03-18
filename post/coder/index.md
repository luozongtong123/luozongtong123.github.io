Coder 是一个基于 Visual Studio Code 的开源的远程开发环境。
<!--more-->
# 缘起  
在 [即刻](https://web.okjike.com/) 的 [Jithub](https://web.okjike.com/topic/55e02198dcef9f0e00d7b3c3/official) 话题下看到 [code-server](https://github.com/codercom/code-server) 这个每日流行项目。觉得有趣就去了解了一下。code-server 利用 Docker 提供远程的 Visual Studio Code 服务，只需要一个浏览器就能使用 VSCode。自己尝试着在 Windows 10 上的 WSL(Ubuntu 16.04) 上搭建，结果没有成功。不知道是不是 WSL 的问题。思量再三，决定先放弃在 WSL 上尝试的想法。转而去尝试官方提供的在线的服务 [Coder](https://coder.com/signup)。  

# 前生  
在此之前也尝试过几个在线的 IDE 。最早是 [Cloud9 IDE](https://c9.io/login)，后来也试过 [Coding](https://coding.net/) 以及后来的腾讯云开发者平台的 [Cloud Studio](https://studio.dev.tencent.com/) 的在线 IDE 服务。试来试去总有种隔靴挠痒的感觉，性能和体验并不好， Cloud9 还是尝试用过一段时间， Coding 的就只是尝试了几下就放弃了。直到看到了 code-server，我知道，这才是我想要的在线 IDE。这三个老大哥的问题是性能体验不好同时还缺乏 Visual Studio Code 多到爆炸的扩展和用户群群体。我不知道这个公司提供的在线服务会不会大火，但我觉得 code-server 这个项目肯定会爆发一阵热潮。  

另外，在 Windows 上使用 Visual Studio Code 结合 WSL 编程一直是一个让人头疼的问题，虽然 VSCode 团队也在跟进相关的功能，但是进展一直不是很理想不知道什么时候能够用得上[#63155](https://github.com/Microsoft/vscode/issues/63155)。或许 code-server 是一个不错的解决方案。

# 今世  
code-server 最早在 GitHub 上的一次 Release 是 6 天前，目前基于的是 Visual Studio Code 1.31.1。项目还处在早期发展的阶段。试用 Coder 的在线服务，发现也有一些问题。接下来就把调教过程记录一下。

## 登陆和注册  
[注册地址在这](https://coder.com/signup)。官方提供了 Github 和 Google 的快速登陆，非常贴心。为了防止服务被滥用需要验证你的手机，一个手机号只能注册一个账号。由于注册登陆过程过于简单，具体操作就不说了。  

{{% figure class="center" src="/img/coder/coder-welcome.png" alt="coder welcome" title="coder welcome" %}}

是这熟悉的画面。右下角可以看到 CPU 、内存和硬盘占用。~~正常模式下内存只有 1GB，在左边那栏的闪电可以开启急速模式，开启后内存变为 16GB，CPU 和网速也变得更强了，可能是现在是公开测试阶段，急速模式好像还没有时间限制~~ 急速模式的开关去掉了。硬盘提供了 3GB，硬盘空间有点小。

## 安装必要的软件  

``` bash
# 看一下系统的情况 
cat /etc/issue
Ubuntu 16.04.5 LTS \n \l

uname -a
Linux coder 4.18.4-041804-generic #201808220230 SMP Wed Aug 22 06:33:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdbf       3.0G  620M  2.2G  22% /
```

更新一下系统。
``` bash
# 首先更新一下系统
apt update
apt upgrade
```
更新系统的时候会出现如下的提示：
``` shell
$ apt upgrade
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
        LANGUAGE = (unset),
        LC_ALL = (unset),
        LANG = "zh_CN.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
```
这是由于 Coder 根据浏览器的设置将容器的 locales 设置成了 `zh_CN.UTF-8`，而默认的容器中不包含 `zh_CN.UTF-8` 所以我们需要自己生成或者是该成其他的。设置的方法如下：
``` bash
# fix locales
apt install locales
cat /etc/default/locale # 查看默认的 locale 设置
locale-gen zh_CN.UTF-8 en_US.UTF-8 # 生成 zh_CN.UTF-8 和 en_US.UTF-8

# 设置系统默认 locale
update-locale LANG=en_US.UTF-8
# update-locale LANG=zh_CN.UTF-8

# 更改用户 locale
nano ~/.bashrc
# 添加如下一行即可
export LANG=en_US.UTF-8
# export LANG=zh_CN.UTF-8
```

## 安装一些扩展  

安装一些常用的 C 语言和 Python 的扩展。

## 更改字体

甚至可以换成 `Fira Code` 字体。

# 实际体验  

~~实际体验起来真的和本地的 Visual Studio Code 没什么两样，真的是太强了。~~

目前存在的缺陷：  
- 可以安装简体中文语言包，但是无法切换到简体中文；  
- 无法使用 settings sync，配置同步是个问题；  
- terminal 字体好像无法更改；  
- ~~无法退出登录，想要退出当前账号只能清除浏览器的 cookie；~~ 是有 logout 选项的，在 File 选项下；  
- 无法打开工作区；  
- 一些奇怪的问题，比如无法运行 lsb-release；  
- 磁盘空间只有 3GB，而且只有 2.2GB 可用，太小了；  
- 没有实现 markdown 预览；  

------  
update: 2019.03.16  
终于可以重置容器了，硬盘空间太小，有一个账号不小心被撑爆了。可以一下还原啦~  


