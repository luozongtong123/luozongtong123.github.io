Windows 上的命令行工具 cmd 是一个有三十多年历史的古老程序。在 Windows 上 cmd 的竞争者的是 PowerShell，
它功能更强大，用起来更方便，早在其版本号为 1.0 的时候我就尝试过，然而其异常缓慢的冷启动速度让人无法接受，
尝试了几次后便无奈放弃。最近，一个偶然的机会，了解到了 [Scoop](https://scoop.sh/) 和 [PowerShell Core](https://github.com/PowerShell/PowerShell)。一番研究之后，最终放弃了 cmd ，开始使用 PowerShell。  

<!--more-->  

# PowerShell Core

在说 Scoop 之前先说一说这个 PowerShell。经过几年的开发，Windows PowerShell 的版本号已经更新到 5.1 了。
中所周知 PowerShell 最初是基于 .NET Framework 的，只能在 Wondows 上使用的。随着微软的开始拥抱开源，
Windows 独占的 .NET Framework 发展 ~~变~~ 成了跨平台的 .NET Core。(.NET Core 是 .NET Framework 的子集)
PowerShell 也顺理成章地变成了开源版本的 PowerShell Core 了。
~~从此，PowerShell Core 不再依赖 .NET Framework 了。
PowerShell 变成了可以跨平台运行在 Windows、Linux 和 macOS 上的工具了。~~
Github 上下载来尝试了一下，启动速度明显变快了。  

# 引子

小前提讲完了，现在来讲一讲最终让我大喊 “真香！”的原因 —— [Scoop](https://scoop.sh/)。  

****
Scoop 是什么？  
是一个软件安装工具。  

Scoop 不是什么？  
不是包管理工具。  
****

Scoop 看以来像一个包管理工具。其实 Scoop 这款工具的定位仅仅是一个程序安装工具，因为她处理不了复杂的依赖关系。
每个被安装的程序仅仅是一个个独立的软件，不会像包管理工具那样产生相互依赖。说起来包管理工具，不得不提 Chocolatey。
Chocolatey 我也尝试过，但是她有许多我不能够忍的缺点：不能选择安装位置、需要管理员权限、UAC 弹窗。
具体二者有何区别详见 [Chocolatey Comparison](https://github.com/lukesampson/scoop/wiki/Chocolatey-Comparison)。

在正式开始之前我们先看一下 Scoop 长啥样：

``` shell
PS C:\Users\luo> scoop
Usage: scoop <command> [<args>]

Some useful commands are:

add         installs an app
alias       Manage scoop aliases
bucket      Manage Scoop buckets
cache       Show or clear the download cache
checkup     Check for potential problems
cleanup     Cleanup apps by removing old versions
config      Get or set configuration values
create      Create a custom app manifest
depends     List dependencies for an app
export      Exports (an importable) list of installed apps
help        Show help for a command
home        Opens the app homepage
info        Display information about an app
install     Install apps
list        List installed apps
prefix      Returns the path to the specified app
reset       Reset an app to resolve conflicts
rm          Uninstalls an app
search      Search available apps
status      Show status and check for new app versions
uninstall   Uninstall an app
update      Update apps, or Scoop itself
upgrade     Updates all apps, just like brew or apt
virustotal  Look for app's hash on virustotal.com
which       Locate a shim/executable (similar to 'which' on Linux)
```

要安装、卸载软件:

``` shell
scoop install <app>
scoop uninstall <app>
```

要更新软件：

``` shell
# 更新 app
scoop update <app>  
# 更新 scoop
scoop update
# 更新所有软件包
scoop update *
```

怎么样是不是心动了。使用 Scoop 甚至都不需要帮助文档，直接就可以上手了。  

# 安装

开始之前。  
虽然，Windows10 已经内置有 PowerShell，但我还是建议你安装一个 [PowerShell Core](https://github.com/PowerShell/PowerShell/releases)。  
提示，Scoop 需要最低版本号为 3 的 PowerShell。

首先，在 PowerShell 里修改一下安全策略：

``` shell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
```

然后，输入以下命令安装：

``` shell
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```

Scoop 默认的安装位置是 `C:\Users\username\Scoop\`。  
或者，你想修改一下 Scoop 默认的安装位置。在安装 Scoop 之前输入以下命令：

``` shell
# 修改用户默认安装位置
# 注意去掉 [environment] 后的两个 \
[environment]\:\:setEnvironmentVariable('SCOOP','D:\Applications\Scoop','User')
$env:SCOOP='D:\Applications\Scoop'
# 用户级安装 app
scoop install <app>

# 修改全局安装位置
[environment]\:\:setEnvironmentVariable('SCOOP_GLOBAL','F:\GlobalScoopApps','Machine')
$env:SCOOP_GLOBAL='F:\GlobalScoopApps'
# 全局安装 app
scoop install -g <app>
```

# 使用

Scoop 一共就没几个命令，输入 `scoop help`，看一下即可。  

Scoop 里有两个重要的概念，`bucket` 和 `manifest`。`bucket` 是一个描述一个 app 安装方法的 `JSON` 文件。
一个简单的例子如下：

``` json
{
    "homepage": "http://gnuwin32.sourceforge.net/packages/sed.htm",
    "version": "4.2.1",
    "license": "GPL-2.0",
    "url": [
        "https://sourceforge.net/projects/gnuwin32/files/sed/4.2.1/sed-4.2.1-bin.zip",
        "https://sourceforge.net/projects/gnuwin32/files/sed/4.2.1/sed-4.2.1-dep.zip"
    ],
    "hash": [
        "ae1651ffb461d69a7c51c76867a859d8addf802b85678834c0ebb402ef79b3cd",
        "53e2e84db3e6c5855fd8013e1b79ccbd53b6bae7e7e4d59c0d18bb1dd5d18961"
    ],
    "bin": "bin\\sed.exe"
}
```

而，`bucket` 则是一些 `manifest` 的集合。Scoop 默认指开启了主 `bucket`，你可以通过以下命令添加其他官方提供的“桶”。

``` shell
scoop bucket known
scoop bucket add extras
scoop bucket add versions
```

# 进阶

Scoop 简洁的设计令你可以非常方便自定义你自己的软件源（buckets）。
下面给出一个自定义软件源的示例。  

首先，你需要一个 git 仓库，可以是公开的也可以是私有的。我在 Gihub 上见了一个我自急用的仓库，[直达](https://github.com/luozongtong123/bucket-luo)。
然后，将这个仓库克隆到本地。按照官方 [wiki](https://github.com/lukesampson/scoop/wiki/App-Manifests) 
参考已有的 `manifest` 编写自己的 `manifest`。下面给出我自己用的两个 `manifest`。  

glib2.json  

``` json
{
    "homepage": "http://www.gtk.org/",
    "version": "2.58.1",
    "license": "LGPL-2.1",
    "url": [
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64/mingw-w64-x86_64-glib2-2.58.1-1-any.pkg.tar.xz",
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64/mingw-w64-x86_64-pcre-8.42-1-any.pkg.tar.xz",
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64/mingw-w64-x86_64-libffi-3.2.1-4-any.pkg.tar.xz"
    ],
    "hash": [
    "a7e00be7ddd00d1a8a5bc7853e1ecdd4f880c6fb26992c0ac84a3b47ed79e1dc",
    "884c91261bec60e379b1f8ea29f35285e7e7066483b9ce819c047d0a0aec5ca8",
    "d80826c868e8cb016fe607e8919883e6062162f6813fe8cf138273cf8b136d14"
    ],
    "extract_dir": [
    "mingw64",
    "mingw64",
    "mingw64"
    ],
    "env_add_path": "bin",
    "env_set": {
    "PKG_CONFIG_PATH": "$dir\\lib\\pkgconfig"
    },
    "notes": "Only 64-bit version is provided.",
}
```

gtester.json  

``` json
{
    "homepage": "http://www.gtk.org/",
    "version": "2.54.3",
    "license": "LGPL-2.1",
    "url": [
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/x86_64/glib2-devel-2.54.3-1-x86_64.pkg.tar.xz",
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/x86_64/glib2-2.54.3-1-x86_64.pkg.tar.xz",
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/x86_64/msys2-runtime-2.11.1-2-x86_64.pkg.tar.xz",
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/x86_64/libiconv-1.15-1-x86_64.pkg.tar.xz",
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/x86_64/libintl-0.19.8.1-1-x86_64.pkg.tar.xz",
    "https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/x86_64/libpcre-8.42-1-x86_64.pkg.tar.xz"
    ],
    "hash": [
    "9f9c9963bb74a815f94679c1116e1487695e1629c7c9c9c5a42fc266d9599a05",
    "0107bfbed3ff4ddb6352b6e2c56755267fbd0711148716e8e7fbab848b688e1b",
    "ec5f37edcc7e0284770b46e8c069d6d82a3c0eefe5bd39cd8052298b85d76c82",
    "992219dc1476c352cafe9a842835a4a5492e8f896f07e996cc50e5435e0c3c5e",
    "5eadc3cc42da78948d65d994f1f8326706afe011f28e2e5bd0872a37612072d2",
    "78fd9f9a0266b14b73264dfe8ff6cc2970bc06ccfa8f7dd314efa5103b09f4eb"
    ],
    "extract_dir": [
    "usr\\bin",
    "usr\\bin",
    "usr\\bin",
    "usr\\bin",
    "usr\\bin",
    "usr\\bin"
    ],
    "bin": "gtester.exe",
    "notes": "Only 64-bit version is provided."
}
```

编写完成后 commit 并 push 到远端仓库。  

使用如下命令添加自定义的 `bucket`。  

``` shell
scoop buckets add <bucket> url/tu/your/repo.git
scoop install <bucket>/<app>
```

然后就可以愉快的使用了。  

**注意：如果自定义仓库和自带的主仓库有重名的 app，Scoop 会默认安装主仓库的 app，若想安装自定义仓库的 app 则需要指明仓库的名字。**

>上面提到的两个 app 是在 [GLib 简介](/post/glib-intro/) 中提到的 glib2 库和该库自带测试框架的一个测试 app。
>可能过后会开一个新坑介绍以下这两个 `manifest` 的详细制作方法。

# 参考

1. [Scoop Wiki](https://github.com/lukesampson/scoop/wiki)  
2. [再谈谈 Scoop 这个 Windows 下的软件包管理器](https://h404bi.com/blog/2018/05/12/talk-about-scoop-the-package-manager-for-windows-again.html#fn4)  
3. [用 Scoop 改善 Windows Powershell](https://h404bi.com/blog/2015/08/23/use-scoop-to-enhance-windows-powershell.html)  
4. [Scoop - A command-line installer for Windows](https://scoop.sh/)  
