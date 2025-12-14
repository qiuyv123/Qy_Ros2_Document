# 使用 `turtlesim`、`ros2` 和 `rqt`

**目标**：安装并使用 `turtlesim` 包和 `rqt` 工具，为后续教程做准备。

**教程级别**：初学者

**时间**：15 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 1 安装 turtlesim
      * 2 启动 turtlesim
      * 3 使用 turtlesim
      * 4 安装 rqt
      * 5 使用 rqt
      * 6 重映射 (Remapping)
      * 7 关闭 turtlesim
  * 总结
  * 下一步
  * 相关内容

## 背景

Turtlesim 是一个用于学习 ROS 2 的轻量级模拟器。它演示了 ROS 2 在最基础层面的功能，让您对以后使用真实机器人或机器人仿真有一个大致的了解。

`ros2` 工具是用户管理 ROS 系统、进行自省（introspection）并与之交互的方式。它支持针对系统及其操作不同方面的多种命令。人们可以使用它来启动节点、设置参数、监听话题等等。`ros2` 工具是 ROS 2 核心安装的一部分。

`rqt` 是一个用于 ROS 2 的图形用户界面 (GUI) 工具。在 `rqt` 中完成的所有操作都可以在命令行上完成，但 `rqt` 提供了一种更用户友好的方式来操作 ROS 2 元素。

本教程涉及核心的 ROS 2 概念，如节点、话题和服务。所有这些概念将在以后的教程中详细阐述；现在，您只需设置工具并感受一下它们。

## 先决条件

上一篇教程[配置环境](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html)将向您展示如何设置环境。

## 任务

### 1 安装 turtlesim

一如既往，首先在新的终端中 source 您的设置文件，如[上一篇教程](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html)所述。

为您使用的 ROS 2 发行版安装 turtlesim 包：

**Linux (Ubuntu)**

```bash
sudo apt update
sudo apt install ros-humble-turtlesim
```

检查包是否已安装，请运行以下命令，该命令应返回 turtlesim 可执行文件的列表：

```bash
ros2 pkg executables turtlesim
```

输出应包含：

```text
turtlesim draw_square
turtlesim mimic
turtlesim turtle_teleop_key
turtlesim turtlesim_node
```

### 2 启动 turtlesim

要启动 turtlesim，请在终端中输入以下命令：

```bash
ros2 run turtlesim turtlesim_node
```

终端将显示类似以下的消息：

```text
[INFO] [turtlesim]: Starting turtlesim with node name /turtlesim
[INFO] [turtlesim]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]
```

在命令下方，您将看到来自节点的消息。那里可以看到默认海龟的名称以及它生成的坐标。

模拟器窗口应该会出现，中间有一只随机样式的海龟。

### 3 使用 turtlesim

打开一个新的终端并再次 source ROS 2。

现在您将运行一个新的节点来控制第一个节点中的海龟：

```bash
ros2 run turtlesim turtle_teleop_key
```

此时您应该打开了三个窗口：一个运行 `turtlesim_node` 的终端，一个运行 `turtle_teleop_key` 的终端，以及 turtlesim 窗口。排列这些窗口，以便您可以看到 turtlesim 窗口，但同时保持运行 `turtle_teleop_key` 的终端处于激活状态，以便您可以控制 turtlesim 中的海龟。

使用键盘上的方向键来控制海龟。它会在屏幕上移动，并使用其附带的“画笔”绘制其移动路径。

> **注意**
> 按下方向键只会让海龟移动很短的距离然后停止。这是因为，实际上，如果操作员与机器人失去连接，您不希望机器人继续执行指令。

您可以使用相应命令的 `list` 子命令查看节点及其关联的话题、服务和动作：

```bash
ros2 node list
ros2 topic list
ros2 service list
ros2 action list
```

您将在接下来的教程中了解有关这些概念的更多信息。由于本教程的目标只是获得 turtlesim 的总体概览，您将使用 `rqt` 调用一些 turtlesim 服务并与 `turtlesim_node` 交互。

### 4 安装 rqt

打开一个新的终端来安装 `rqt` 及其插件：

