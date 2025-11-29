# 日志与日志记录器配置 (Logging and logger configuration)

## 概述

ROS 2 中的日志子系统旨在将日志消息传递给多种目标，包括：

  * 控制台（如果有连接）
  * 磁盘上的日志文件（如果本地存储可用）
  * ROS 2 网络上的 `/rosout` 话题

**默认情况下**，ROS 2 节点的日志消息会输出到控制台（标准错误 stderr）、磁盘上的日志文件以及 ROS 2 网络上的 `/rosout` 话题。所有这些目标都可以针对每个节点单独启用或禁用。

本文档的其余部分将介绍日志子系统背后的一些理念。

## 严重级别

日志消息关联了一个严重级别：`DEBUG`、`INFO`、`WARN`、`ERROR` 或 `FATAL`，按严重程度递增排序。

日志记录器（Logger）只会处理严重级别等于或高于该记录器所选指定级别的消息。

每个节点都有一个关联的日志记录器，该记录器会自动包含节点的名称和命名空间。如果节点的名称在外部被重映射（Remap）为源代码中定义以外的名称，这也会反映在日志记录器名称中。此外，也可以创建使用特定名称的非节点日志记录器。

**日志记录器名称代表一种层级结构**。如果名为 "abc.def" 的日志记录器级别未设置，它将推迟使用其父级 "abc" 的级别；如果该级别也未设置，则使用默认的日志记录器级别。当日志记录器 "abc" 的级别发生更改时，其所有后代（例如 "abc.def"、"abc.ghi.jkl"）的级别都会受到影响，除非它们的级别已被显式设置。

-----

## API 接口

以下是 ROS 2 日志基础设施的终端用户应使用的 API，按客户端库划分。

=== "CPP" 
```cpp

        以下宏定义用于 C++ 开发。每个 API 都接受一个 `rclcpp::Logger` 对象作为第一个参数。可以通过调用 `node->get_logger()` 从节点 API 获取（推荐），或者构建一个独立的 `rclcpp::Logger` 对象。

        > **注意：** 下列宏中的 `{DEBUG,INFO,WARN,ERROR,FATAL}` 代表可替换的严重级别占位符。

        * **`RCLCPP_{LEVEL}`**: 每次运行此行代码时，输出给定的 printf 风格的消息。
        * **`RCLCPP_{LEVEL}_ONCE`**: 仅在第一次运行此行代码时，输出给定的 printf 风格的消息。
        * **`RCLCPP_{LEVEL}_EXPRESSION`**: 仅当给定的表达式为真时，输出给定的 printf 风格的消息。
        * **`RCLCPP_{LEVEL}_FUNCTION`**: 仅当给定的函数返回真时，输出给定的 printf 风格的消息。
        * **`RCLCPP_{LEVEL}_SKIPFIRST`**: 除了第一次运行此行代码外，其余每次都输出给定的 printf 风格的消息。
        * **`RCLCPP_{LEVEL}_THROTTLE`**: 输出给定的 printf 风格的消息，但输出频率不超过给定的速率（以整数毫秒为单位）。
        * **`RCLCPP_{LEVEL}_SKIPFIRST_THROTTLE`**: 输出给定的 printf 风格的消息，频率不超过给定的速率（以整数毫秒为单位），但跳过第一次。

        **流式 (Stream) 风格 API:**

        * **`RCLCPP_{LEVEL}_STREAM`**: 每次运行此行代码时，输出给定的 C++ 流式 (stream-style) 消息。
        * **`RCLCPP_{LEVEL}_STREAM_ONCE`**: 仅在第一次运行此行代码时，输出给定的 C++ 流式消息。
        * **`RCLCPP_{LEVEL}_STREAM_EXPRESSION`**: 仅当给定的表达式为真时，输出给定的 C++ 流式消息。
        * **`RCLCPP_{LEVEL}_STREAM_FUNCTION`**: 仅当给定的函数返回真时，输出给定的 C++ 流式消息。
        * **`RCLCPP_{LEVEL}_STREAM_SKIPFIRST`**: 除了第一次运行此行代码外，其余每次都输出给定的 C++ 流式消息。
        * **`RCLCPP_{LEVEL}_STREAM_THROTTLE`**: 输出给定的 C++ 流式消息，但输出频率不超过给定的速率（以整数毫秒为单位）。
        * **`RCLCPP_{LEVEL}_STREAM_SKIPFIRST_THROTTLE`**: 输出给定的 C++ 流式消息，频率不超过给定的速率（以整数毫秒为单位），但跳过第一次。

        **通用工具:**

        * `rcutils_logging_set_logger_level`: 将特定日志记录器名称的日志级别设置为给定的严重级别。
        * `rcutils_logging_get_logger_effective_level`: 给定一个日志记录器名称，返回其日志级别（可能未设置）。

        -----
```
=== "Python"
```python
        logger.{debug,info,warning,error,fatal} 将给定的 Python 字符串输出到日志基础设施。这些调用接受以下关键字参数来控制行为：

        throttle_duration_sec: 如果不为 None，则指定节流（频率限制）的间隔时长，单位为浮点秒。

        skip_first: 如果为 True，则除了第一次执行到此行代码外，其余每次都输出消息（即跳过第一次）。

        once: 如果为 True，仅在第一次执行到此行代码时输出消息。

        rclpy.logging.set_logger_level 将特定日志记录器名称（Logger Name）的日志级别设置为给定的严重性级别（Severity Level）。

        rclpy.logging.get_logger_effective_level 给定一个日志记录器名称，返回其当前的日志级别（该级别可能处于未设置状态）。
```
## 配置

