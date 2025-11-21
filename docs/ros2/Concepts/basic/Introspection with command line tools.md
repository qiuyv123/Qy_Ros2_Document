# 使用命令行工具进行内省（Introspection with command line tools）

## 目录

- 用法（Usage）  
- 示例（Example）  
- ROS 2 守护进程：后台发现服务（ROS 2 Daemon: Background Discovery Service）  
- 实现（Implementation）  

---

## 用法（Usage）

ROS 2 提供了一套命令行工具，用于**内省**（introspect）ROS 2 系统。

这些工具的主要入口是 `ros2` 命令，它包含多个子命令，用于内省和操作节点、话题、服务等。

要查看所有可用的子命令，请运行：

```bash
ros2 --help
```

部分可用的子命令包括：

- `action`：内省/交互 ROS 动作  
- `bag`：录制/回放 rosbag  
- `component`：管理组件容器  
- `daemon`：内省/配置 ROS 2 守护进程  
- `doctor`：检查 ROS 环境中的潜在问题  
- `interface`：显示 ROS 接口信息  
- `launch`：运行/内省 launch 文件  
- `lifecycle`：内省/管理具有生命周期的节点  
- `multicast`：多播调试命令  
- `node`：内省 ROS 节点  
- `param`：内省/配置节点参数  
- `pkg`：内省 ROS 包  
- `plugin`：内省 ROS 插件  
- `run`：运行 ROS 节点  
- `security`：配置安全设置  
- `service`：内省/调用 ROS 服务  
- `test`：运行 ROS launch 测试  
- `topic`：内省/发布 ROS 话题  
- `trace`：用于获取 ROS 节点执行信息的追踪工具（仅 Linux 可用）  
- `wtf`：`doctor` 命令的别名  

---

## 示例（Example）

要使用命令行工具复现经典的 **talker-listener 示例**，可使用 `topic` 子命令在话题上发布和回显消息。

**在一个终端中发布消息**：
```bash
ros2 topic pub /chatter std_msgs/msg/String "data: Hello world"
```
输出：
```
publisher: beginning loop
publishing #1: std_msgs.msg.String(data='Hello world')
publishing #2: std_msgs.msg.String(data='Hello world')
...
```

**在另一个终端中回显接收到的消息**：
```bash
ros2 topic echo /chatter
```
输出：
```
data: Hello world
data: Hello world
...
```

---

## ROS 2 守护进程：后台发现服务（ROS 2 Daemon: Background Discovery Service）

ROS 2 使用**分布式发现机制**让节点相互连接。由于该机制**故意不采用中心化发现方式**，节点可能需要一定时间才能发现图中的所有参与者。

为加速查询响应（例如列出所有节点名称），ROS 2 会运行一个**后台守护进程**（daemon），持续维护 ROS 图的信息。

- 当您首次使用 `ros2 node list`、`ros2 topic list` 等内省命令时，**守护进程会自动启动**。
- 如果未运行守护进程，这些工具会在后台**自动创建一个新的守护进程**，然后再执行请求的命令。

守护进程通过 **localhost 网络接口**（127.0.0.1）通信，并使用 `ROS_DOMAIN_ID` 环境变量作为**端口偏移量**。

这意味着：
- 若想控制特定的守护进程实例（例如执行 `ros2 daemon stop`），**必须确保当前 `ROS_DOMAIN_ID` 与该守护进程启动时使用的值一致**。
- 不同的 `ROS_DOMAIN_ID` 会导致**多个独立的守护进程实例**运行在不同端口上。

您可运行以下命令查看与守护进程交互的更多选项：
```bash
ros2 daemon --help
```
包括启动、停止或检查守护进程状态的命令。

---

## 实现（Implementation）

`ros2` 命令的源代码位于：  
[https://github.com/ros2/ros2cli](https://github.com/ros2/ros2cli)

`ros2` 工具被设计为一个**可扩展的插件框架**。  
例如，当安装 `sros2` 包后，它会自动提供 `security` 子命令，`ros2` 工具会自动检测并加载该插件。