对 Qt Creator 的界面是在是不能忍受，所以萌生了改用 Visual Studio Code 的想法，探索一番，成功实现 Qt 代码在 VSCode 中的代码提示和调试的功能。
<!--more-->

# Qmake  

Qt Creator，准确的说应该是 Qmake，使用 `.pro` 来进行项目的管理，包括设定头文件包含目录、链接库、`.ui` 文件、`.qrc` 文件、以及编译选项等等的管理。Qmake 自动替我们将`.ui` 文件预处理为 `ui_xx.h` 文件、`.qrc` 文件机文件中的资源预处理为 `qrc_xx.cpp`、将带有 `Q_OBJECT` 等 Qt 自带宏的头文件预处理为 `moc_xx.cpp`。这三个预处理过程分别由 `uic` 、`rcc`、`moc` 三个工具来完成。  

正常来说，只要在 VSCode 中编译任务中将 Qmake 命令写好就可以了，但是，由于 VSCode 中没有 Qmake 的插件，头文件的包含无法自动完成。于是考虑使用 CMake 来进行编译过程的管理。  

# CMake  

一个简单的 CMake 例子如下所示：

``` cmake
cmake_minimum_required(VERSION 3.10)

# your project name
project("SubControl")

add_compile_options(-g -O0 
                    # -Wno-unused-parameter
                    -static)
                    # -shared)

# find includes in the corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# run moc automatically
# set(CMAKE_AUTOMOC ON)
# run uic automatically
# set(CMAKE_AUTOUIC ON)
# run rcc automatically
# set (CMAKE_AUTORCC ON)

message("Looking for Qt...")
# Qt modules (https://doc.qt.io/qt-5/qtmodules.html) you're using in your application
find_package(Qt5 REQUIRED Widgets Core Gui Charts Gamepad)
if (${Qt5_FOUND})
    message("Found Qt " ${Qt5_VERSION})
else()
    message("Couldn't find Qt")
endif()

link_libraries(
    Qt5::Core
    Qt5::Widgets
    Qt5::Gui
    Qt5::Charts
    Qt5::Gamepad
    )

qt5_wrap_ui(ui_list 
    "ui/mainwindow.ui"
    "ui/videowindow.ui"
    )

qt5_wrap_cpp(moc_list
    "inc/chart.h"
    "inc/mainwindow.h"
    "inc/qFlightInstruments.h"
    "inc/videowindow.h"
)

qt5_add_resources(qrc_list 
    "assets.qrc")

# check and search moudles
find_package(PkgConfig REQUIRED)
pkg_search_module(GLIB REQUIRED glib-2.0)
pkg_search_module(LIBSERIALPORT REQUIRED libserialport)
pkg_search_module(LIBVLC_CORE REQUIRED libVLCQtCore)
pkg_search_module(LIBVLC_WIDGETS REQUIRED libVLCQtWidgets)

include_directories(
    ${GLIB_INCLUDE_DIRS} ${LIBSERIALPORT_INCLUDE_DIRS} 
    ${LIBVL_CCORE_INCLUDE_DIRS} ${LIBVLC_WIDGETS_INCLUDE_DIRS} 
    )

link_libraries(
    ${GLIB_LDFLAGS} ${LIBSERIALPORT_LDFLAGS} 
    ${LIBVLC_CORE_LDFLAGS} ${LIBVLC_WIDGETS_LDFLAGS} 
    )

ADD_SUBDIRECTORY(ardusub_api)
link_libraries(ardusub_static)
# link_libraries(ardusub_shared)

# add include dir
include_directories("ardusub_api/mavlink_c_library_v2")
include_directories("inc")

# your source files
set(src_list
    "src/chart.cpp"
    "src/control.cpp"
    "src/joystick.cpp"
    "src/main.cpp"
    "src/mainwindow.cpp"
    "src/qFlightInstruments.cpp"
    "src/timer.cpp"
    "src/video.cpp"
    "src/videowindow.cpp"
)

# name of the .exe file, window flag and the list of things to compile
# add_executable(${CMAKE_PROJECT_NAME} WIN32 ${sources})
add_executable(${CMAKE_PROJECT_NAME} ${src_list} ${ui_list} ${moc_list} ${qrc_list})
```
> CMakeLists.txt 来自 [SubControl](https://github.com/zt-luo/SubControl/blob/master/CMakeLists.txt) 项目。  

`set(CMAKE_AUTOMOC ON)` `set(CMAKE_AUTOUIC ON)` `set (CMAKE_AUTORCC ON)` 可以打开相应预处理的自动处理开关。不过，自动处理的前提是这些文件需要和源文件在同一个目录下。如果不在一个目录下的话只能通过 `qt5_wrap_ui` `qt5_wrap_cpp` `qt5_add_resources` 来完成预处理的配置。  


{{% admonition important %}}

Qt 模块的第一个字母需要大写。

{{% /admonition %}}

# Visual Studio Code

其实到了这一步基本上已经完成了，开篇提到的任务，已经可以在 VSCode 上完成日常的代码编辑的调试工作了。只需要再安装一个 [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) 扩展即可。  

最后，在 `.vscode/settings.json` 中，添加以下一行就可以让 CMake Tools 自动为 VSCode 提供代码提示公共必要的信息。

``` json
"C_Cpp.default.configurationProvider": "vector-of-bool.cmake-tools"
```

# 参考  
[Visual Studio Code and CMake for Qt](https://retifrav.github.io/blog/2019/05/11/vscode-cmake-qt/)  
[用 cmake 构建 Qt 工程 (对比 qmake 进行学习)](https://blog.csdn.net/Bruce_0712/article/details/53574170)  
[qt 的 moc,uic,rcc 命令的使用](https://www.cnblogs.com/xiangism/p/4621108.html)  
[linux 给运行程序指定动态库路径](https://blog.csdn.net/hktkfly6/article/details/61922685)


[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

