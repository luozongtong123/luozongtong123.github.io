MAVLink 是一个很轻量的应用层通信协议，最初是应用与无人机领域，目前已经在小型机器人无人车无人水下航行器等领域有所应用。本文主要是介绍如何生成自定义的 MAVLink 消息。
<!--more-->
# 环境的准备  

MAVLink tools 需要如下依赖：  

- [Python](https://www.python.org/) 2.7+ 或 Python 3.3+ ：工具链是用 Python 写的
- Python [future](http://python-future.org/) module：提供 Python 2/3 的兼容层
- (可选) Python [TkInter](https://wiki.python.org/moin/TkInter) module：GUI 工具
- (可选) 添加仓库到环境变量：可直接从命令行中调用；也可以 cd 到仓库目录

以 ArchLinux 为例简单说明下依赖的安装：

``` bash
sudo pacman -S python-pip
pip install --user future
sudo pacman -S python-tk
```

然后克隆 mavlink 仓库：
``` bash
cd ~
git clone https://github.com/zt-luo/mavlink.git
git submodule update --init --recursive
```

仓库的结构如下：
```
.
├── cmake
├── ...
├── doc
├── examples
│   └── linux
├── include
│   └── mavlink
├── mavgenerate.py
├── message_definitions
│   └── v1.0
│       ├── ardupilotmega.xml
│       ├── ASLUAV.xml
│       ├── autoquad.xml
│       ├── common.xml
│       ├── icarous.xml
│       ├── matrixpilot.xml
│       ├── minimal.xml
│       ├── paparazzi.xml
│       ├── python_array_test.xml
│       ├── slugs.xml
│       ├── standard.xml
│       ├── test.xml
│       ├── ualberta.xml
│       └── uAvionix.xml
├── pc.in
├── pymavlink
│   ├── generator
│   ├── ...
│   └── tools
│       ├── AccelSearch.py
│       ├── ...
│       ├── mavgen.py
│       └── sertotcp.py
├── README.md
└── scripts
    ├── format_xml.sh
    ├── test.sh
    ├── travis_update_generated_repos.sh
    └── update_c_library.sh
```

# 消息定义文件  

MAVLink协议中的消息是由XML文件定义的。仓库中的 message_definitions 存放着官方的消息定义文件。每一个XML文件定义了一组受特定系统支持的消息，这每一组消息被称为一种“方言”。受大多数地面站软件和自动驾驶仪支持的消息集合在“common.xml”中定义。Ardupilot的“方言”定义在“ardupilotmega.xml”文件中。  

MAVLink XML 语法规则如下：
``` xml
<?xml version="1.0"?>
<mavlink>
    <include>common.xml</include>
    <include>other_dialect.xml</include>
    <!-- NOTE: If the included file already contains a version tag, 
    remove the version tag here, else uncomment to enable. -->
    <version>6</version>
    <dialect>8</dialect>
    <enums>
        <!-- Enums are defined here (optional) -->
      <enum name="NAME_OF_ENUM">
        <entry value="1" name="NAME_OF_ENUM_ENTRY1"/>
        <entry value="2" name="NAME_OF_ENUM_ENTRY2"/>
        <entry value="3" name="NAME_OF_ENUM_ENTRY3"/>
      </enum>
    </enums>
    <messages>
        <!-- Messages are defined here (optional) -->
      <message id="11065" name="DEPTH_HOLD">
        <description>Hold desire depth.</description>
        <field type="uint8_t" name="cmd">1 for enable, 0 for disable.</field>
        <field type="float" name="depth" units="m">desire depth.</field>
      </message>
    </messages>
</mavlink>
```

include：包含当前方言需要导入的基础方言，一般会包含 common.xml，现已允许多层嵌套  
version：当前协议的版本，如果 include 的文件中包含版本信息则本文件应该去掉版本信息  
dialect：标志特定方言的 ID
enums：当前方言定义的枚举类型  
messages：当前方言定义的消息  

# 添加自定义消息  

## 补充现有通信协议  

以 Ardupilot 的消息定义文件为例，如果想要添加自定义消息只需要在 “ardupilotmega.xml” 文件中添加消息即可。需要注意的是新添加的消息的 ID 和 name 不能和现有的消息重复，同时消息 ID 大于 255 的消息只支持 MAVLink 2。

一个添加自定义实例如下：
``` xml
<!-- ARMs ArduSub messages -->
<message id="11065" name="DEPTH_HOLD">
  <description>Hold desire depth.</description>
  <field type="uint8_t" name="cmd">1 for enable, 0 for disable.</field>
  <field type="float" name="depth" units="m">desire depth.</field>
</message>

<message id="11066" name="ATTITUDE_HOLD">
  <description>Hold desire attitude.</description>
  <field type="uint8_t" name="cmd">1 for enable, 0 for disable.</field>
  <field type="float" name="yaw" units="degree">desire yaw.</field>
  <field type="float" name="pitch" units="degree">desire pitch.</field>
  <field type="float" name="roll" units="roll">desire roll.</field>
</message>
```

## 编写你自己的“方言”  

如果你想创建一个新的方言用于你自己的系统，那么只需要参考现 xml 文件的语法规则编写一个新的 xml 文件即可。  
需要注意的是新建的 xml 文件需要放到 `message_definitions/v1.0` 文件夹中。

# 编译到具体语言  

## mavgenerate
可以使用 mavgenerate.py 工具生成指定的语言。

{{% figure class="center" src="/img/0040-gen-new-mavlink-msg/mavgenerate.png" alt="mavgenerate" title="mavgenerate" %}}

## mavgen  

mavgenerate 只是一个 GUI 的包裹工具，其后端是 mavgen。mavgen 是一个命令行的工具，位于 `pymavlink/tools/mavgen.py`。其使用方法可以参考[这里](https://mavlink.io/en/getting_started/generate_libraries.html#mavgen)。

``` bash
# 显示帮助信息
./pymavlink/tools/mavgen.py -h
```

## update_c_library  

由于我做自定义的消息的时候的主要目的是用于 Ardupilot 的开发，所以为了方便我直接使用 `scripts/update_c_library.sh` 脚本生成 c 语言的版本并且自动同步到 github 的仓库。update_c_library 同样使用 mavgen 作为后端。

``` bash
./scripts/update_c_library.sh 2 # MAVLink 2 
./scripts/update_c_library.sh 1 # MAVLink 1
```

# 参考  
[MAVLink Developer Guide](https://mavlink.io/en/)  
[MAVLink XML File Schema / Format](https://mavlink.io/en/guide/xml_schema.html#enum)  
[Generating MAVLink Libraries](https://mavlink.io/en/getting_started/generate_libraries.html)


