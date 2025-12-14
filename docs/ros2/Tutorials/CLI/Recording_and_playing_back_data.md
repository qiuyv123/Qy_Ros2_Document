# 记录和回放数据 (Recording and playing back data)

**目标：** 记录发布在话题上的数据，以便随时回放和检查。

**教程等级：** 初学者

**时间：** 10 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 1 设置
      * 2 选择一个话题
      * 3 ros2 bag record
      * 4 ros2 bag info
      * 5 ros2 bag play
  * 总结
  * 下一步
  * 相关内容

## 背景

`ros2 bag` 是一个命令行工具，用于记录系统中发布在话题上的数据。它会积累在任意数量的话题上传递的数据，并将其保存在数据库中。然后，您可以回放这些数据以重现测试和实验的结果。记录话题也是分享您的工作并允许他人复现它的好方法。

## 先决条件

作为常规 ROS 2 设置的一部分，您应该已经安装了 `ros2 bag`。

如果您需要安装 ROS 2，请参阅[安装说明](https://docs.ros.org/en/humble/Installation.html)。

本教程涉及先前教程中涵盖的概念，如[节点](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html)和[话题](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)。它还使用了 `turtlesim` 包。

一如既往，不要忘记在打开的**每个新终端**中 source ROS 2 环境。

## 任务

### 1 设置

您将记录 `turtlesim` 系统中的键盘输入，以便稍后保存和回放，因此首先启动 `/turtlesim` 和 `/teleop_turtle` 节点。

打开一个新的终端并运行：

```bash
ros2 run turtlesim turtlesim_node
```

打开另一个终端并运行：

```bash
ros2 run turtlesim turtle_teleop_key
```

作为一种良好的习惯，让我们也创建一个新目录来存储我们保存的录音：

**Linux / macOS**

```bash
mkdir bag_files
cd bag_files
```

**Windows**

```bat
md bag_files
cd bag_files
```

### 2 选择一个话题

`ros2 bag` 只能记录从话题中发布的消息数据。要查看系统的话题列表，请打开一个新终端并运行命令：

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

在话题教程中，您了解到 `/turtle_teleop` 节点在 `/turtle1/cmd_vel` 话题上发布命令以使乌龟在 turtlesim 中移动。

要查看 `/turtle1/cmd_vel` 正在发布的数据，请运行命令：

```bash
ros2 topic echo /turtle1/cmd_vel
```

起初不会显示任何内容，因为 teleop 没有发布任何数据。回到运行 teleop 的终端并选中它，使其处于活动状态。使用箭头键移动乌龟，您将看到数据在运行 `ros2 topic echo` 的终端上发布。

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

### 3 ros2 bag record

#### 3.1 记录单个话题

要记录发布到话题的数据，请使用以下命令语法：

```bash
ros2 bag record <topic_name>
```

在对所选话题运行此命令之前，请打开一个新终端并进入您之前创建的 `bag_files` 目录，因为 rosbag 文件将保存在您运行它的目录中。

运行命令：

```bash
ros2 bag record /turtle1/cmd_vel
```

```text
[INFO] [rosbag2_storage]: Opened database 'rosbag2_2019_10_11-05_18_45'.
[INFO] [rosbag2_transport]: Listening for topics...
[INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/cmd_vel'
[INFO] [rosbag2_transport]: All requested topics are subscribed. Stopping discovery...
```

现在 `ros2 bag` 正在记录发布在 `/turtle1/cmd_vel` 话题上的数据。回到 teleop 终端并再次移动乌龟。具体的移动并不重要，但试着做一个可识别的模式，以便稍后回放数据时能看出来。

按 `Ctrl+C` 停止记录。

数据将积累在一个新的 bag 目录中，名称模式为 `rosbag2_year_month_day-hour_minute_second`。此目录将包含一个 `metadata.yaml` 文件以及记录格式的 bag 文件。

#### 3.2 记录多个话题

您还可以记录多个话题，并更改 `ros2 bag` 保存的文件名。

运行以下命令：

```bash
ros2 bag record -o subset /turtle1/cmd_vel /turtle1/pose
```

```text
[INFO] [rosbag2_storage]: Opened database 'subset'.
[INFO] [rosbag2_transport]: Listening for topics...
[INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/cmd_vel'
[INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/pose'
[INFO] [rosbag2_transport]: All requested topics are subscribed. Stopping discovery...
```

`-o` 选项允许您为 bag 文件选择一个唯一的名称。在这个例子中，字符串 `subset` 就是文件名。

要一次记录多个话题，只需列出每个话题，用空格分隔。在本例中，上面的命令输出确认了两个话题都在被记录。

您可以移动乌龟，完成后按 `Ctrl+C`。

> **注意**
>
> 您可以向命令添加另一个选项 `-a`，它会记录系统上的**所有**话题。

### 4 ros2 bag info

您可以通过运行以下命令查看有关录音的详细信息：

```bash
ros2 bag info <bag_file_name>
```

对 `subset` bag 文件运行此命令将返回该文件的信息列表：

```bash
ros2 bag info subset
```

```text
Files:             subset.db3
Bag size:          228.5 KiB
Storage id:        sqlite3
Duration:          48.47s
Start:             Oct 11 2019 06:09:09.12 (1570799349.12)
End                Oct 11 2019 06:09:57.60 (1570799397.60)
Messages:          3013
Topic information: Topic: /turtle1/cmd_vel | Type: geometry_msgs/msg/Twist | Count: 9 | Serialization Format: cdr
                   Topic: /turtle1/pose | Type: turtlesim/msg/Pose | Count: 3004 | Serialization Format: cdr
```

### 5 ros2 bag play

在回放 bag 文件之前，请在运行 teleop 的终端中输入 `Ctrl+C`。然后确保您的 turtlesim 窗口可见，以便您可以观看 bag 文件的运行效果。

输入命令：

```bash
ros2 bag play subset
```

```text
[INFO] [rosbag2_storage]: Opened database 'subset'.
```

您的乌龟将遵循您在记录时输入的相同路径（虽然不是 100% 精确；turtlesim 对系统时序的微小变化很敏感）。

因为 `subset` 文件记录了 `/turtle1/pose` 话题，所以只要您运行着 turtlesim，`ros2 bag play` 命令就不会退出，即使您没有移动。

这是因为只要 `/turtlesim` 节点处于活动状态，它就会定期在 `/turtle1/pose` 话题上发布数据。您可能已经在上面的 `ros2 bag info` 示例结果中注意到，`/turtle1/cmd_vel` 话题的 `Count` 信息只有 9；这就是我们在记录时按下箭头键的次数。

请注意，`/turtle1/pose` 的 `Count` 值超过 3000；在我们记录期间，数据在该话题上发布了 3000 次。

要了解位置数据发布的频率，您可以运行以下命令：

```bash
ros2 topic hz /turtle1/pose
```

## 总结

您可以使用 `ros2 bag` 命令记录在 ROS 2 系统中的话题上传递的数据。无论您是与他人分享您的工作，还是内省您自己的实验，这都是一个非常有用的工具。

## 下一步

您已经完成了“初学者：CLI 工具”教程！下一步是处理“初学者：客户端库”教程，从[创建工作空间](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html)开始。

## 相关内容

可以在[此处](https://www.google.com/search?q=https://github.com/ros2/rosbag2/tree/humble/README.md)找到对 `ros2 bag` 更详尽的解释。有关 QoS 兼容性和 `ros2 bag` 的更多信息，请参阅 [rosbag2：覆盖 QoS 策略](https://docs.ros.org/en/humble/How-To-Guides/Overriding-QoS-Policies-For-Recording-And-Playback.html)。