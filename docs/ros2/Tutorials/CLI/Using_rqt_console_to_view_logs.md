# 使用 `rqt_console` 查看日志

**目标：** 了解 `rqt_console`，一个用于内省日志消息的工具。

**教程等级：** 初学者

**时间：** 5 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 1 设置
      * 2 rqt\_console 上的消息
      * 3 日志记录器级别
  * 总结
  * 下一步

## 背景

`rqt_console` 是一个用于在 ROS 2 中内省（introspect）日志消息的 GUI 工具。通常，日志消息会显示在您的终端中。使用 `rqt_console`，您可以随着时间的推移收集这些消息，以更有条理的方式仔细查看它们，过滤它们，保存它们，甚至重新加载保存的文件以便在不同的时间进行内省。

节点使用日志以各种方式输出有关事件和状态的消息。为了用户的利益，它们的内容通常是信息性的。

## 先决条件

您需要安装 `rqt_console` 和 `turtlesim`。

一如既往，不要忘记在打开的**每个新终端**中 source ROS 2 环境。

## 任务

### 1 设置

使用以下命令在一个新终端中启动 `rqt_console`：

```bash
ros2 run rqt_console rqt_console
```

`rqt_console` 窗口将会打开：

控制台的第一部分是显示来自系统的日志消息的地方。

在中间部分，您可以选择通过排除严重性级别来过滤消息。您还可以使用右侧的加号按钮添加更多排除过滤器。

底部部分用于高亮显示包含您输入的字符串的消息。您也可以向此部分添加更多过滤器。

现在，使用以下命令在一个新终端中启动 `turtlesim`：

```bash
ros2 run turtlesim turtlesim_node
```

### 2 rqt\_console 上的消息

为了产生让 `rqt_console` 显示的日志消息，让我们让乌龟撞到墙上。在一个新终端中，输入下面的 `ros2 topic pub` 命令（在[话题教程](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)中有详细讨论）：

```bash
ros2 topic pub -r 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0,y: 0.0,z: 0.0}}"
```

由于上述命令以稳定的速率发布话题，乌龟会不断地撞向墙壁。在 `rqt_console` 中，您将看到一遍又一遍显示的带有 **Warn**（警告）严重性级别的相同消息，如下所示：

在运行 `ros2 topic pub` 命令的终端中按 `Ctrl+C` 以停止乌龟撞墙。

### 3 日志记录器级别

ROS 2 的日志记录器级别按严重性排序：

  * **Fatal** (致命)
  * **Error** (错误)
  * **Warn** (警告)
  * **Info** (信息)
  * **Debug** (调试)

对于每个级别表示什么并没有确切的标准，但可以安全地假设：

  * **Fatal** 消息表示系统即将终止以试图保护自身免受损害。
  * **Error** 消息表示不会必然损坏系统但阻止其正常运行的重大问题。
  * **Warn** 消息表示意外活动或非理想结果，可能代表更深层次的问题，但不会直接损害功能。
  * **Info** 消息表示事件和状态更新，作为系统按预期运行的视觉验证。
  * **Debug** 消息详细说明了系统执行的整个分步过程。

默认级别是 **Info**。您只会看到默认严重性级别和更严重级别的消息。

通常，只有 **Debug** 消息被隐藏，因为它们是唯一比 **Info** 严重性低的级别。例如，如果您将默认级别设置为 **Warn**，您只会看到严重性为 **Warn**、**Error** 和 **Fatal** 的消息。

#### 3.1 设置默认日志记录器级别

您可以在首次运行 `/turtlesim` 节点时使用重映射来设置默认日志记录器级别。在您的终端中输入以下命令：

```bash
ros2 run turtlesim turtlesim_node --ros-args --log-level WARN
```

现在您将不会看到上次启动 `turtlesim` 时控制台中出现的初始 **Info** 级别消息。这是因为 **Info** 消息的优先级低于新的默认严重性级别 **Warn**。

## 总结

如果您需要仔细检查系统的日志消息，`rqt_console` 会非常有帮助。您可能出于多种原因想要检查日志消息，通常是为了找出哪里出了问题以及导致该问题的一系列事件。

## 下一步

下一篇教程将教您如何使用 [ROS 2 Launch](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Launching-Multiple-Nodes/Launching-Multiple-Nodes.html) 一次启动多个节点。