# 理解话题 (Understanding topics)

**目标**：使用 `rqt_graph` 和命令行工具来内省（introspect）ROS 2 话题。

**教程级别**：初学者

**时间**：20 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 1 设置
      * 2 rqt\_graph
      * 3 ros2 topic list
      * 4 ros2 topic echo
      * 5 ros2 topic info
      * 6 ros2 interface show
      * 7 ros2 topic pub
      * 8 ros2 topic hz
      * 9 ros2 topic bw
      * 10 ros2 topic find
      * 11 清理
  * 总结
  * 下一步

## 背景

ROS 2 将复杂的系统分解为许多模块化的节点 (nodes)。话题 (Topics) 是 ROS 图 (ROS graph) 中至关重要的元素，充当节点交换消息的总线。

一个节点可以将数据发布 (publish) 到任意数量的话题，同时也可以订阅 (subscribe) 任意数量的话题。

话题是数据在节点之间传输（因此也是在系统不同部分之间传输）的主要方式之一。

## 先决条件

[上一篇教程](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html)提供了关于节点的有用背景信息，本教程将以此为基础。

一如既往，不要忘记在打开的**每个新终端**中 source ROS 2。

## 任务

### 1 设置

到目前为止，您应该能够熟练启动 turtlesim 了。

打开一个新终端并运行：

```bash
ros2 run turtlesim turtlesim_node
```

打开另一个终端并运行：

```bash
ros2 run turtlesim turtle_teleop_key
```

回顾[上一篇教程](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html)，这些节点的名称默认分别为 `/turtlesim` 和 `/teleop_turtle`。

### 2 rqt\_graph

在本教程中，我们将使用 `rqt_graph` 来可视化节点和话题的变化，以及它们之间的连接。

