# 参数（Parameters）

## 目录

- 概述  
- 参数背景  
- 声明参数  
- 参数类型  
- 参数回调  
- 与参数交互  
- 运行节点时设置初始参数值  
- 启动节点时设置初始参数值  
- 在运行时操作参数值  
- 从 ROS 1 迁移  

---

## 概述

ROS 2 中的参数与**单个节点**关联。参数用于在**启动时**（以及运行时）配置节点，而无需修改代码。参数的生命周期与其所属节点的生命周期绑定（尽管节点可以实现某种持久化机制，在重启后重新加载参数值）。

参数通过**节点名称**、**节点命名空间**、**参数名称**和**参数命名空间**进行寻址。参数命名空间是可选的。

每个参数包含一个**键**（key）、一个**值**（value）和一个**描述符**（descriptor）。  
- 键为字符串  
- 值的类型为以下之一：`bool`、`int64`、`float64`、`string`、`byte[]`、`bool[]`、`int64[]`、`float64[]` 或 `string[]`  
- 描述符默认为空，但可包含参数说明、取值范围、类型信息及其他约束

如需动手实践 ROS 参数，请参阅：[Understanding parameters](https://docs.ros.org/en/humble/Tutorials/Parameters/Understanding-Parameters.html)。

---

## 参数背景

（原文无内容，仅标题）

---

## 声明参数

默认情况下，节点必须**在其生命周期内声明所有将要接受的参数**。  
这是为了在节点启动时就明确定义参数的类型和名称，从而降低后续配置错误的风险。

有关如何在节点中声明和使用参数的教程，请参阅：  
- [在类中使用参数（C++）](https://docs.ros.org/en/humble/Tutorials/Parameters/Using-Parameters-In-A-Class-CPP.html)  
- [在类中使用参数（Python）](https://docs.ros.org/en/humble/Tutorials/Parameters/Using-Parameters-In-A-Class-Python.html)

对于某些类型的节点，并非所有参数都能提前获知。此时可在实例化节点时将 `allow_undeclared_parameters` 设为 `true`，从而允许获取和设置未声明的参数。

---

## 参数类型

ROS 2 节点上的每个参数都属于概述中提到的预定义类型之一。  
默认情况下，**在运行时尝试更改已声明参数的类型会失败**，以防止常见错误（例如将布尔值赋给整型参数）。

如果某个参数需要支持多种类型，且使用该参数的代码能够处理类型变化，则可更改此默认行为。  
在声明参数时，应使用 `ParameterDescriptor` 并将 `dynamic_typing` 成员变量设为 `true`。

---

## 参数回调

ROS 2 节点可注册三种不同类型的回调函数，以响应参数变化。这三种回调均为可选。

1. **“预设置参数”回调**（pre set parameter callback）  
   通过节点 API 的 `add_pre_set_parameters_callback` 注册。  
   回调接收一个待修改的 `Parameter` 对象列表，不返回值。  
   调用时可修改该列表，以更改、添加或删除条目。  
   例如：当 `parameter1` 变化时自动更新 `parameter2`，可通过此回调实现。

2. **“设置参数”回调**（set parameter callback）  
   通过 `add_on_set_parameters_callback` 注册。  
   回调接收一个**不可变**的 `Parameter` 对象列表，返回 `rcl_interfaces/msg/SetParametersResult`。  
   主要用途：**检查即将发生的参数更改并明确拒绝无效变更**。

   > **注意**：  
   > “设置参数”回调**不应产生副作用**。  
   > 由于多个此类回调可串联调用，单个回调无法得知后续回调是否会拒绝更新。  
   > 若回调提前修改了内部状态，可能导致状态与实际参数值不一致。  
   > 如需在参数成功更改后执行操作，请使用下一种回调。

3. **“后设置参数”回调**（post set parameter callback）  
   通过 `add_post_set_parameters_callback` 注册。  
   回调接收一个不可变的 `Parameter` 对象列表，不返回值。  
   主要用途：**对已成功接受的参数变更做出响应**。

ROS 2 示例包中提供了上述所有回调的使用示例。

---

## 与参数交互

ROS 2 节点可通过节点 API 执行参数操作（参见 C++/Python 教程）。  
外部进程则可通过节点实例化时**默认创建的参数服务**进行操作。这些服务包括：

- `/node_name/describe_parameters`  
  服务类型：`rcl_interfaces/srv/DescribeParameters`  
  输入参数名列表，返回对应的描述符列表。

- `/node_name/get_parameter_types`  
  服务类型：`rcl_interfaces/srv/GetParameterTypes`  
  输入参数名列表，返回对应的参数类型列表。

- `/node_name/get_parameters`  
  服务类型：`rcl_interfaces/srv/GetParameters`  
  输入参数名列表，返回对应的参数值列表。

- `/node_name/list_parameters`  
  服务类型：`rcl_interfaces/srv/ListParameters`  
  输入可选的参数前缀列表，返回匹配的参数名；若前缀为空，则返回所有参数。

- `/node_name/set_parameters`  
  服务类型：`rcl_interfaces/srv/SetParameters`  
  输入参数名和值列表，尝试设置参数。返回每个参数的设置结果（部分成功、部分失败）。

- `/node_name/set_parameters_atomically`  
  服务类型：`rcl_interfaces/srv/SetParametersAtomically`  
  输入参数名和值列表，尝试设置参数。返回**单一结果**：任一参数失败则全部失败。

---

## 运行节点时设置初始参数值

运行节点时，可通过**命令行参数**或**YAML 文件**设置初始参数值。  
示例请参阅：[从命令行直接设置参数](https://docs.ros.org/en/humble/Tutorials/Parameters/Set-parameters-at-startup.html)。

---

## 启动节点时设置初始参数值

也可通过 ROS 2 的 **launch 机制**在启动节点时设置初始参数值。  
详情请参阅：[通过 launch 指定参数](https://docs.ros.org/en/humble/Tutorials/Launching/Creating-Launch-Files.html)。

---

## 在运行时操作参数值

`ros2 param` 命令是与**已运行节点**的参数交互的通用方式。  
它使用上述参数服务 API 执行各种操作。  
使用详情请参阅：[`ros2 param` 使用指南](https://docs.ros.org/en/humble/Tutorials/Parameters/Using-ros2-param-command-line-tool.html)。

---

## 从 ROS 1 迁移

- [Launch 文件迁移指南](https://docs.ros.org/en/humble/How-To-Guides/Launch-files-migration-guide.html) 说明了如何将 ROS 1 中的 `<param>` 和 `<rosparam>` 标签迁移到 ROS 2。
- [迁移指南](https://docs.ros.org/en/humble/How-To-Guides/Parameter-Migration-Guide.html) 说明了如何将 ROS 1 的参数机制迁移到 ROS 2。