# RQt 概述与使用

## 概述

RQt 是一个图形用户界面框架，以插件形式实现各种工具和接口。你可以在 RQt 中以浮动窗口的形式运行所有现有的 GUI 工具。工具仍然可以以传统的独立方式运行，但 RQt 可以更方便地在一个屏幕布局中管理所有各种窗口。

你可以通过以下命令轻松运行任何 RQt 工具/插件：

```bash
rqt
```

这个 GUI 允许你选择系统中可用的任何插件。你也可以在独立窗口中运行插件。例如，RQt Python 控制台：

```bash
ros2 run rqt_py_console rqt_py_console
```

用户可以使用 Python 或 C++ 为 RQt 创建自己的插件。要查看系统可用的 RQt 插件，请运行：

```bash
ros2 pkg list
```

然后查找以 `rqt_` 开头的包。

## 系统设置

### 从 deb 安装

```bash
sudo apt install ros-kilted-rqt*
```

## RQt 组件结构

RQt 由两个元包组成：

- `rqt` - 核心基础设施模块。
- `rqt_common_plugins` - 常用的调试工具。

## RQt 框架的优势

与从头开始构建自己的 GUI 相比：

- 提供标准化的常见 GUI 过程（启动关闭钩子、恢复先前状态）。
- 多个部件可以停靠在一个窗口中。
- 轻松将现有的 Qt 小部件转换为 RQt 插件。
- 在机器人堆栈交换（ROS 社区网站上的问答站点）获得支持。

从系统架构的角度来看：

- 支持多平台（基本在任何运行 QT 和 ROS 的地方）和多语言（Python, C++）。
- 可管理生命周期：使用通用 API 的 RQt 插件简化了维护和重用。

## 进一步阅读

- [ROS 2 发布关于移植到 ROS 2 的公告](https://discourse.ros.org/)
- [适用于 ROS 1 的 RQt 文档](http://wiki.ros.org/rqt)
- [RQt 简介](http://www.willowgarage.com/blog/2010/07/23/qt-gui-overview-and-roadmap)（Willow Garage 实习生博客文章）
ROS GUI视频:
https://youtu.be/CyP9wHu2PpY