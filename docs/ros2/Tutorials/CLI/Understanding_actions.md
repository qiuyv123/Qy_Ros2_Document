# 理解动作 (Understanding actions)

**目标：** 内省 (Introspect) ROS 2 中的动作。

**教程等级：** 初学者

**时间：** 15 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 1 设置
      * 2 使用动作
      * 3 ros2 node info
      * 4 ros2 action list
      * 5 ros2 action info
      * 6 ros2 interface show
      * 7 ros2 action send\_goal
  * 总结
  * 下一步
  * 相关内容

## 背景

动作 (Actions) 是 ROS 2 中的通信类型之一，专用于长时间运行的任务。它们由三部分组成：**目标 (goal)**、**反馈 (feedback)** 和 **结果 (result)**。

动作建立在话题 (topics) 和服务 (services) 之上。它们的功能类似于服务，但动作可以被取消。与服务只返回单个响应不同，动作还可以提供持续的反馈。

动作使用客户端-服务端模型，类似于发布者-订阅者模型（在[话题教程](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)中描述）。“动作客户端”节点向“动作服务端”节点发送目标，服务端确认目标并返回反馈流和结果。

## 先决条件

本教程建立在先前教程中涵盖的概念（如[节点](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html)和[话题](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)）之上。

本教程使用 `turtlesim` 包。

一如既往，不要忘记在打开的**每个新终端**中 source ROS 2 环境。

## 任务

### 1 设置

启动两个 turtlesim 节点：`/turtlesim` 和 `/teleop_turtle`。

打开一个新的终端并运行：

```bash
ros2 run turtlesim turtlesim_node
```

打开另一个终端并运行：

```bash
ros2 run turtlesim turtle_teleop_key
```

### 2 使用动作

当您启动 `/teleop_turtle` 节点时，您将在终端中看到以下消息：

```text
Use arrow keys to move the turtle.
Use G|B|V|C|D|E|R|T keys to rotate to absolute orientations. 'F' to cancel a rotation.
```

让我们关注第二行，它对应于一个动作。（第一条指令对应于“cmd\_vel”话题，之前在[话题教程](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)中讨论过。）

