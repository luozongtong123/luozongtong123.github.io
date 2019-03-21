由于目前需要实现自定义的功能，这要求可以对 Ardupilot 中的通信进行干预，包括接收处理消息和发送消息。为了达成这个目的，阅读了通信相关的代码，看一下 Ardupilot 是怎样处理 Mavlink 消息的。  
<!--more-->
{{% admonition note %}}
`GCS_Sub->GCS` 是指 `GCS_Sub` 的基类是 `GCS`。
{{% /admonition %}}

Ardupilot 对 Mavlink 消息的处理，主要实现在 `GCS_Sub->GCS` 和 `GCS_MAVLINK_Sub->GCS_MAVLINK` 这两个类中。  

`GCS_Sub` 中包含多个 MAVLink 通道。  

# GCS初始化  

初始化的执行在 `void Sub::init_ardupilot()` 中，相关代码如下：

``` cpp
GCS_Sub &Sub::gcs()

// 初始化第一个 MAVLink 通道
// setup first port early to allow BoardConfig to report errors
gcs().chan(0).setup_uart(serial_manager, AP_SerialManager::SerialProtocol_MAVLink, 0);

// 初始化所有的 MAVLink 通道
gcs().setup_uarts(serial_manager);

void GCS::setup_uarts(AP_SerialManager &serial_manager)
{
    for (uint8_t i = 1; i < MAVLINK_COMM_NUM_BUFFERS; i++) {
        chan(i).setup_uart(serial_manager, AP_SerialManager::SerialProtocol_MAVLink, i);
    }
}

void GCS_MAVLINK::setup_uart(const AP_SerialManager& serial_manager, AP_SerialManager::SerialProtocol protocol, uint8_t instance)
{
  AP_HAL::UARTDriver *uart;
  uart = serial_manager.find_serial(protocol, instance);

  // 在 begin 中创建了串口线程
  uart->begin(115200);
}
```

# 接收发送任务  

在主线程 `scheduler_tasks[]` 中有如下两个任务，这两个任务是用来处理 GCS 消息的接收和处理需要周期性发送的消息。两个任务的执行频率都是 400 Hz。  

``` cpp
SCHED_TASK_CLASS(GCS, (GCS*)&sub._gcs, update_receive,     400, 180),
SCHED_TASK_CLASS(GCS, (GCS*)&sub._gcs, update_send,        400, 550),
```

## 接收消息任务  

先来看 `update_receive()` 任务，基类 `GCS` 实现了该函数，在该函数中，调用每个 `GCS_MAVLINK` 通道的 `update_receive()`。  

`GCS::update_receive() => GCS_MAVLINK::update_receive()`  

`GCS_MAVLINK::update_receive()` 接收到一个完整的消息包后调用 `packetReceived()`，检查消息，如果消息需要我们做处理则调用 `handleMessage()`。基类 `GCS_MAVLINK` 中定义的 `handleMessage()` 是纯虚函数，该函数在 `GCS_MAVLINK_Sub` 中进行了重写。在重写的函数中首先处理航行器特定的消息，然后调用 `void GCS_MAVLINK::handle_common_message(mavlink_message_t *msg)` 处理一些通用的不需要特定航行器数据的消息。所以整个消息的接收我们只需要关注 `void GCS_MAVLINK_Sub::handleMessage(mavlink_message_t* msg)`  的实现即可。其实现结构如下：  

``` cpp
// file: ArduSub/GCS_Mavlink.cpp
void GCS_MAVLINK_Sub::handleMessage(mavlink_message_t* msg)
{
  switch (msg->msgid) {

    case MAVLINK_MSG_ID_HEARTBEAT: {
      ...
      break;
    }

    case ...: {
      ...
      break;
    }

    default: {
      // handle messages which don't require vehicle specific data
      handle_common_message(msg);
      break;
    }

  }  // end switch
}    // end handle mavlink
```

如果我们需要添加自己的消息只需要在 `handleMessage()` 的最开始的地方先对消息进行处理，如果是我们自己定义的消息就直接将函数返回，如果不是自定义的函数则再进入 switch 进行消息的处理。  

