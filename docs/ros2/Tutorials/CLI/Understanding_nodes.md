# 理解节点 (Understanding nodes)

**目标**：了解 ROS 2 中节点的功能，以及与它们交互的工具。

**教程级别**：初学者

**时间**：10 分钟

**目录**

  * 背景
      * 1 ROS 2 图 (The ROS 2 graph)
      * 2 ROS 2 中的节点
  * 先决条件
  * 任务
      * 1 ros2 run
      * 2 ros2 node list
      * 3 ros2 node info
  * 总结
  * 下一步
  * 相关内容

## 背景

### 1 ROS 2 图 (The ROS 2 graph)

在接下来的几个教程中，您将学习一系列核心的 ROS 2 概念，这些概念构成了所谓的“ROS (2) 图”。

ROS 图是一个由同时处理数据的 ROS 2 元素组成的网络。如果您将所有可执行文件及其之间的连接全部映射出来并可视化，它就涵盖了所有这些内容。

### 2 ROS 2 中的节点

ROS 中的每个节点应该负责一个单一的、模块化的目的，例如控制车轮电机或发布激光测距仪的传感器数据。每个节点都可以通过话题 (topics)、服务 (services)、动作 (actions) 或参数 (parameters) 向其他节点发送和接收数据。

一个完整的机器人系统由许多协同工作的节点组成。在 ROS 2 中，单个可执行文件（C++ 程序、Python 程序等）可以包含一个或多个节点。

## 先决条件

[上一篇教程](https://www.google.com/search?q=https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Using-Turtlesim-Ros2-And-Rqt/Using-Turtlesim-Ros2-And-Rqt.html)向您展示了如何安装这里使用的 `turtlesim` 包。

一如既往，不要忘记在您打开的**每一个新终端**中 source ROS 2。

## 任务

### 1 ros2 run

命令 `ros2 run` 用于从包中启动一个可执行文件。

```bash
ros2 run <package_name> <executable_name>
```

要运行 turtlesim，请打开一个新的终端，并输入以下命令：

```bash
ros2 run turtlesim turtlesim_node
```

turtlesim 窗口将会打开，正如您在上一篇教程中看到的那样。

在这里，包名是 `turtlesim`，可执行文件名是 `turtlesim_node`。

然而，我们还不知道节点名称。您可以使用 `ros2 node list` 来查找节点名称。

### 2 ros2 node list

`ros2 node list` 将显示所有正在运行的节点的名称。当您想与节点交互时，或者当您的系统运行着许多节点并需要对其进行跟踪时，这特别有用。

在 turtlesim 仍在另一个终端运行时打开一个新的终端，并输入以下命令。终端将返回节点名称：

```bash
ros2 node list
```

```text
/turtlesim
```

打开另一个新终端，并使用以下命令启动 teleop（遥操作）节点：

```bash
ros2 run turtlesim turtle_teleop_key
```

在这里，我们要再次引用 `turtlesim` 包，但这次我们的目标是名为 `turtle_teleop_key` 的可执行文件。

返回到您运行 `ros2 node list` 的终端并再次运行它。您现在将看到两个活动节点的名称：

```bash
ros2 node list
```

```text
/turtlesim
/teleop_turtle
```

#### 2.1 重映射 (Remapping)

[重映射](https://docs.ros.org/en/humble/How-To-Guides/Node-arguments.html)允许您将默认的节点属性（如节点名称、话题名称、服务名称等）重新分配给自定义值。在上一篇教程中，您在 `turtle_teleop_key` 上使用了重映射来更改 `cmd_vel` 话题并以此控制 `turtle2`。

现在，让我们重新分配 `/turtlesim` 节点的名称。在一个新的终端中，运行以下命令：

```bash
ros2 run turtlesim turtlesim_node --ros-args --remap __node:=my_turtle
```

由于您再次在 turtlesim 上调用了 `ros2 run`，因此将打开另一个 turtlesim 窗口。但是，现在如果您返回到运行 `ros2 node list` 的终端，并再次运行它，您将看到三个节点名称：

```bash
ros2 node list
```

```text
/my_turtle
/turtlesim
/teleop_turtle
```

### 3 ros2 node info

既然您知道了节点的名称，就可以通过以下命令访问有关它们的更多信息：

```bash
ros2 node info <node_name>
```

要检查您最新的节点 `my_turtle`，请运行以下命令：

```bash
ros2 node info /my_turtle
```

```text
/my_turtle
  Subscribers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
  Service Servers:
    /clear: std_srvs/srv/Empty
    /kill: turtlesim/srv/Kill
    /my_turtle/describe_parameters: rcl_interfaces/srv/DescribeParameters
    /my_turtle/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
    /my_turtle/get_parameters: rcl_interfaces/srv/GetParameters
    /my_turtle/list_parameters: rcl_interfaces/srv/ListParameters
    /my_turtle/set_parameters: rcl_interfaces/srv/SetParameters
    /my_turtle/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
    /reset: std_srvs/srv/Empty
    /spawn: turtlesim/srv/Spawn
    /turtle1/set_pen: turtlesim/srv/SetPen
    /turtle1/teleport_absolute: turtlesim/srv/TeleportAbsolute
    /turtle1/teleport_relative: turtlesim/srv/TeleportRelative
  Service Clients:

  Action Servers:
    /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
  Action Clients:
```

`ros2 node info` 返回订阅者、发布者、服务和动作的列表，即与该节点交互的 ROS 图连接。

现在尝试在 `/teleop_turtle` 节点上运行相同的命令，看看它的连接与 `my_turtle` 有何不同。

您将在接下来的教程中了解有关 ROS 图连接概念（包括消息类型）的更多信息。

## 总结

节点是基本的 ROS 2 元素，在机器人系统中用于服务于单一的、模块化的目的。

在本教程中，您通过运行可执行文件 `turtlesim_node` 和 `turtle_teleop_key` 使用了在 `turtlesim` 包中创建的节点。

您学习了如何使用 `ros2 node list` 来发现活动的节点名称，以及使用 `ros2 node info` 来内省单个节点。这些工具对于理解复杂、真实的机器人系统中的数据流至关重要。

## 下一步

既然您已经了解了 ROS 2 中的节点，就可以继续学习*话题教程*了。话题是连接节点的通信类型之一。

## 相关内容

*概念*页面为节点概念补充了更多细节。