注意字母键 `G|B|V|C|D|E|R|T` 在美式 QWERTY 键盘上围绕着 `F` 键形成一个“方框”（如果您使用的不是 QWERTY 键盘，请参阅[此链接](https://en.wikipedia.org/wiki/QWERTY)以进行对照）。围绕 `F` 的每个键的位置对应于 turtlesim 中的那个方向。例如，`E` 将把乌龟的方向旋转到左上角。

请注意运行 `/turtlesim` 节点的终端。每次按下这些键之一时，您都在向属于 `/turtlesim` 节点的动作服务端发送一个目标。该目标是让乌龟旋转以面向特定方向。一旦乌龟完成旋转，应该会显示一条传递目标结果的消息：

```text
[INFO] [turtlesim]: Rotation goal completed successfully
```

`F` 键将取消执行中的目标。

尝试按下 `C` 键，然后在乌龟完成旋转之前按下 `F` 键。在运行 `/turtlesim` 节点的终端中，您将看到消息：

```text
[INFO] [turtlesim]: Rotation goal canceled
```

不仅客户端（您在 teleop 中的输入）可以停止目标，服务端（`/turtlesim` 节点）也可以。当服务端选择停止处理目标时，称之为“中止 (abort)”目标。

尝试按下 `D` 键，然后在第一次旋转完成之前按下 `G` 键。在运行 `/turtlesim` 节点的终端中，您将看到消息：

```text
[WARN] [turtlesim]: Rotation goal received before a previous goal finished. Aborting previous goal
```

此动作服务端选择中止第一个目标，因为它收到了一个新的目标。它也可以选择其他方式，比如拒绝新目标，或者在第一个目标完成后执行第二个目标。不要假设每个动作服务端在收到新目标时都会选择中止当前目标。

### 3 ros2 node info

要查看节点（在本例中为 `/turtlesim`）提供的动作列表，请打开一个新终端并运行命令：

```bash
ros2 node info /turtlesim
```

```text
/turtlesim
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
    /reset: std_srvs/srv/Empty
    /spawn: turtlesim/srv/Spawn
    /turtle1/set_pen: turtlesim/srv/SetPen
    /turtle1/teleport_absolute: turtlesim/srv/TeleportAbsolute
    /turtle1/teleport_relative: turtlesim/srv/TeleportRelative
    /turtlesim/describe_parameters: rcl_interfaces/srv/DescribeParameters
    /turtlesim/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
    /turtlesim/get_parameters: rcl_interfaces/srv/GetParameters
    /turtlesim/list_parameters: rcl_interfaces/srv/ListParameters
    /turtlesim/set_parameters: rcl_interfaces/srv/SetParameters
    /turtlesim/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
  Service Clients:

  Action Servers:
    /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
  Action Clients:
```

该命令返回 `/turtlesim` 的订阅者、发布者、服务、动作服务端和动作客户端的列表。

注意 `/turtlesim` 的 `/turtle1/rotate_absolute` 动作位于 **Action Servers**（动作服务端）下。这意味着 `/turtlesim` 响应 `/turtle1/rotate_absolute` 动作并提供反馈。

`/teleop_turtle` 节点在 **Action Clients**（动作客户端）下有 `/turtle1/rotate_absolute` 名称，这意味着它发送该动作名称的目标。要查看这一点，请运行：

```bash
ros2 node info /teleop_turtle
```

```text
/teleop_turtle
  Subscribers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  Service Servers:
    /teleop_turtle/describe_parameters: rcl_interfaces/srv/DescribeParameters
    /teleop_turtle/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
    /teleop_turtle/get_parameters: rcl_interfaces/srv/GetParameters
    /teleop_turtle/list_parameters: rcl_interfaces/srv/ListParameters
    /teleop_turtle/set_parameters: rcl_interfaces/srv/SetParameters
    /teleop_turtle/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
  Service Clients:

  Action Servers:

  Action Clients:
    /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
```

### 4 ros2 action list

要识别 ROS 图中的所有动作，请运行命令：

```bash
ros2 action list
```

```text
/turtle1/rotate_absolute
```

这是目前 ROS 图中唯一的动作。正如您之前看到的，它控制乌龟的旋转。通过使用 `ros2 node info <node_name>` 命令，您也已经知道该动作有一个动作客户端（属于 `/teleop_turtle`）和一个动作服务端（属于 `/turtlesim`）。

#### 4.1 ros2 action list -t

动作具有类型，类似于话题和服务。要查找 `/turtle1/rotate_absolute` 的类型，请运行命令：

```bash
ros2 action list -t
```

```text
/turtle1/rotate_absolute [turtlesim/action/RotateAbsolute]
```

在每个动作名称右侧的括号中（在本例中只有 `/turtle1/rotate_absolute`）是动作类型 `turtlesim/action/RotateAbsolute`。当您想要从命令行或代码执行动作时，将需要此信息。

### 5 ros2 action info

您可以使用以下命令进一步内省 `/turtle1/rotate_absolute` 动作：

```bash
ros2 action info /turtle1/rotate_absolute
```

```text
Action: /turtle1/rotate_absolute
Action clients: 1
    /teleop_turtle
Action servers: 1
    /turtlesim
```

这告诉了我们之前通过在每个节点上运行 `ros2 node info` 学到的内容：`/teleop_turtle` 节点有一个动作客户端，而 `/turtlesim` 节点有一个动作服务端，它们都针对 `/turtle1/rotate_absolute` 动作。

### 6 ros2 interface show

在您自己发送或执行动作目标之前，您还需要一条信息，即动作类型的结构。

回想一下，您在运行命令 `ros2 action list -t` 时识别了 `/turtle1/rotate_absolute` 的类型。在终端中输入以下命令及动作类型：

```bash
ros2 interface show turtlesim/action/RotateAbsolute
```

将返回：

```text
# The desired heading in radians
float32 theta
---
# The angular displacement in radians to the starting position
float32 delta
---
# The remaining rotation in radians
float32 remaining
```

此消息中第一个 `---` 上方的部分是**目标请求**的结构（数据类型和名称）。下一部分是**结果**的结构。最后一部分是**反馈**的结构。

### 7 ros2 action send\_goal

现在，让我们使用以下语法从命令行发送一个动作目标：

```bash
ros2 action send_goal <action_name> <action_type> <values>
```

`<values>` 需要采用 YAML 格式。

关注 turtlesim 窗口，并在终端中输入以下命令：

```bash
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 1.57}"
```

```text
Waiting for an action server to become available...
Sending goal:
   theta: 1.57

Goal accepted with ID: f8db8f44410849eaa93d3feb747dd444

Result:
  delta: -1.568000316619873

Goal finished with status: SUCCEEDED
```

您应该看到乌龟在旋转。

所有目标都有一个唯一的 ID，显示在返回消息中。您还可以看到结果，一个名为 `delta` 的字段，即与起始位置的位移。

要查看此目标的反馈，请向 `ros2 action send_goal` 命令添加 `--feedback`：

```bash
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: -1.57}" --feedback
```

```text
Sending goal:
   theta: -1.57

Goal accepted with ID: e6092c831f994afda92f0086f220da27

Feedback:
  remaining: -3.1268222332000732

Feedback:
  remaining: -3.1108222007751465

…

Result:
  delta: 3.1200008392333984

Goal finished with status: SUCCEEDED
```

您将持续收到反馈（剩余的弧度），直到目标完成。

## 总结

动作就像服务，允许您执行长时间运行的任务，提供定期反馈，并且是可取消的。

机器人系统可能会使用动作进行导航。动作目标可以告诉机器人移动到一个位置。当机器人导航到该位置时，它可以沿途发送更新（即反馈），并在到达目的地后发送最终结果消息。

Turtlesim 有一个动作服务端，动作客户端可以向其发送目标以旋转乌龟。在本教程中，您内省了该动作 `/turtle1/rotate_absolute`，以更好地了解动作是什么以及它们是如何工作的。

## 下一步

现在您已经涵盖了所有核心的 ROS 2 概念。这组教程中的最后几个将向您介绍一些使 ROS 2 使用更轻松的工具和技术，首先是[使用 rqt\_console 查看日志](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Using-Rqt-Console/Using-Rqt-Console.html)。

## 相关内容

您可以在[此处](https://design.ros2.org/articles/actions.html)阅读有关 ROS 2 中动作背后的设计决策的更多信息。