**Ubuntu Linux**

```bash
sudo apt update
sudo apt install '~nros-humble-rqt*'
```

运行 rqt：

```bash
rqt
```

### 5 使用 rqt

首次运行 rqt 时，窗口将是空白的。别担心；只需从顶部的菜单栏中选择 **Plugins** \> **Services** \> **Service Caller**。

> **注意**
> rqt 可能需要一些时间来定位所有插件。如果您点击 **Plugins** 但没有看到 **Services** 或任何其他选项，您应该关闭 rqt 并在终端中输入命令 `rqt --force-discover`。

使用 **Service** 下拉列表左侧的刷新按钮，确保 turtlesim 节点的所有服务都可用。

点击 **Service** 下拉列表查看 turtlesim 的服务，并选择 `/spawn` 服务。

#### 5.1 尝试 spawn 服务

让我们使用 rqt 来调用 `/spawn` 服务。您可以从它的名字猜到 `/spawn` 将在 turtlesim 窗口中创建另一只海龟。

双击 **Expression** 列中空的单引号之间，为新海龟起一个唯一的名称，例如 `turtle2`。您可以看到此表达式对应于 `name` 的值，且类型为 `string`。

接下来输入生成新海龟的有效坐标，例如 `x = 1.0` 和 `y = 1.0`。

> **注意**
> 如果您尝试生成一只与现有海龟同名（如默认的 `turtle1`）的新海龟，您将在运行 `turtlesim_node` 的终端中收到错误消息：
> `[ERROR] [turtlesim]: A turtle named [turtle1] already exists`

要生成 `turtle2`，您需要点击 rqt 窗口右上角的 **Call** 按钮来调用服务。

如果服务调用成功，您应该会看到一只新海龟（同样是随机设计）在您输入的 `x` 和 `y` 坐标处生成。

如果您刷新 rqt 中的服务列表，您将看到除了 `/turtle1/...` 之外，现在还有与新海龟相关的服务 `/turtle2/...`。

#### 5.2 尝试 set\_pen 服务

现在让我们使用 `/set_pen` 服务给 `turtle1` 一个独特的画笔：

`r`、`g` 和 `b` 的值在 0 到 255 之间，用于设置 `turtle1` 画笔的颜色，`width` 设置线条的粗细。

要让 `turtle1` 画出明显的红线，请将 `r` 的值更改为 255，将 `width` 的值更改为 5。不要忘记在更新值后调用服务。

如果您返回运行 `turtle_teleop_key` 的终端并按下方向键，您将看到 `turtle1` 的画笔已经改变。

您可能也注意到了没办法移动 `turtle2`。这是因为没有针对 `turtle2` 的 teleop（遥操作）节点。

### 6 重映射 (Remapping)

您需要第二个 teleop 节点来控制 `turtle2`。但是，如果您尝试运行与之前相同的命令，您会发现它控制的也是 `turtle1`。改变这种行为的方法是重映射 `cmd_vel` 话题。

在一个新的终端中，source ROS 2，并运行：

```bash
ros2 run turtlesim turtle_teleop_key --ros-args --remap turtle1/cmd_vel:=turtle2/cmd_vel
```

现在，当此终端处于激活状态时，您可以移动 `turtle2`；而当另一个运行 `turtle_teleop_key` 的终端处于激活状态时，您可以移动 `turtle1`。

### 7 关闭 turtlesim

要停止模拟，您可以在 `turtlesim_node` 终端中输入 `Ctrl + C`，在 `turtle_teleop_key` 终端中输入 `q`。

## 总结

使用 turtlesim 和 rqt 是学习 ROS 2 核心概念的好方法。

## 下一步

既然您已经安装并运行了 turtlesim 和 rqt，并且了解了它们的工作原理，那么让我们在下一个教程[理解节点 (Understanding nodes)](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html) 中深入探讨第一个 ROS 2 核心概念。

## 相关内容

`turtlesim` 包可以在 [ros\_tutorials](https://github.com/ros/ros_tutorials/tree/humble/turtlesim) 仓库中找到。