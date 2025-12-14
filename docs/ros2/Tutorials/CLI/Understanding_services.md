# 理解服务 (Understanding services)

**目标**：使用命令行工具了解 ROS 2 中的服务。

**教程级别**：初学者

**时间**：10 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 1 设置
      * 2 ros2 service list
      * 3 ros2 service type
      * 4 ros2 service find
      * 5 ros2 interface show
      * 6 ros2 service call
  * 总结
  * 下一步
  * 相关内容

## 背景

服务 (Services) 是 ROS 图中节点通信的另一种方法。服务基于调用-响应 (call-and-response) 模型，而话题基于发布-订阅 (publisher-subscriber) 模型。话题允许节点订阅数据流并获取持续更新，而服务仅在客户端专门调用时才提供数据。

## 先决条件

本教程中提到的一些概念（如[节点](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html)和[话题](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)）已在本系列的先前教程中介绍过。

您将需要 `turtlesim` 包。

一如既往，不要忘记在打开的**每个新终端**中 source ROS 2。

## 任务

### 1 设置

启动两个 turtlesim 节点：`/turtlesim` 和 `/teleop_turtle`。

打开一个新终端并运行：

```bash
ros2 run turtlesim turtlesim_node
```

打开另一个终端并运行：

```bash
ros2 run turtlesim turtle_teleop_key
```

### 2 ros2 service list

在新终端中运行 `ros2 service list` 命令将返回系统中当前活动的所有服务的列表：

```bash
ros2 service list
```

```text
/clear
/kill
/reset
/spawn
/teleop_turtle/describe_parameters
/teleop_turtle/get_parameter_types
/teleop_turtle/get_parameters
/teleop_turtle/list_parameters
/teleop_turtle/set_parameters
/teleop_turtle/set_parameters_atomically
/turtle1/set_pen
/turtle1/teleport_absolute
/turtle1/teleport_relative
/turtlesim/describe_parameters
/turtlesim/get_parameter_types
/turtlesim/get_parameters
/turtlesim/list_parameters
/turtlesim/set_parameters
/turtlesim/set_parameters_atomically
```

您会看到两个节点都有六个名称中带有 `parameters` 的相同服务。ROS 2 中的几乎每个节点都具有这些基础结构服务，参数就是基于这些服务构建的。下一篇教程将详细介绍参数。在本教程中，将忽略参数服务。

现在，让我们专注于 turtlesim 特有的服务：`/clear`、`/kill`、`/reset`、`/spawn`、`/turtle1/set_pen`、`/turtle1/teleport_absolute` 和 `/turtle1/teleport_relative`。您可能还记得在[使用 turtlesim、ros2 和 rqt](https://www.google.com/search?q=https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Using-Turtlesim-Ros2-And-Rqt/Using-Turtlesim-Ros2-And-Rqt.html) 教程中使用 rqt 与其中一些服务进行过交互。

### 3 ros2 service type

服务具有描述服务请求和响应数据结构的类型。服务类型的定义与话题类型类似，不同之处在于服务类型有两部分：一条消息用于请求，另一条用于响应。

要查找服务的类型，请使用命令：

```bash
ros2 service type <service_name>
```

让我们看看 turtlesim 的 `/clear` 服务。在新终端中，输入命令：

```bash
ros2 service type /clear
```

```text
std_srvs/srv/Empty
```

`Empty` 类型意味着服务调用在发出请求时不发送数据，在接收响应时不接收数据。

#### 3.1 ros2 service list -t

要同时查看所有活动服务的类型，可以将 `--show-types` 选项（缩写为 `-t`）附加到 `list` 命令：

```bash
ros2 service list -t
```

```text
/clear [std_srvs/srv/Empty]
/kill [turtlesim/srv/Kill]
/reset [std_srvs/srv/Empty]
/spawn [turtlesim/srv/Spawn]
...
/turtle1/set_pen [turtlesim/srv/SetPen]
/turtle1/teleport_absolute [turtlesim/srv/TeleportAbsolute]
/turtle1/teleport_relative [turtlesim/srv/TeleportRelative]
...
```

### 4 ros2 service find

如果要查找特定类型的所有服务，可以使用以下命令：

```bash
ros2 service find <type_name>
```

例如，您可以像这样找到所有 `Empty` 类型服务：

```bash
ros2 service find std_srvs/srv/Empty
```

```text
/clear
/reset
```

### 5 ros2 interface show

您可以从命令行调用服务，但首先需要知道输入参数的结构。

```bash
ros2 interface show <type_name>
```

在 `/clear` 服务的类型 `Empty` 上尝试一下：

```bash
ros2 interface show std_srvs/srv/Empty
```

```text
---
```

`---` 将请求结构（上方）与响应结构（下方）分隔开。但是，正如您之前了解到的，`Empty` 类型不发送或接收任何数据。所以，很自然地，它的结构是空白的。

让我们内省一个发送和接收数据的服务类型，比如 `/spawn`。从 `ros2 service list -t` 的结果中，我们知道 `/spawn` 的类型是 `turtlesim/srv/Spawn`。

要查看 `/spawn` 服务的请求和响应参数，请运行命令：

```bash
ros2 interface show turtlesim/srv/Spawn
```

```text
float32 x
float32 y
float32 theta
string name # Optional.  A unique name will be created and returned if this is empty
---
string name
```

`---` 线以上的信息告诉我们调用 `/spawn` 所需的参数。`x`、`y` 和 `theta` 决定了生成的海龟的 2D 位姿，而 `name` 显然是可选的。

线以下的信息不是您在这种情况下需要知道的，但它可以帮助您了解从调用中获得的响应的数据类型。

### 6 ros2 service call

既然您知道了什么是服务类型，如何查找服务的类型，以及如何查找该类型参数的结构，您就可以使用以下命令调用服务：

```bash
ros2 service call <service_name> <service_type> <arguments>
```

`<arguments>` 部分是可选的。例如，您知道 `Empty` 类型服务没有任何参数：

```bash
ros2 service call /clear std_srvs/srv/Empty
```

此命令将清除 turtlesim 窗口中您的海龟绘制的任何线条。

现在，让我们通过调用 `/spawn` 并设置参数来生成一只新海龟。命令行服务调用中的输入 `<arguments>` 需要采用 YAML 语法。

输入命令：

```bash
ros2 service call /spawn turtlesim/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"
```

```text
requester: making request: turtlesim.srv.Spawn_Request(x=2.0, y=2.0, theta=0.2, name='')

response:
turtlesim.srv.Spawn_Response(name='turtle2')
```

您将获得这种关于正在发生的事情的方法式视图，然后是服务响应。

您的 turtlesim 窗口将立即更新为新生成的海龟：

## 总结

节点可以使用 ROS 2 中的服务进行通信。与话题（一种单向通信模式，节点发布信息，由一个或多个订阅者使用）不同，服务是一种请求/响应模式，客户端向提供服务的节点发出请求，服务处理请求并生成响应。

您通常不希望将服务用于连续调用；话题甚至动作会更适合。

在本教程中，您使用了命令行工具来识别、内省和调用服务。

## 下一步

在下一篇教程[理解参数](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Parameters/Understanding-ROS2-Parameters.html)中，您将学习配置节点设置。

## 相关内容

查看[这篇教程](https://www.google.com/search?q=https://docs.ros.org/en/humble/Tutorials/Intermediate/Ros2bag/Recording-And-Playing-Back-Data.html)；这是一个使用 Robotis 机械臂应用 ROS 服务的绝佳现实案例。