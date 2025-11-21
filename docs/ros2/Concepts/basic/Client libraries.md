# 客户端库（Client libraries）

## 目录

- 概述  
- 支持的客户端库  
- C++ 客户端库（rclcpp 包）  
- Python 客户端库（rclpy 包）  
- 社区维护的库  
- 公共功能：rcl  
- 语言特定功能  
- 演示  
- 与 ROS 1 的对比  
- 总结  

---

## 概述

**客户端库**（Client libraries）是用户编写 ROS 2 代码所使用的 API。通过客户端库，用户可以访问 ROS 2 的核心概念，如节点、话题、服务等。

客户端库支持多种编程语言，使用户能够选择最适合其应用需求的语言。例如：
- **Python** 适合快速原型开发（如可视化工具）
- **C++** 适合对性能和效率要求高的模块

使用不同客户端库编写的节点**可以互相通信**，因为所有客户端库都实现了**代码生成器**，使用户能在各自语言中操作 ROS 2 接口文件。

除了语言特定的通信工具，客户端库还向用户暴露了使 ROS 成为“ROS”的**核心功能**，例如：
- 名称与命名空间  
- 时间（真实时间或仿真时间）  
- 参数  
- 控制台日志  
- 线程模型  
- 进程内通信（Intra-process communication）  

---

## 支持的客户端库

C++ 客户端库（**rclcpp**）和 Python 客户端库（**rclpy**）均基于 **rcl**（ROS Client Library）的公共功能构建。

---

## C++ 客户端库（rclcpp 包）

**rclcpp** 是 ROS 2 的 C++ 客户端库，提供了符合 C++ 习惯的用户接口，支持创建节点、发布者、订阅者等所有 ROS 客户端功能。

- 基于 **rcl** 和 **rosidl API** 构建  
- 专为与 **rosidl_generator_cpp** 生成的 C++ 消息配合使用而设计  
- 充分利用 C++ 和 C++17 特性，提供简洁易用的接口  
- 通过复用 rcl 的实现，与其他客户端库保持行为一致性  

**资源**：
- GitHub 仓库：[ros2/rclcpp](https://github.com/ros2/rclcpp)  
- API 文档：[https://docs.ros.org/en/kilted/p/rclcpp/](https://docs.ros.org/en/kilted/p/rclcpp/)  

---

## Python 客户端库（rclpy 包）

**rclpy** 是 ROS 2 的 Python 客户端库，是 rclcpp 的 Python 对应版本。

- 同样基于 **rcl C API** 实现  
- 提供符合 Python 习惯的接口，使用原生 Python 类型（如列表、上下文对象）  
- 通过 rcl API 保证与其他客户端库在功能和行为上的一致性  
- 负责管理执行模型（如使用 `threading.Thread` 运行 rcl 函数）

**消息处理机制**：
- 为每个 ROS 消息生成 Python 类
- 用户操作的是 **Python 原生消息对象**
- 当需要传递给 rcl 层时，才转换为 **C 版本消息**
- **同一进程内的发布者与订阅者通信会跳过转换**，以提升性能

**资源**：
- GitHub 仓库：[ros2/rclpy](https://github.com/ros2/rclpy)  
- API 文档：[https://docs.ros.org/en/kilted/p/rclpy/](https://docs.ros.org/en/kilted/p/rclpy/)  

---

## 社区维护的库

除 C++ 和 Python 外，ROS 2 社区还维护了多种语言的客户端库：

- **Ada**：包含 rcl 绑定、消息生成器、tf2 绑定、示例和教程  
- **C**（rclc）：不覆盖 rcl，而是补充 rcl，使其成为完整的 C 客户端库（详见 [micro.ros.org](https://micro.ros.org)）  
- **JVM 和 Android**：Java 和 Android 绑定  
- **.NET Core / UWP / C#**：包含绑定、代码生成器、示例等  
- **Node.js**（rclnodejs）：提供简洁的 JavaScript API  
- **Rust**：包含 rclrs 客户端库、代码生成器和示例  
- **Flutter 和 Dart**：Flutter 与 Dart 绑定  

**已停止维护的库**：
- C#（旧版）
- Objective C 和 iOS
- Zig

---

## 公共功能：rcl

客户端库中的大部分功能**与编程语言无关**（如参数行为、命名空间逻辑）。为避免重复实现，ROS 2 引入了**公共核心 ROS 客户端库**（rcl）：

- 实现所有**语言无关的 ROS 概念逻辑**
- 通过 **C 接口暴露**（因 C 最易被其他语言封装）
- 客户端库只需通过**外部函数接口**（FFI）包装 rcl 功能

**优势**：
- 减轻客户端库开发负担
- 确保不同语言间行为高度一致
- 核心逻辑变更（如命名空间规则）自动同步到所有客户端库
- 修复 bug 时只需在 rcl 中修改一次

**rcl API 文档**：[链接](https://docs.ros.org/en/rolling/p/rcl/)

---

## 语言特定功能

需要依赖语言特性的功能**不在 rcl 中实现**，而由各客户端库自行实现。例如：
- “spin” 函数使用的**线程模型**
- 内存管理策略
- 异步编程模型（如 async/await）

---

## 演示

如需了解 **rclpy 发布者** 与 **rclcpp 订阅者** 之间消息交换的详细过程，请观看 [ROSCon 演讲](https://vimeo.com/)（从 17:25 开始），[幻灯片在此](https://roscon.ros.org/)。

---

## 与 ROS 1 的对比

在 **ROS 1** 中，所有客户端库都是**从零开始独立开发**的：
- 优势：例如 Python 库可完全用 Python 实现，无需编译
- 劣势：
  - 命名规范和行为**不一致**
  - bug 修复需在多个库中重复进行
  - 许多功能仅在单一客户端库中实现（如 UDPROS）

---

## 总结

通过利用**公共核心 ROS 客户端库**（rcl），ROS 2 能够：
- 更轻松地开发多种语言的客户端库  
- 确保不同语言实现之间**行为高度一致**  
- 显著降低维护成本，提升系统整体可靠性