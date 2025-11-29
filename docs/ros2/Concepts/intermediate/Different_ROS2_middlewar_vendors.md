# 不同的 ROS 2 中间件供应商（Different ROS 2 Middleware Vendors）


## 概述

ROS 2 支持**多种中间件实现**，用于提供**发现**（discovery）、**序列化**（serialization）和**传输通信**（transportation）功能。  
这种灵活性源于一个基本理念：**中间件选择并非“一刀切”**（one size fits all）。

虽然 ROS 2 最初基于 **DDS/RTPS** 构建，但其生态系统已扩展至其他架构，例如 **Zenoh**。

---

## DDS 中间件

**DDS**（Data Distribution Service）是工业标准，由多家厂商实现，包括：

- **RTI Connext DDS**
- **eProsima Fast DDS**
- **Eclipse Cyclone DDS**
- **GurumNetworks GurumDDS**

DDS 使用 **RTPS**（Real-Time Publish-Subscribe，又称 DDSI-RTPS）作为其**网络层传输协议**。

### 为何选择 DDS？
- 提供**端到端中间件**功能  
- **分布式发现机制**（非 ROS 1 式的中心化）  
- 支持丰富的 **QoS**（Quality of Service，服务质量）策略，可精细控制通信行为

> 📚 详见：[DDS 在 ROS 2 中的应用动机](https://design.ros2.org/articles/ros_on_dds.html)

---

## Zenoh 中间件

**Zenoh** 是一种融合了**互联网级发布/订阅**与**分布式查询**的协议，专为从**服务器级硬件**到**资源受限边缘设备**的广泛场景设计。

### 核心特性
- **位置透明性**扩展至存储数据：查询无需关心数据物理位置  
- **轻量级**：相比 DDS 更小的运行时开销  
- **无冲突 QoS**：Zenoh 中基本不存在“不兼容”的 QoS 配置  
- **低线缆开销** + **灵活路由**：适用于**恶劣网络环境**

作为 ROS 2 的 RMW 实现，Zenoh 为 IoT 和边缘计算提供了高效替代方案。

---

## 支持的 RMW 实现

| 产品名称 | 许可证 | RMW 实现 | 状态 |
|--------|--------|---------|------|
| **eProsima Fast DDS** | Apache 2.0 | `rmw_fastrtps_cpp` | ✅ 完全支持，默认 RMW，二进制包内置 |
| **Eclipse Cyclone DDS** | EPL v2.0 | `rmw_cyclonedds_cpp` | ✅ 完全支持，二进制包内置 |
| **RTI Connext DDS** | 商业/科研 | `rmw_connextdds` | ✅ 完全支持（需单独安装 Connext） |
| **GurumNetworks GurumDDS** | 商业 | `rmw_gurumdds_cpp` | 👥 社区支持（需单独安装 GurumDDS） |
| **Eclipse Zenoh** | EPL v2.0 | `rmw_zenoh_cpp` | ✅ 完全支持（自 **Kilted Kaiju** 起内置） |

> 📘 实操指南：[《使用多个 RMW 实现》教程](https://docs.ros.org/en/humble/Tutorials/Advanced/Using-Different-Middleware.html)

---

## 选择中间件实现

选择时需权衡多种因素：

### 逻辑因素
- 许可证类型（开源 vs 商业）
- 平台支持（Linux、Windows、RTOS 等）

### 技术因素
- 资源占用（内存、CPU）
- 实时性与确定性
- 安全认证需求

### 厂商策略示例
- **RTI**：提供多个 Connext 变体（微控制器版、安全认证版等），但 ROS 2 仅支持标准桌面版
- **Eclipse**：
  - **Cyclone DDS**：轻量级 DDS，**优化实时确定性通信**
  - **Zenoh**：面向 **IoT/边缘计算**，强调高吞吐、低延迟、异构环境互操作性

> 💡 **关键机制**：通过 **RMW**（ROS Middleware）实现 ROS 2 与具体中间件的解耦，确保生态不被单一厂商锁定。

---

## 多 RMW 实现

### 二进制发行版
- 当前活跃 ROS 2 发行版默认包含多个 RMW（Fast DDS、Cyclone DDS、Connext、GurumDDS）
- **自 Kilted Kaiju 起**，Zenoh 也包含在内
- **默认 RMW**：`rmw_fastrtps_cpp`（无需额外安装）

### 源码构建
- 工作空间可**同时构建多个 RMW**
- 编译时自动检测已安装的中间件（如 Connext Pro），并构建对应 RMW 包

### 默认 RMW 选择规则
1. 若安装了 `rmw_fastrtps_cpp` → **优先使用**
2. 否则，按 **RMW 包名的字母顺序** 选择  
   - 例如：`rmw_connextdds` < `rmw_cyclonedds_cpp` → 选 Connext

> ⚙️ 使用指南：[指定 ROS 2 示例使用的 RMW](https://docs.ros.org/en/humble/Tutorials/Advanced/Specifying-RMW-Implementation.html)

---

## DDS 中间件间的跨厂商通信

- **多数情况下**，不同 DDS 实现的节点**可以互通**
- **但并非绝对保证**：DDS 标准允许厂商扩展，可能导致兼容性问题

> ⚠️ **建议**：在分布式系统中，**所有节点应使用相同的 ROS 2 版本和 RMW 实现**，以确保稳定通信。