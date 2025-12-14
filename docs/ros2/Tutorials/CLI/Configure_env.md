# 配置环境

**目标**：本教程将向您展示如何准备 ROS 2 环境。

**教程级别**：初学者

**时间**：5 分钟

**目录**

  * 背景
  * 先决条件
  * 任务
      * 1 Source 设置文件
      * 2 在 Shell 启动脚本中添加 source 命令
      * 3 检查环境变量
  * 总结
  * 下一步

## 背景

ROS 2 依赖于使用 Shell 环境组合工作空间（workspaces）的概念。“工作空间”是一个 ROS 术语，指您系统上用来开发 ROS 2 的位置。核心的 ROS 2 工作空间被称为“底层（underlay）”。后续的本地工作空间被称为“覆盖层（overlays）”。在开发 ROS 2 时，您通常会有多个并发活动的工作空间。

组合工作空间使得针对不同版本的 ROS 2 或不同的包集合进行开发变得更加容易。它还允许在同一台计算机上安装多个 ROS 2 发行版（或“distros”，例如 Dashing 和 Eloquent）并在它们之间进行切换。

这是通过在每次打开新 Shell 时 `source`（执行）设置文件，或者将 `source` 命令一次性添加到 Shell 启动脚本中来实现的。如果不 `source` 设置文件，您将无法访问 ROS 2 命令，也无法找到或使用 ROS 2 包。换句话说，您将无法使用 ROS 2。

## 先决条件

在开始这些教程之前，请按照 ROS 2 **安装**页面上的说明安装 ROS 2。

本教程中使用的命令假设您遵循了操作系统的二进制包安装指南（Linux 的 deb 包）。如果您是源码编译安装的，仍然可以参考本教程，但设置文件的路径可能会有所不同。此外，如果您是通过源码安装的，将无法使用 `sudo apt install ros-<distro>-<package>` 命令（在初学者教程中经常使用）。

如果您使用的是 Linux 或 macOS，但尚不熟悉 Shell，**本教程**会有所帮助。

## 任务

### 1 Source 设置文件

您需要在打开的每个新 Shell 中运行此命令，才能访问 ROS 2 命令，如下所示：

**Linux / macOS**

```bash
source /opt/ros/humble/setup.bash
```

**Windows**

```bat
call C:\dev\ros2_humble\local_setup.bat
```

*(注意：以上路径取决于您的安装位置，Windows 命令通常会有所不同，具体取决于您的安装目录)*

如果您使用的不是 bash，请将 `.bash` 替换为您的 Shell。可能的值包括：`setup.bash`、`setup.sh`、`setup.zsh`。

> **注意**
> 确切的命令取决于您安装 ROS 2 的位置。如果遇到问题，请确保文件路径指向您的安装目录。

### 2 在 Shell 启动脚本中添加 source 命令

如果您不想每次打开新 Shell 时都必须 `source` 设置文件（跳过任务 1），则可以将该命令添加到您的 Shell 启动脚本中：

**Linux / macOS**

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

要撤消此操作，请找到系统的 Shell 启动脚本并删除附加的 source 命令。

### 3 检查环境变量

Sourcing ROS 2 设置文件将设置操作 ROS 2 所需的多个环境变量。如果您在查找或使用 ROS 2 包时遇到问题，请使用以下命令确保您的环境已正确设置：

**Linux / macOS**

```bash
printenv | grep -i ROS
```

**Windows**

```cmd
set | findstr -i ROS
```

检查 `ROS_DISTRO` 和 `ROS_VERSION` 等变量是否已设置。

```text
ROS_VERSION=2
ROS_PYTHON_VERSION=3
ROS_DISTRO=humble
```

如果环境变量未正确设置，请返回您所遵循的安装指南中的 ROS 2 包安装部分。如果您需要更具体的帮助（因为环境设置文件可能来自不同的位置），您可以从社区**获取解答**。

#### 3.1 `ROS_DOMAIN_ID` 变量

有关 ROS 域 ID 的详细信息，请参阅**域 ID** 文章。

一旦确定了用于您的 ROS 2 节点组的唯一整数，就可以使用以下命令设置环境变量：

**Linux / macOS**

```bash
export ROS_DOMAIN_ID=<your_domain_id>
```

**Windows**

```cmd
set ROS_DOMAIN_ID=<your_domain_id>
```

要在 Shell 会话之间保持此设置，可以将命令添加到您的 Shell 启动脚本中：

```bash
echo "export ROS_DOMAIN_ID=<your_domain_id>" >> ~/.bashrc
```

#### 3.2 `ROS_LOCALHOST_ONLY` 变量

默认情况下，ROS 2 通信不限于 localhost（本地主机）。`ROS_LOCALHOST_ONLY` 环境变量允许您将 ROS 2 通信限制为仅在 localhost 进行。这意味着您的 ROS 2 系统及其话题、服务和动作对本地网络上的其他计算机不可见。

在某些环境（例如教室）中，使用 `ROS_LOCALHOST_ONLY` 很有帮助，因为在这些环境中，多个机器人可能会发布到同一话题，从而导致奇怪的行为。您可以使用以下命令设置环境变量：

**Linux / macOS**

```bash
export ROS_LOCALHOST_ONLY=1
```

**Windows**

```cmd
set ROS_LOCALHOST_ONLY=1
```

要在 Shell 会话之间保持此设置，可以将命令添加到您的 Shell 启动脚本中：

```bash
echo "export ROS_LOCALHOST_ONLY=1" >> ~/.bashrc
```

## 总结

ROS 2 开发环境在使用前需要正确配置。这可以通过两种方式完成：要么在打开的每个新 Shell 中 `source` 设置文件，要么将 source 命令添加到您的启动脚本中。

如果您在定位或使用 ROS 2 包时遇到任何问题，首先应该做的是检查您的环境变量，确保它们已设置为您预期的版本和发行版。

## 下一步

既然您已经拥有一个可工作的 ROS 2 安装环境，并且知道如何 source 其设置文件，您就可以开始使用 **turtlesim 工具**来学习 ROS 2 的来龙去脉了。