由于 `rclcpp` 和 `rclpy` 使用相同的底层日志基础设施，因此配置选项是相同的。

### 环境变量

以下环境变量控制 ROS 2 日志记录器的某些方面。请注意，这些是**进程范围 (process-wide)** 的设置，因此它们适用于该进程中的所有节点。

  * **`ROS_LOG_DIR`**

      * 控制用于将日志消息写入磁盘（如果已启用）的日志目录。如果不为空，则使用此变量中指定的精确目录。如果为空，则使用 `ROS_HOME` 环境变量的内容构建形式为 `$ROS_HOME/.log` 的路径。在所有情况下，`~` 字符都会扩展为用户的 HOME 目录。

  * **`ROS_HOME`**

      * 控制用于各种 ROS 文件（包括日志和配置文件）的主目录。在日志记录的上下文中，此变量用于构建日志文件目录的路径。如果不为空，则将此变量的内容用作 ROS\_HOME 路径。在所有情况下，`~` 字符都会扩展为用户的 HOME 目录。

  * **`RCUTILS_LOGGING_USE_STDOUT`**

      * 控制输出消息流向的目标。如果未设置或为 `0`，使用标准错误 (`stderr`)。如果是 `1`，使用标准输出 (`stdout`)。

  * **`RCUTILS_LOGGING_BUFFERED_STREAM`**

      * 控制日志流（按 `RCUTILS_LOGGING_USE_STDOUT` 配置）是行缓冲还是无缓冲。如果未设置，则使用流的默认设置（通常 stdout 为行缓冲，stderr 为无缓冲）。如果是 `0`，强制流为无缓冲。如果是 `1`，强制流为行缓冲。

  * **`RCUTILS_COLORIZED_OUTPUT`**

      * 控制输出消息时是否使用颜色。如果未设置，则根据平台和控制台是否为 TTY 自动确定。如果是 `0`，强制禁用颜色输出。如果是 `1`，强制启用颜色输出。

  * **`RCUTILS_CONSOLE_OUTPUT_FORMAT`**

      * 控制每条日志消息输出的字段。可用字段包括：
          * `{severity}` - 严重级别。
          * `{name}` - 日志记录器的名称（可能为空）。
          * `{message}` - 日志消息（可能为空）。
          * `{function_name}` - 调用此消息的函数名（可能为空）。
          * `{file_name}` - 调用此消息的文件名（可能为空）。
          * `{time}` - 自 epoch 以来的时间（秒）。
          * `{time_as_nanoseconds}` - 自 epoch 以来的时间（纳秒）。
          * `{date_time_with_ms}` - ISO 格式的时间，例如 `2024-06-11 09:29:19.304`。
          * `{line_number}` - 调用此消息的行号（可能为空）。

    如果未给出格式，则使用默认值：`[{severity}] [{time}] [{name}]: {message}`。

**转义字符语法：**
`RCUTILS_CONSOLE_OUTPUT_FORMAT` 也支持以下转义字符语法。

| 转义字符语法 | 代表字符 |
| :--- | :--- |
| `\a` | Alert (警报) |
| `\b` | Backspace (退格) |
| `\n` | New line (换行) |
| `\r` | Carriage return (回车) |
| `\t` | Horizontal tab (水平制表符) |

-----

### 节点创建

