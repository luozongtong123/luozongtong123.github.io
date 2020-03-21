在 [SubControl](https://github.com/zt-luo/SubControl) 地面站软件中视频流最初使用的是 [vlc-qt](https://github.com/vlc-qt/vlc-qt) 插件，但由于在调试时发现一个在有视频播放时软件无法正常退出，同时友考虑到 vlc-qt 不够灵活，所以花了点时间探索了下使用 gst-plugins-good 中的 qmlglsink 播放视频。
<!--more-->

# 安装环境

Qt 的安装就不多说了，GStreamer 的话注意要安装好 gst-plugins-good。如果运行软件时有如下提示则说明环境没有配置好。

```
module "org.freedesktop.gstreamer.GLVideoItem" is not installed
```
依次做以下检查：
``` shell
gst-inspect-1.0 qmlglsink
```
不显示 `No such element or plugin 'qmlglsink'` 或 `没有那样的组件或插件‘qmlglsink’` 则表明正常，
在我的机器上显示找不到 qmlglsink ，但是 gst-plugins-good 确认安装没问题。在谷歌上搜索了一番发现是 qmlglsink 进了黑名单。

``` shell
# 检查黑名单文件
gst-inspect-1.0 -b
```
输出如下：
``` shell
Blacklisted files:
  libgstqmlgl.so

文件黑名单:
  libgstqmlgl.so
```

排查过程：
``` shell
# 进入 ~/.cache/gstreamer-1.0/
# 注册文件名字为 registry.[arch].bin ，如 registry.x86_64.bin
cd ~/.cache/gstreamer-1.0/ && ls

# 删除注册文件
rm registry.x86_64.bin

# 运行 gst-inspect-1.0 刷新注册文件
gst-inspect-1.0
```
接下来就可以根据提示进行排查原因了。在我的机器上显示如下警告：

`(gst-plugin-scanner:5461): GStreamer-WARNING **: 09:58:48.318: Failed to load plugin '/usr/lib/gstreamer-1.0/libgstqmlgl.so': libQt5WaylandClient.so.5: cannot open shared object file: No such file or directory`

猜测可能跟 Qt5 的 `Wayland` 协议有关有关。安装 `qt5-wayland` 包后再进行一次上述的排查过程就可以解决这个问题。

# 官方例子

例子代码详见 [GItHub 链接](https://github.com/GStreamer/gst-plugins-good/tree/master/tests/examples/qt/qmlsink)。

例子的核心代码如下：

``` cpp
...
gst_init (&argc, &argv);
...
GstElement *pipeline = gst_pipeline_new (nullptr);
GstElement *src = gst_element_factory_make ("videotestsrc", nullptr);
GstElement *glupload = gst_element_factory_make ("glupload", nullptr);
/* the plugin must be loaded before loading the qml file to register the
 * GstGLVideoItem qml item */
GstElement *sink = gst_element_factory_make ("qmlglsink", nullptr);
...
gst_bin_add_many (GST_BIN (pipeline), src, glupload, sink, nullptr);
gst_element_link_many (src, glupload, sink, nullptr);
...
QQmlApplicationEngine engine;
engine.load(QUrl(QStringLiteral("qrc:/main.qml")));

QQuickItem *videoItem;
QQuickWindow *rootObject;

/* find and set the videoItem on the sink */
rootObject = static_cast<QQuickWindow *> (engine.rootObjects().first());
videoItem = rootObject->findChild<QQuickItem *> ("videoItem");
g_assert (videoItem);
g_object_set(sink, "widget", videoItem, nullptr);

rootObject->scheduleRenderJob (new SetPlaying (pipeline),
    QQuickWindow::BeforeSynchronizingStage);
...

...
void SetPlaying::run()
...
gst_element_set_state(m_pipeline, GST_STATE_PLAYING);
...
```

``` qml
import QtQuick 2.0
import QtQuick.Controls 1.0

import org.freedesktop.gstreamer.GLVideoItem 1.0

Rectangle {
    objectName:"Rectangle"
    Item {
        anchors.fill: parent

        GstGLVideoItem {
            id: video
            objectName: "videoItem"
            anchors.centerIn: parent
            width: parent.width
            height: parent.height
        }
    }
}
```

具体过程如下：  
首先，使用 `gst_init()` 进行初始化；  
接着，使用 `gst_pipeline_new()` 和 `gst_element_factory_make()` 新建 pipeline 和 element；  
接着，使用 `gst_bin_add_many()` 和  `gst_element_link_many()` 将所有元素连接起来；  
接着，加载 qml ，并找到 `videoItem` 节点和 `Quick Window`。
接着，使用 `g_object_set()` 设定对象的属性，将 `videoItem` 作为 `widget` 添加到 sink；  
最后，使用 `scheduleRenderJob()` 在 `Quick Window` 进行渲染前设定 pipeline 为播放状态。  

官方例子是完全使用 Qt Quick 来实现的，在 SubControl 里的界面是使用 Widget 实现的，所以要拿来用的话还要解决 qml 嵌入到 Widget 的问题。

# c++ 与 qml 交互  

所有的 QML 对象类型，包括 QML 引擎内部实现或者实现第三方库，都是 QObject 子类，都允许 QML 引擎使用 Qt 元对象系统动态实例化任何 QML 对象类型。在启动 QML 时，会初始化一个 QQmlEngine 作为 QML 引擎，然后使用 QQmlComponent 对象加载 QML 文档，QML 引擎会提供一个默认的 QQmlContext 对象作为顶层执行的上下文，用来执行 QML 文档中定义的函数和表达式。
`QQmlEngine::rootContext()` 返回当前引擎 QML 的上下文。`QQuickItem* QQuickView::rootObject()` 返回当前 QQuickView 的根节点，也就是 QML 的根节点。

在这里 quickWidget 的实现中的主要区别是寻找到 `videoItem` 对象的方式不同，其主要代码如下：

``` cpp
QQuickItem *videoItem;
QQuickWindow *qQuickWindows;

/* find and set the videoItem on the sink */
videoItem = ui->quickWidget->rootObject();
ideoItem = videoItem->findChild<QQuickItem *>("videoItem");
assert(videoItem);
g_object_set(sink, "widget", videoItem, nullptr);

qQuickWindows = videoItem->window();
assert(qQuickWindows);
qQuickWindows->scheduleRenderJob(new SetPlaying(pipeline),
                                     QQuickWindow::BeforeSynchronizingStage);
```

最后，完整的 Demo 见 GitHub [链接](https://github.com/zt-luo/QuickWidgetPlayer)。

