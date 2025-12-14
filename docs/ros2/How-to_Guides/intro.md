当然，以下是清除所有 Markdown 链接后的纯文本版本（保留标题结构和注释，仅移除超链接）：

---

# 操作指南 (How-to Guides)

操作指南针对“我该如何……？”这类问题提供直接的解决方案。它们以配方/食谱（recipe）为导向，并假设您对 ROS 2 的核心概念有一定的了解。

## 目录

### 安装与配置

  * 安装故障排除 (Installation troubleshooting)
  * 使用 ROS 1 桥接器 (Using the ROS 1 bridge)
      * 这篇指南主要针对从源码构建 ROS 1 桥接器。
  * 使用自定义生成的中间件接口 (Using custom middleware interfaces)
      * (注：实际页面可能包含关于 DDS 配置的更多细节)

### 开发应用程序

  * 使用 colcon 构建包 (Using colcon to build packages)
      * 关于如何使用 `colcon` 工具构建 ROS 2 工作空间的指南。
  * 使用 rosdep 管理依赖项 (Managing dependencies with rosdep)
  * 创建和使用插件 (Creating and using plugins)
      * (注：Humble 文档中可能有专门的 Pluginlib 教程)
  * 使用 Python 包 (Using Python packages)
      * 如何在 ROS 2 能够找到的地方安装和使用 Python 包。
  * 同步与异步服务客户端 (Synchronous vs. asynchronous service clients)
  * DDS 调优信息 (DDS tuning information)
  * 超额订阅：在一个进程中运行多个节点 (Over-subscription: Running multiple nodes in a single process)
      * (注：此处通常指组件/组合 Composition，或 Docker 相关指南)

### ROS 2 工具

  * 使用 ros2bag 记录和回放数据 (Recording and playing back data with ros2bag)
  * 使用 rqt 查看图和数据 (Visualizing graph and data with rqt)
  * 使用 ros2doctor 诊断问题 (Diagnosing issues with ros2doctor)
  * 启动文件 (Launch files)
      * 包括从 ROS 1 迁移 launch 文件以及使用 Python、XML 和 YAML 格式。
  * 参数 (Parameters)
      * 关于在 YAML 文件中传递参数以及参数覆盖的指南。

### RMW 实现 (ROS 中间件)

  * 使用多种 RMW 实现 (Working with multiple RMW implementations)
  * 使用 Eclipse Cyclone DDS (Working with Eclipse Cyclone DDS)
  * 使用 Fast DDS (Working with Fast DDS)
  * 使用 GurumNetworks GurumDDS (Working with GurumNetworks GurumDDS)
  * 使用 RTI Connext DDS (Working with RTI Connext DDS)

### 从 ROS 1 迁移

  * 从 ROS 1 迁移到 ROS 2 (Migrating from ROS 1 to ROS 2)

### 其他

  * 在 Docker 中运行 ROS 2 (Running ROS 2 in Docker)
  * 使用质量服务策略 (Using Quality of Service settings)
      * (链接到概念部分，但操作指南中通常包含具体的配置示例)

---

**译者注**：  
该页面主要是一个导航目录。如果您需要翻译列表中具体的某一篇指南（例如“安装故障排除”或“使用多种 RMW 实现”），请直接告诉我您想看哪一篇的详细翻译。