在初始化 ROS 2 节点时，可以通过节点选项控制某些行为。由于这些是**每个节点 (per-node)** 的选项，即使节点组合在同一个进程中，也可以对不同的节点进行不同的设置。

  * **`log_levels`**

      * 用于此特定节点内组件的日志级别。可以通过以下方式设置：
        `ros2 run demo_nodes_cpp talker --ros-args --log-level talker:=DEBUG`

  * **`external_log_config_file`**

      * 用于配置后端日志记录器的外部文件。如果为 NULL，将使用默认配置。请注意，此文件的格式是特定于后端的（对于默认的 spdlog 后端日志记录器，目前尚未实现）。可以通过以下方式设置：
        `ros2 run demo_nodes_cpp talker --ros-args --log-config-file log-config.txt`

  * **`log_stdout_disabled`**

      * 是否禁用将日志消息写入控制台。可以通过以下方式完成：
        `ros2 run demo_nodes_cpp talker --ros-args --disable-stdout-logs`

  * **`log_rosout_disabled`**

      * 是否禁用将日志消息输出到 `/rosout`。这可以显著节省网络带宽，但外部观察者将无法监控日志。可以通过以下方式完成：
        `ros2 run demo_nodes_cpp talker --ros-args --disable-rosout-logs`

  * **`log_ext_lib_disabled`**

      * 是否完全禁用使用外部日志库。在某些情况下这可能会更快，但也意味着日志将不会写入磁盘。可以通过以下方式完成：
        `ros2 run demo_nodes_cpp talker --ros-args --disable-external-lib-logs`

-----

## 日志子系统设计

下图显示了日志子系统的五个主要部分及其交互方式。
![alt text](ros2_logging_architecture.png)
### rcutils

`rcutils` 拥有一个日志实现，可以根据特定格式（参见上面的[配置](https://www.google.com/search?q=%23%E9%85%8D%E7%BD%AE)）格式化日志消息，并将这些日志消息输出到控制台。`rcutils` 实现了一个完整的日志解决方案，但也允许更高级别的组件以依赖注入（dependency-injection）模型将自己插入到日志基础设施中。当我们讨论下面的 `rcl` 层时，这一点将变得更加明显。

> **注意：** 这是一个**每进程 (per-process)** 的日志实现，因此在此级别配置的任何内容都将影响整个进程，而不仅仅是单个节点。

### rcl\_logging\_spdlog

`rcl_logging_spdlog` 实现了 `rcl_logging_interface` API，从而为 `rcl` 层提供外部日志服务。具体来说，`rcl_logging_spdlog` 实现接收格式化的日志消息，并使用 `spdlog` 库将它们写入磁盘上的日志文件，通常位于 `~/.ros/log` 中（尽管这是可配置的；参见上面的[配置](https://www.google.com/search?q=%23%E9%85%8D%E7%BD%AE)）。

### rcl

`rcl` 中的日志子系统使用 `rcutils` 和 `rcl_logging_spdlog` 来提供大部分 ROS 2 日志服务。当日志消息传入时，`rcl` 决定将其发送到何处。日志消息可以传递到的 3 个主要位置（单个节点可以启用其中任何组合）：

1.  通过 `rcutils` 层发送到**控制台**。
2.  通过 `rcl_logging_spdlog` 层发送到**磁盘**。
3.  通过 RMW 层发送到 ROS 2 网络上的 **/rosout 话题**。

### rclcpp

这是位于 `rcl` API 之上的主要 ROS 2 C++ API。在日志记录的上下文中，`rclcpp` 提供了 `RCLCPP_` 日志宏；有关完整列表，请参阅上面的 [API 接口](https://www.google.com/search?q=%23api-%E6%8E%A5%E5%8F%A3)。当 `RCLCPP_` 宏之一运行时，它会将节点的当前严重级别与宏的严重级别进行检查。如果宏的严重级别大于或等于节点严重级别，则消息将被格式化并输出到当前配置的所有位置。

> **注意：** `rclcpp` 对日志调用使用全局互斥锁 (global mutex)，因此同一进程内的所有日志调用最终都是单线程的。

### rclpy

这是位于 `rcl` API 之上的主要 ROS 2 Python API。在日志记录的上下文中，`rclpy` 提供了 `logger.debug` 风格的函数。当 `logger.debug` 函数之一运行时，它会将节点的当前严重级别与宏的严重级别进行检查。如果宏的严重级别大于或等于节点严重级别，则消息将被格式化并输出到当前配置的所有位置。

-----

## 日志用法

### C++

请参阅 [rclcpp logging demo](https://www.google.com/search?q=https://github.com/ros2/demos/tree/master/logging_demo) 获取一些简单示例。

### Python

请参阅 [logging demo](https://www.google.com/search?q=https://github.com/ros2/demos/tree/master/logging_demo) 获取示例用法。

-----

Would you like me to create a specific `RCLCPP` logging example snippet demonstrating how to set up a custom logger with a specific severity threshold?