最近在翻 《C 程序设计新思维（第2版）》这本书的时候，看到了 GLib 这么一个现代化的 C 语言第三方库，
一时惊为天人，想着这不就是我一直想要的 C 语言的跨平台的库了吗。所以就花了一些时间研究了一下。  
<!--more-->

# 简介

GLib 是一个在 Linux 平台下应用非常广泛的 C 语言函数库。GLib 库具有很好的可移植性和实用性。 GLib 是
Gtk+ 的基础库，最早是 Gtk+ 的一部分。GLib 可以在多个平台下使用，比如 Linux、Unix、Windows等。
GLib 由基础类型、核心应用支持、实用功能、数据类型和对象系统五个部分组成。其中核心应用支持包括线程、线程池、
同步队列、内存管理、 socket 等的支持。实用工具包括字符串函数、数据校验、定时器、测试框架等功能。GLib 功能
非常丰富，可以称的上 C 语言的 Boost 库了。  

# 依赖

GLib 的依赖很少。其中一个是 [GNU libiconv library](http://www.gnu.org/software/libiconv/)，这个库是用来
转换文字编码用的，如果你的系统提供 `iconv()` 函数的话，那么这个库则不是必须的。另一个库是 [GNU gettext package](http://www.gnu.org/software/gettext)，如果你的系统提供 `gettext()` 这个函数的话，那么这个库也不是必须的。
除了以上两个库以外还要求系统提供多线程的实现。基本上任何一个现代操作系统都会提供多线程的实现。GLib 支
持 `POSIX threads` 和 `win32 threads`。  

# 安装

Linux 上的安装就不必多讲了，直接上包管理器就好了。对于 Windows 而言有多种选择，最方便的方法是
用 [msys2](https://www.msys2.org/)，直接在 MINGW64 或 MINGW32 环境中安装 GLib 库和 pkg-config 即可。
如果不想安装 msys2，也可以自己使用 Visual Studio 编译库，具体编译的方法就不多说了，Google 一下即可。
如果既不想自己编译有不想安装 msys2 ，那么我这有一个比较好的方法。
首先，你需要一个 gcc 编译器，推荐使用 [scoop](/tags/scoop/) 安装。然后到 [msys2](https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/) 清华镜像源上下载编译好的 GLib 库，直达连接如下 [GLib binary](https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64/mingw-w64-x86_64-glib2-2.58.1-1-any.pkg.tar.xz)。
将压缩包解压到合适的位置即可。  

# 编译

**如果你是 Linux 或者使用 msys2**，那么编译使用 GLib 的程序就很简单了。只需要根据情况添加如下编译选项即可：

``` bash
`pkg-config --cflags --libs glib-2.0`               # 只使用了 glib
`pkg-config --cflags --libs gobject-2.0`            # 使用了 gobject
`pkg-config --cflags --libs gmodule-2.0`            # 使用了 gmodule
`pkg-config --cflags --libs gmodule-no-export-2.0`  #- -export-dynamic
```

例如，如果你想编译一个如下一个 hello word 程序：

``` c
/**
 * @file hello_world.c
 * @author Zongtong Luo (luozongtong123@163.com)
 * @brief  
 * @version 0.1
 * @date 2018-10-16
 * 
 * @copyright Copyright (c) 2018
 *  
 */

#include <stdio.h>

#include "glib.h"
#include "glib/gprintf.h"


/**
 * @brief  
 *  
 * @param parameter_size  
 * @param parameter  
 * @return int  
 */
int main(int argc, char* argv[])
{
    gchar ch[128];
    g_sprintf(ch, "hello world");
    g_printf(g_strup(ch));
    g_print("\n");
    g_printf(g_strdown(ch));
    g_print("\n");

    gint i = 0;
    gchar chch[2] = {0x00};
    chch[0] = ch[i];

    while(0x00 != chch[0])
    {
        g_printf(chch);
        i++;
        chch[0] = ch[i];
    }

    g_print("\n");
    g_print("\n");

    return 0;
}
```

只需要在 shell 里面输入下面一行命令即可：

``` bash
gcc hello_world.c `pkg-config --cflags --libs glib-2.0` -o hello_world -g -Wall -O0
```

上面的命令中利用了 shell 的反引号特性。被反引号包围的命令会在执行整条指令之前先执行，并将执行的结果嵌入到
整条命令中。反引号在 shell 中被广泛支持，除了反引号外，还可以使用 `$(...)`，但是需要注意的是 `$(...)` 格式受到 POSIX 标准支持，但是并不是所有 shell 都支持。  

**如果你是自己编译 GLib 或者下载别人编译好的 GLib，**那么你可以直接将 GLib 添加到你的工程项目里。  
编译时需要手动添加 include 目录和链接的 lib 目录和 lib 的名字。下面给出一个 CMake 的例子。

``` cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0.0)
project(hello_world VERSION 0.1.0)

include_directories("glib/include/glib-2.0" "glib/lib/glib-2.0/include")
link_directories("glib/bin")
link_libraries("libglib-2.0-0.dll" "libgio-2.0-0.dll")

# 用来关闭 gcc 编译器的 DEPRECATION_WARNINGS
add_compile_options(-DGLIB_DISABLE_DEPRECATION_WARNINGS)

add_executable(hello_world  hello_world .c)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```

# 运行

同样，如果你是 Linux 或者使用 msys2 的用户，那么这一步就简单的很 ———— 直接跳过，唯一需要注意的是，如果你需要将
编译好的程序分发到没有安装 GLib 库的用户哪里的话，你需要拷贝程序依赖的动态链接库（使用了动态链接库）。  

如果你是将 GLib 库直接添加到了项目中，你的程序在调试的运行的时候会报缺少库的错误。解决这个问题只需要将需要的库
拷贝到程序的当前目录即可。除了你编译程序时手动添加的链接库，还有可能需要如下两个动态链接
库 `libiconv-2.dll` 和 `libintl-8.dll`。这两个库在之前的依赖那部分谈到过。我使用的是从 msys2 上下载的编译好
的二进制文件，程序运行时提示需要这两个库，如果你也需要这两个库只需要将这两个文件从 msys2 中拷贝出来即可。  

>另外，  
>文中提到的那本书的译者是 [赵岩](http://zhaoyan.website/blog/)，近期在看他写的另一本书 《C语言点滴》。
>这本书的语言非常生动活泼，看起来非常有趣，同时作为一本 2012 年写的书来讲一点都不过时。等书看完了会写一篇读书笔记。  
** **
>另另外，  
>文中 GLib 会新开一个专题，专门记录关于 GLib 学习的内容。除了 GLib 文中提到的 scoop ，也需要写一篇博客。
>感觉自己开了一万个新坑，上次 RT-thread 的内容才更新了一章。  
>{{% figure class="center" src="/img/glib-intro/smile.jpg" alt="smile" title="Smile" %}}  

# 参考资料

1. [GLib Reference Manual](https://developer.gnome.org/glib/)  
2. [浅析 GLib](https://www.ibm.com/developerworks/cn/linux/l-glib/index.html)  
3. [Linux：shell 脚本之命令替换（eval，反引号和 $()）](https://blog.csdn.net/if9600/article/details/74221548)  

