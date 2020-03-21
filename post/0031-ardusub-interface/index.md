ArduSub 的事情又要捡起来了。这篇博客就写一些 ArduSub 留给我们来与其交互的接口。  
<!--more-->

# 远程登陆  
这是最基本的交互方式，只要树莓派可以正常运行且连接到了同一个网络，我们就可以远程登陆到树莓派进行一些操作。ArduSub 中树莓派的账户密码如下：
``` json
{
  "user": "pi",
  "passward": "companion"
}
```

# 串口连接树莓派  
当树莓派甚至都无法连接到网络时，我们可以通过串口连接到树莓派，从而进行一些操作。关于树莓派你可能需要了解[这些](/tags/raspberry-pi/)。  

# 网页界面  
ArduSub 提供了一个 Web 界面。这个界面的地址是 ip:2770，默认的地址为 [192.168.2.2:2770](http://192.168.2.2:2770/)。  
这个界面一共有四个部分，分别是 网络(Network)、系统(System)、摄像头(Camera)、路由(Routing)。  

网络页面有如下功能：设置 WiFi、检查 WiFi 和网络状态、设置以太网 IP 地址、测试网络上传下载速度、以太网设置(IP 获取方式：手动、DHCP 服务器、DHCP 客户端，默认的 IP 地址是 192.168.2.2)。

{{% figure class="center" src="/img/0031-ardusub-interface/network.png" alt="Network" title="Network" %}}

系统页面有如下功能：显示 CPU 和内存状态、显示活动的服务、显示视频音频和串口设备的状态、显示 Companion Software 状态和更新 Companion Software、显示当前飞控中的固件版本、更新恢复飞控固件、下载 Web Interface 日志。

{{% figure class="center" src="/img/0031-ardusub-interface/system.png" alt="System" title="System" %}}

摄像头页面有如下下功能：设置摄像头的详细的信息、传输视频的设置。  

{{% figure class="center" src="/img/0031-ardusub-interface/camera.png" alt="Camera" title="Camera" %}}

路由页面有如下功能：设置终端节点和路由。  

{{% figure class="center" src="/img/0031-ardusub-interface/routing.png" alt="Routeing" title="Routeing" %}}

如果进行如下的节点和路由设置，则意味着 192.168.1.2:14500 与 飞控的串口连接到了一起。这样就可以通过网络通信与飞控进行信息交互。

{{% figure class="center" src="/img/0031-ardusub-interface/endpoints-and-routes.png" alt="Endpoints and Routes" title="Endpoints and Routes" %}}

# 其他网页界面

除了上面提到的可以直接在网页中访问的界面，还有几个其他的隐藏界面，需要使用链接才能访问。  

MAVProxy 页面，地址是 ip:2770/mavproxy，默认的地址为 [192.168.2.2:2770/mavproxy](http://192.168.2.2:2770/mavproxy)。在该页面上可以调整 MAVProxy 的设置和重启 MAVProxy 服务。

{{% figure class="center" src="/img/0031-ardusub-interface/mavproxy.png" alt="MAVProxy" title="MAVProxy" %}}

Git 页面，地址是 ip:2770/git，默认的地址为[192.168.2.2:2770/git](http://192.168.2.2:2770/git)。在该页面可以显示当前树莓派上运行的 Companion Software 的仓库的版本号，可以添加远端仓库，并可以进行本地仓库分支的切换。

{{% figure class="center" src="/img/0031-ardusub-interface/git.png" alt="Git" title="Git" %}}

Waterlinked 页面，地址是 ip:2770/waterlinked，默认的地址为[192.168.2.2:2770/waterlinked](http://192.168.2.2:2770/waterlinked)。在该页面可以进行 Waterlinked 水下 GPS 驱动的设置。

{{% figure class="center" src="/img/0031-ardusub-interface/waterlinked.png" alt="Waterlinked" title="Waterlinked" %}}

# 浏览器中的终端   

可以直接在浏览器中使用远程连接的方式连接到树莓派的终端。  
终端地址为 ip:8088，默认地址为 [192.168.2.2:8088](http://192.168.2.2:8088/)。

{{% figure class="center" src="/img/0031-ardusub-interface/terminal.png" alt="Terminal" title="Terminal" %}}

# 文件浏览器  

可以使用文件浏览器查看、移动、重命名、上传文件。文件浏览器的地址为 ip:7777，默认地址为 [192.168.2.2:7777](http://192.168.2.2:7777/)。  

{{% figure class="center" src="/img/0031-ardusub-interface/file-manager.png" alt="File Manager" title="File Manager" %}}

参考：  
[Companion Web Interface](https://www.ardusub.com/operators-manual/companion-web.html)  




