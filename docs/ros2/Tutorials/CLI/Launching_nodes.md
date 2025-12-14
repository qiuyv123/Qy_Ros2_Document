# 启动节点 (Launching nodes)

**目标：** 使用命令行工具一次启动多个节点。

**教程等级：** 初学者

**时间：** 5 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 运行启动文件
      * (可选) 控制 Turtlesim 节点
  * 总结
  * 下一步

## 背景

在大多数入门教程中，您每运行一个新节点都要打开一个新的终端。随着您创建更复杂的系统，同时运行的节点越来越多，打开终端并重新输入配置详细信息变得非常繁琐。

启动文件 (Launch files) 允许您同时启动和配置多个包含 ROS 2 节点的可执行文件。

使用 `ros2 launch` 命令运行单个启动文件将一次性启动整个系统——包括所有节点及其配置。

## 先决条件

在开始这些教程之前，请按照 ROS 2 **安装**页面上的说明安装 ROS 2。

本教程中使用的命令假设您遵循了操作系统的二进制包安装指南（Linux 的 deb 包）。如果您是源码编译安装的，仍然可以参考本教程，但设置文件的路径可能会有所不同。此外，如果您是通过源码安装的，将无法使用 `sudo apt install ros-<distro>-<package>` 命令（在初学者教程中经常使用）。

如果您使用的是 Linux 但尚不熟悉 shell，**本教程**会有所帮助。

一如既往，不要忘记在您打开的**每一个新终端**中 source ROS 2。

## 任务

### 运行启动文件

打开一个新的终端并运行：

```bash
ros2 launch turtlesim multisim.launch.py
```

此命令将运行以下启动文件：

```python
from launch import LaunchDescription
import launch_ros.actions

def generate_launch_description():
    return LaunchDescription([
        launch_ros.actions.Node(
            namespace='turtlesim1', package='turtlesim',
            executable='turtlesim_node', output='screen'),
        launch_ros.actions.Node(
            namespace='turtlesim2', package='turtlesim',
            executable='turtlesim_node', output='screen'),
    ])
```

> **注意**
>
> 上面的启动文件是用 Python 编写的，但您也可以使用 XML 和 YAML 来创建启动文件。您可以在**使用 XML、YAML 和 Python 编写 ROS 2 启动文件**中查看这些不同 ROS 2 启动格式的比较。

这将运行两个 turtlesim 节点：

目前，不必担心此启动文件的内容。您可以在 **ROS 2 launch 教程**中找到有关 ROS 2 launch 的更多信息。

### (可选) 控制 Turtlesim 节点

现在这些节点正在运行，您可以像控制任何其他 ROS 2 节点一样控制它们。例如，您可以通过打开另外两个终端并运行以下命令让海龟向相反方向移动：

在第二个终端中：

```bash
ros2 topic pub  /turtlesim1/turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

在第三个终端中：

```bash
ros2 topic pub  /turtlesim2/turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: -1.8}}"
```

运行这些命令后，您应该会看到类似以下内容：

## 总结

您目前所做工作的重要性在于，您用一条命令运行了两个 turtlesim 节点。一旦您学会编写自己的启动文件，您将能够使用 `ros2 launch` 命令以类似的方式运行多个节点并设置它们的配置。

有关 ROS 2 启动文件的更多教程，请参阅**主要启动文件教程页面**。

## 下一步

在下一个教程**记录和回放数据**中，您将了解另一个有用的工具 `ros2 bag`。