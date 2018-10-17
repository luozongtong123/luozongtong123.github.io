最近在翻 《C 程序设计新思维（第2版）》这本书的时候，看到了 Glib 这么一个现代化的 C 语言第三方库，
一时惊为天人，想着这不就是我一直想要的 C 语言的跨平台的库了吗。所以就花了一些时间研究了一下。  
<!--more-->

# 简介

Glib 是一个在 Linux 平台下应用非常广泛的 C 语言函数库。Glib 库具有很好的可移植性和实用性。 Glib 是
Gtk+ 的基础库，最早是 Gtk+ 的一部分。Glib 可以在多个平台下使用，比如 Linux、Unix、Windows等。
GLib 由基础类型、核心应用支持、实用功能、数据类型和对象系统五个部分组成。其中核心应用支持包括线程、线程池、
同步队列、内存管理、 socket 等的支持。实用工具包括字符串函数、数据校验、定时器、测试框架等功能。Glib 功能
非常丰富，可以称的上 C 语言的 Boost 库了。  

# 依赖

Glib 的依赖很少。其中一个是 [GNU libiconv library](http://www.gnu.org/software/libiconv/)，这个库是用来
转换文字编码用的，如果你的系统提供 `iconv()` 函数的话，那么这个库则不是必须的。另一个库是 [GNU gettext package](http://www.gnu.org/software/gettext)，如果你的系统提供 `gettext()` 这个函数的话，那么这个库也不是必须的。
除了以上两个库以外还要求系统提供多线程的实现。基本上任何一个现代操作系统都会提供多线程的实现。Glib 支
持 `POSIX threads` 和 `win32 threads`。  

# 安装

Linux 上的安装就不必多讲了，直接上包管理器就好了。对于 Windows 而言有多种选择，最方便的方法是
用 [msys2](https://www.msys2.org/)，直接在 MINGW64 或 MINGW32 环境中安装 Glib 库和 pkg-config 即可。
如果不想安装 msys2，也可以自己使用 Visual Studio 编译库，具体编译的方法就不多说了，Google 一下即可。
如果既不想自己编译有不想安装 msys2 ，那么我这有一个比较好的方法。
首先，你需要一个 gcc 编译器，推荐使用 scoop 安装。然后到 msys2 清华镜像源上下载编译好的 Glib 库，直达连接如下 [Glib binary](https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64/mingw-w64-x86_64-libffi-3.2.1-4-any.pkg.tar.xz)。
将压缩包解压到合适的位置即可。  

# 使用

//:TODO  