[turtlesim 教程](https://www.google.com/search?q=https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Using-Turtlesim-Ros2-And-Rqt/Using-Turtlesim-Ros2-And-Rqt.html) 告诉您如何安装 rqt 及其所有插件，包括 `rqt_graph`。

要运行 rqt\_graph，请打开一个新终端并输入命令：

```bash
ros2 run rqt_graph rqt_graph
```

您也可以通过打开 `rqt` 并选择 **Plugins** \> **Introspection** \> **Node Graph** 来打开 rqt\_graph。

您应该能看到上述节点和话题，以及图周围的两个动作（我们暂时忽略这些）。如果您将鼠标悬停在中心的话题上，您将看到如上图所示的颜色高亮显示。

该图描绘了 `/turtlesim` 节点和 `/teleop_turtle` 节点如何通过话题相互通信。`/teleop_turtle` 节点正在将数据（您输入的用于移动海龟的按键）发布到 `/turtle1/cmd_vel` 话题，而 `/turtlesim` 节点订阅了该话题以接收数据。

在检查具有许多节点和以多种不同方式连接的话题的更复杂系统时，`rqt_graph` 的高亮显示功能非常有帮助。

`rqt_graph` 是一个图形化的内省工具。现在我们将通过一些命令行工具来查看话题。

### 3 ros2 topic list

在一个新终端中运行 `ros2 topic list` 命令将返回系统中当前活动的所有话题的列表：

```bash
ros2 topic list
```

```text
/parameter_events
/rosout
/turtle1/cmd_vel
/turtle1/color_sensor
/turtle1/pose
```

`ros2 topic list -t` 将返回相同的话题列表，这次在括号中附加了话题类型：

```bash
ros2 topic list -t
```

```text
/parameter_events [rcl_interfaces/msg/ParameterEvent]
/rosout [rcl_interfaces/msg/Log]
/turtle1/cmd_vel [geometry_msgs/msg/Twist]
/turtle1/color_sensor [turtlesim/msg/Color]
/turtle1/pose [turtlesim/msg/Pose]
```

这些属性，特别是类型，是节点知道它们在话题上传输的是相同信息的方式。

如果您想知道这些话题在 `rqt_graph` 中的位置，您可以取消选中 **Hide:** 下的所有框：

不过现在，请保持选中这些选项以避免混淆。

### 4 ros2 topic echo

要查看发布在话题上的数据，请使用：

```bash
ros2 topic echo <topic_name>
```

既然我们知道 `/teleop_turtle` 通过 `/turtle1/cmd_vel` 话题向 `/turtlesim` 发布数据，让我们使用 `echo` 来查看该话题：

```bash
ros2 topic echo /turtle1/cmd_vel
```

起初，此命令不会返回任何数据。那是因为它正在等待 `/teleop_turtle` 发布内容。

回到运行 `turtle_teleop_key` 的终端，使用箭头键移动海龟。同时观察运行 `echo` 的终端，您将看到为您所做的每一个动作发布的位置数据：

```text
linear:
  x: 2.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
  ---
```

现在回到 `rqt_graph` 并取消选中 **Debug** 框。

`/_ros2cli_26646` 是我们刚刚运行的 `echo` 命令创建的节点（数字可能不同）。现在您可以看到发布者正在通过 `cmd_vel` 话题发布数据，并且有两个订阅者订阅了它。

### 5 ros2 topic info

话题不一定只是一对一的通信；它们可以是一对多、多对一或多对多的。

查看此情况的另一种方法是运行：

```bash
ros2 topic info /turtle1/cmd_vel
```

```text
Type: geometry_msgs/msg/Twist
Publisher count: 1
Subscription count: 2
```

#### 5.1 ros2 topic info --verbose

要获取有关话题的更详细信息，可以使用 `--verbose`（或 `-v`）标志：

```bash
ros2 topic info /turtle1/cmd_vel --verbose
```

这将返回更多详细信息，包括：

  * 发布者和订阅者的节点名称及命名空间
  * 话题类型
  * QoS 配置文件

<!-- end list -->

```text
Type: geometry_msgs/msg/Twist
Publisher count: 1
Node name: teleop_turtle
Node namespace: /
Topic type: geometry_msgs/msg/Twist
Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
Endpoint type: PUBLISHER
GID: 24.ba.3e.e7.c1.51.bb.46.21.41.de.36.1b.14.73.5e
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (7)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite
Subscription count: 2
Node name: _ros2cli_300492
Node namespace: /
Topic type: geometry_msgs/msg/Twist
Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
Endpoint type: SUBSCRIPTION
GID: cc.4d.98.79.29.91.fe.25.8a.0a.c9.03.db.1a.ec.81
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (5)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite
Node name: turtlesim
Node namespace: /
Topic type: geometry_msgs/msg/Twist
Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
Endpoint type: SUBSCRIPTION
GID: 9c.33.59.38.b2.f2.42.47.69.1b.7f.0e.5e.1d.86.f5
QoS profile:
  Reliability: RELIABLE
  History (Depth): KEEP_LAST (7)
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite
```

### 6 ros2 interface show

节点使用消息在话题上发送数据。发布者和订阅者必须发送和接收相同类型的消息才能进行通信。

我们在前面运行 `ros2 topic list -t` 后看到的话题类型让我们知道每个话题使用了什么消息类型。回想一下 `cmd_vel` 话题的类型是：

```text
geometry_msgs/msg/Twist
```

这意味着在 `geometry_msgs` 包中有一个名为 `Twist` 的消息 (`msg`)。

现在我们可以对此类型运行 `ros2 interface show <msg_type>` 来了解其详细信息。具体来说，就是消息期望的数据结构。

```bash
ros2 interface show geometry_msgs/msg/Twist
```

这将返回：

```text
# This expresses velocity in free space broken into its linear and angular parts.
    Vector3  linear
            float64 x
            float64 y
            float64 z
    Vector3  angular
            float64 x
            float64 y
            float64 z
```

这告诉您 `/turtlesim` 节点期望一条包含两个向量（`linear` 和 `angular`）的消息，每个向量包含三个元素。如果您回想一下我们使用 `echo` 命令看到的 `/teleop_turtle` 传递给 `/turtlesim` 的数据，它具有相同的结构：

```text
linear:
  x: 2.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
  ---
```

### 7 ros2 topic pub

既然您已经掌握了消息结构，就可以使用以下命令直接从命令行将数据发布到话题：

```bash
ros2 topic pub <topic_name> <msg_type> '<args>'
```

`'<args>'` 参数是您将传递给话题的实际数据，其结构正是您在上一节中发现的。

海龟（以及它所模拟的真实机器人）通常需要连续的命令流才能持续运行。因此，要让海龟移动并保持移动，您可以使用以下命令。需要注意的是，此参数需要以 YAML 语法输入。输入完整的命令如下：

```bash
ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

在没有命令行选项的情况下，`ros2 topic pub` 会以 1 Hz 的频率持续发布命令流。

有时您可能希望仅将数据发布到话题一次（而不是持续发布）。要仅发布一次命令，请添加 `--once` 选项。

```bash
ros2 topic pub --once -w 2 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

  * `--once` 是一个可选参数，意思是“发布一条消息然后退出”。
  * `-w 2` 是一个可选参数，意思是“等待两个匹配的订阅”。这是必需的，因为我们有 turtlesim 和 topic echo 两个订阅者。

您将在终端中看到以下输出：

```text
Waiting for at least 2 matching subscription(s)...
publisher: beginning loop
publishing #1: geometry_msgs.msg.Twist(linear=geometry_msgs.msg.Vector3(x=2.0, y=0.0, z=0.0), angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=1.8))
```

并且您将看到您的海龟像这样移动：

您可以刷新 `rqt_graph` 以图形方式查看正在发生的事情。您将看到 `ros2 topic pub ...` 节点 (`/_ros2cli_30358`) 正在通过 `/turtle1/cmd_vel` 话题发布，该话题现在正被 `ros2 topic echo ...` 节点 (`/_ros2cli_26646`) 和 `/turtlesim` 节点接收。

最后，您可以在 `pose` 话题上运行 `echo` 并重新检查 `rqt_graph`：

```bash
ros2 topic echo /turtle1/pose
```

您可以看到 `/turtlesim` 节点也在向 `pose` 话题发布，新的 `echo` 节点已订阅该话题。

当发布带有时间戳的消息时，`pub` 有两种方法可以用当前时间自动填充它们。对于带有 `std_msgs/msg/Header` 的消息，可以将 header 字段设置为 `auto` 以填充 `stamp` 字段。

```bash
ros2 topic pub /pose geometry_msgs/msg/PoseStamped '{header: "auto", pose: {position: {x: 1.0, y: 2.0, z: 3.0}}}'
```

如果消息不使用完整的 header，而只是具有类型为 `builtin_interfaces/msg/Time` 的字段，则可以将其设置为值 `now`。

```bash
ros2 topic pub /reference sensor_msgs/msg/TimeReference '{header: "auto", time_ref: "now", source: "dumy"}'
```

### 8 ros2 topic hz

您还可以使用以下命令查看数据发布的速率：

```bash
ros2 topic hz /turtle1/pose
```

```text
average rate: 59.354
  min: 0.005s max: 0.027s std dev: 0.00284s window: 58
```

它将返回 `/turtlesim` 节点向 `pose` 话题发布数据的速率。

回想一下，您使用 `ros2 topic pub --rate 1` 将 `turtle1/cmd_vel` 的发布速率设置为稳定的 1 Hz。如果您使用 `turtle1/cmd_vel` 而不是 `turtle1/pose` 运行上述命令，您将看到反映该速率的平均值。

> **注意**
> 该速率反映了由 `ros2 topic hz` 命令创建的订阅的接收速率，这可能会受到平台资源和 QoS 配置的影响，并且可能与发布者的速率不完全匹配。

### 9 ros2 topic bw

可以使用以下命令查看话题使用的带宽：

```bash
ros2 topic bw /turtle1/pose
```

```text
Subscribed to [/turtle1/pose]
1.51 KB/s from 62 messages
    Message size mean: 0.02 KB min: 0.02 KB max: 0.02 KB
```

它返回带宽利用率和发布到 `/turtle1/pose` 话题的消息数量。

> **注意**
> 该带宽反映了由 `ros2 topic bw` 命令创建的订阅的接收速率，这可能会受到平台资源和 QoS 配置的影响，并且可能与发布者的带宽不完全匹配。

### 10 ros2 topic find

要列出给定类型的可用话题列表，请使用：

```bash
ros2 topic find <topic_type>
```

回想一下 `cmd_vel` 话题的类型是：

```text
geometry_msgs/msg/Twist
```

使用 `find` 命令输出给定消息类型可用的在话题：

```bash
ros2 topic find geometry_msgs/msg/Twist
```

```text
/turtle1/cmd_vel
```

### 11 清理

此时，您将运行许多节点。不要忘记在每个终端中输入 `Ctrl+C` 来停止它们。

## 总结

节点通过话题发布信息，这允许任意数量的其他节点订阅并访问该信息。在本教程中，您使用 `rqt_graph` 和命令行工具检查了多个节点之间通过话题进行的连接。您现在应该对数据如何在 ROS 2 系统中流动有了一个很好的了解。

## 下一步

接下来，您将通过教程[理解服务 (Understanding services)](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html) 了解 ROS 图中的另一种通信类型。