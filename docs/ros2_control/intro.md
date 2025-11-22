# 欢迎使用 ros2_control 文档！

`ros2_control` 是一个用于（实时）控制机器人的框架，基于 **ROS 2** 构建。  
它是对 ROS 1 中 `ros_control` 软件包的**重写与改进**，旨在**简化新硬件的集成**并**克服原有设计的若干不足**。

> 💡 **提示**：如果您不熟悉控制理论，请先通过 [维基百科](https://en.wikipedia.org/wiki/Control_theory) 等资源了解基本概念，以便更好地理解本文档中使用的术语。

---

## ros2_control 代码仓库

该框架由 **ros-controls** GitHub 组织维护，包含以下核心仓库：

| 仓库 | 说明 |
|------|------|
| [`ros2_control`](https://github.com/ros-controls/ros2_control) | 框架的核心接口与组件 |
| [`ros2_controllers`](https://github.com/ros-controls/ros2_controllers) | 常用控制器（如前馈命令控制器、关节轨迹控制器、差速驱动控制器等） |
| [`control_toolbox`](https://github.com/ros-controls/control_toolbox) | 控制理论常用实现（如 PID 控制器） |
| [`realtime_tools`](https://github.com/ros-controls/realtime_tools) | 实时支持工具（如实时缓冲区、实时发布者） |
| [`control_msgs`](https://github.com/ros-controls/control_msgs) | 通用控制消息类型 |
| [`kinematics_interface`](https://github.com/ros-controls/kinematics_interface) | 用于集成 C++ 运动学框架 |
| [`gz_ros2_control`](https://github.com/ros-controls/gz_ros2_control) | Gazebo 仿真插件 |
| [`topic_based_hardware_interfaces`](https://github.com/ros-controls/topic_based_hardware_interfaces) | 面向仅支持 ROS 话题通信的仿真器或硬件的硬件接口 |

### 其他相关（未发布）仓库

| 仓库 | 用途 |
|------|------|
| [`ros2_control_demos`](https://github.com/ros-controls/ros2_control_demos) | 常见用例的示例实现，便于快速上手 |
| [`roadmap`](https://github.com/ros-controls/roadmap) | 项目规划与设计文档 |
| [`ros2_control_ci`](https://github.com/ros-controls/ros2_control_ci) | 可复用的 GitHub Actions 与 CI Docker 镜像（Ubuntu/RHEL/Debian） |
| [`.github`](https://github.com/ros-controls/.github) | 组织级文件（如 Issue 模板） |
| [`ros2_control_cmake`](https://github.com/ros-controls/ros2_control_cmake) | 项目专用 CMake 宏 |
| [`control.ros.org`](https://control.ros.org) | 本官方文档站点 |

---

## 开发组织与沟通渠道

### 治理机构
`ros-controls` 项目由 **开源机器人联盟**（Open Source Robotics Alliance, **OSRA**）管理。  
详见：[Project Governance](./governance/governance.html)

### 问题咨询
请使用 **[Robotics Stack Exchange](https://robotics.stackexchange.com/)**，并添加 `ros2_control` 标签。

### PMC 会议
- **频率**：每两周一次（通常在周三）
- **参与方式**：关注 [ROS Discourse](https://discourse.ros.org/) 上的公告，通过 Google Groups 或 Zoom 加入
- **议程与纪要**：[会议文档](https://docs.google.com/document/d/...)

### 项目管理
- 使用 **GitHub Projects**（`ros-controls` 组织下）跟踪开发任务

### 问题报告与功能请求
请在**对应仓库的 Issue Tracker** 中提交：
1. 简明描述问题
2. 提供**可复现问题的最小步骤**
3. 注明操作系统、ROS 版本等环境信息

### 一般性讨论
请使用 **[ROS Discourse](https://discourse.ros.org/)**。