添加自定义消息的处理函数，详见 Github [commit](https://github.com/zt-luo/ardupilot/commit/471326c92979c284a7a9a1b5a8ebf68636394e8f)。

## 发送消息任务  

接下来看 `update_send` 任务，同样是基类 `GCS` 实现了该函数，在该函数中，调用每个 `GCS_MAVLINK` 通道的 `update_send()`。  

`GCS::update_send() => GCS_MAVLINK::update_send()`  

`GCS_MAVLINK::update_send()` 中调用 `do_try_send_message()`，之后调用 `try_send_message()`。实际的消息的发送是在 `try_send_message()` 中处理的，`GCS_MAVLINK_Sub` 重新实现了该函数：

``` cpp
bool GCS_MAVLINK_Sub::try_send_message(enum ap_message id)
{
    if (sub.scheduler.time_available_usec() < 250 && sub.motors.armed()) {
        gcs().set_out_of_time(true);
        return false;
    }

    switch (id) {

    case MSG_NAMED_FLOAT:
        send_info();
        break;
    ...

    default:
        return GCS_MAVLINK::try_send_message(id);
    }

    return true;
}
```

`try_send_message()` 根据不同的消息 `msg_id` 发送不同的消息。`*_try_send_message()` 需要提供消息的 ID 来发送指定的消息。有一个 `deferred_message` 用来指定不同的待发送的消息。`deferred_message` 是一个 `deferred_message_t` 如下的结构体：  

``` cpp
// outbound ("deferred message") queue.

// "special" messages such as heartbeat, next_param etc are stored
// separately to stream-rated messages like AHRS2 etc.  If these
// were to be stored in buckets then they would be slowed down
// based on stream_slowdown, which we have not traditionally done.
struct deferred_message_t {
    const ap_message id;
    uint16_t interval_ms;
    uint16_t last_sent_ms;
} deferred_message[2] = {
    { MSG_HEARTBEAT, },
    { MSG_NEXT_PARAM, },
};

// returns index of a message in deferred_message[] which should
// be sent (or -1 if none to send at the moment)
int8_t deferred_message_to_send_index();
```

通过 `deferred_message_to_send_index()` 确定需要发送的消息。`enum ap_message` 中包含了所有的消息类型，其定义在 `libraries/GCS_MAVLink/GCS.h` 中，添加自定义的消息类型需要更改 `enum ap_message`。  

消息的发送是分组的，分组的信息由 `GCS_MAVLINK::all_stream_entries[]` ，其在 ArduSub 中的定义如下：

``` cpp
const struct GCS_MAVLINK::stream_entries GCS_MAVLINK::all_stream_entries[] = {
    MAV_STREAM_ENTRY(STREAM_RAW_SENSORS),
    MAV_STREAM_ENTRY(STREAM_EXTENDED_STATUS),
    MAV_STREAM_ENTRY(STREAM_POSITION),
    MAV_STREAM_ENTRY(STREAM_RC_CHANNELS),
    MAV_STREAM_ENTRY(STREAM_EXTRA1),
    MAV_STREAM_ENTRY(STREAM_EXTRA2),
    MAV_STREAM_ENTRY(STREAM_EXTRA3),
    MAV_STREAM_ENTRY(STREAM_PARAMS),
    MAV_STREAM_TERMINATOR // must have this at end of stream_entries
};
```

共有 8 个分组，每个分组分别包含不同的消息。一个分组的示意如下：

``` cpp
static const ap_message STREAM_EXTRA1_msgs[] = {
    MSG_ATTITUDE,
    MSG_SIMSTATE,
    MSG_AHRS2,
    MSG_AHRS3,
    MSG_PID_TUNING
};
```

通过跟踪 `all_stream_entries` 的引用发现，Ardupilot 是通过不同的分组来确定消息发送的频率。在第一次调用 `void GCS_MAVLINK::update_send()` 时会进行初始化消息的发送频率。在确定消息的发送频率时有如下的调用关系：

``` cpp
// file: libaries/GCS_MAVLink/GCS_Common.cpp
void GCS_MAVLINK::initialise_message_intervals_from_streamrates();
// ->
void GCS_MAVLINK::initialise_message_intervals_for_stream(GCS_MAVLINK::streams id);
// ->
uint16_t GCS_MAVLINK::get_interval_for_stream(GCS_MAVLINK::streams id);
```

最后发现实际的发送频率是由下面一组可由地面站修改并可以保存在闪存中的参数决定的。

``` cpp
// saveable rate of each stream
AP_Int16 streamRates[NUM_STREAMS];
```

一个参数的示例：

``` cpp
const AP_Param::GroupInfo GCS_MAVLINK::var_info[] = {
    // @Param: RAW_SENS
    // @DisplayName: Raw sensor stream rate
    // @Description: Stream rate of RAW_IMU, SCALED_IMU2, SCALED_PRESSURE, and SENSOR_OFFSETS to ground station
    // @Units: Hz
    // @Range: 0 10
    // @Increment: 1
    // @User: Advanced
    AP_GROUPINFO("RAW_SENS", 0, GCS_MAVLINK, streamRates[STREAM_RAW_SENSORS],  0),
    AP_GROUPEND
};
```

在 Mission Planner 中的参数截图如下：

{{% figure class="center" src="/img/ardupilot-handle-msg/GCS_MAVLINK_Param-Mission-Planner.png" alt="GCS_MAVLINK_Param" title="GCS_MAVLINK_Param" %}}

**如我我们需要修改某条现有的消息的发送频率只需要调整这条消息的分组或者调整消息分组的发送频率。**  

**如果我们要添加一条自定义的消息到周期性发送的任务中则需要做一下几点：**

1. 在 `enum ap_message` 添加自定义的消息类型；  
2. 在消息分组中添加自定义的消息；  
3. 在 `try_send_message()` 中发送自定义的消息；  

如果只是需要发送一条消息的话可以直接调用 `mavlink_ms_*_send()` 这一系列函数。

# ~~添加自定义消息~~  

添加自定义消息需要修改 `modules/mavlink/message_definitions/v1.0/common.xml` 文件，而这个文件位于 `mavlink` 子模块，这就意味着如果要修改这个子模块中的文件就要再另外维护一个子模块仓库，所以暂时不考虑做这一操作。  

为了达成相似的功能，使用 [NAMED_VALUE_FLOAT (#251)](https://mavlink.io/en/messages/common.html#NAMED_VALUE_FLOAT)  和 [NAMED_VALUE_INT (#252)](https://mavlink.io/en/messages/common.html#NAMED_VALUE_INT) 两个消息可以完成单个数据的传递的功能。[RC_CHANNELS_OVERRIDE (#70)](https://mavlink.io/en/messages/common.html#RC_CHANNELS_OVERRIDE) 可以一次完成多个数据的传递。  

{{% admonition note %}}

通过阅读这部分代码，发现可以通过类的继承添加自己的代码到现有的代码中，这样组织代码会比较合理。打算添加 `GCS_Sub_lab->GCS_Sub` 和  `GCS_MAVLINK_Sub_lab->GCS_MAVLINK_Sub` 这两个子类来添加通信相关的代码。相关实现见这个 [commit](https://github.com/zt-luo/ardupilot/commit/e00726b691ce8a1ab2217c3961391d39f7dcb7ca) 。

{{% /admonition %}}


