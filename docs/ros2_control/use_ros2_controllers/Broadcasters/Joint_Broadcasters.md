# Joint State Broadcaster（关节状态广播器）

该广播器读取所有**状态接口**（state interfaces），并将数据发布到以下两个话题：
- `/joint_states`
- `/dynamic_joint_states`

---

## 命令接口（Commands）

广播器**不是真正的控制器**，因此**不接收任何命令**。

---

## 硬件接口类型（Hardware Interface Type）

- **默认行为**：使用所有可用的关节状态接口。
- **自定义行为**：若配置了 `joints` 和 `interfaces` 参数，则接口集合由这两个参数的**笛卡尔积**（cross-product）决定。
- **容错机制**：
  - 若某些请求的接口缺失，控制器会**打印警告**，但仍会广播其他可用接口。
  - 若**所有请求的接口均未定义**，控制器在激活时将**返回错误**。

---

## 发布的话题（Published Topics）

### 1. `/joint_states`（`sensor_msgs/msg/JointState`）

- **仅发布运动相关接口**：`position`、`velocity`、`effort`
- 若某关节未提供某个接口，则该字段在消息中**留空**（对应位置设为 0 或省略）
- 专为兼容现有 ROS 工具（如 `robot_state_publisher`、RViz）设计

### 2. `/dynamic_joint_states`（`control_msgs/msg/DynamicJointState`）

- **发布所有可用状态接口**，包括：
  - 标准运动接口（位置、速度、力/力矩）
  - **自定义接口**（如温度、电压、扭矩传感器读数、校准标志等）
- 消息结构为**每个关节映射其接口名称与对应值**

#### 示例载荷：
```yaml
joint_names: [joint1, joint2]
interface_values:
  - interface_names: [position, velocity, effort]
    values: [1.5708, 0.0, 0.20]
  - interface_names: [position, temperature]
    values: [0.7854, 42.1]
```

> ✅ **使用建议**：若需访问**所有接口**（不仅限于运动量），请订阅此话题。

---

## 命名空间控制（注意）

- 若 `use_local_topics = true`：  
  话题发布在控制器命名空间下，如：  
  `/my_state_broadcaster/joint_states`  
  `/my_state_broadcaster/dynamic_joint_states`
- 若 `use_local_topics = false`（**默认**）：  
  话题发布在根命名空间下：  
  `/joint_states`  
  `/dynamic_joint_states`

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。完整说明见 `src/` 目录下的参数定义文件。

### 参数列表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `use_local_topics` | `bool` | `false` | 是否在控制器命名空间内发布话题 |
| `joints` | `string_array` | `{}` | 指定要广播的关节列表（需与 `interfaces` 联用） |
| `extra_joints` | `string_array` | `{}` | 额外添加的关节名，其状态值设为 0 |
| `interfaces` | `string_array` | `{}` | 指定要广播的接口类型（需与 `joints` 联用） |

> ⚠️ 若 `joints` 或 `interfaces` 任一为空，则广播**所有可用状态接口**。

---

### 接口映射（`map_interface_to_joint_state`）

**可选映射**，用于将**自定义接口名**映射到 `JointState` 消息的标准字段（`position`/`velocity`/`effort`）。

#### 典型用例：
- **液压机器人**：反馈值与命令值存在偏移，需同时可视化两者
- **多传感器融合**：同一关节有多个测量源（如编码器 + 视觉），值略有差异

#### 使用方式：
- 启动**多个** `joint_state_broadcaster` 实例，每个实例映射不同接口
- 配合多个 `robot_state_publisher`，在 RViz 中同时显示不同数据

#### 示例配置：
```yaml
map_interface_to_joint_state:
  position: kf_estimated_position

map_interface_to_joint_state:
  velocity: derived_velocity
  effort: derived_effort

map_interface_to_joint_state:
  effort: torque_sensor
```

#### 子参数：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `map_interface_to_joint_state.position` | `"position"` | 映射到 `JointState.position` 的接口名 |
| `map_interface_to_joint_state.velocity` | `"velocity"` | 映射到 `JointState.velocity` 的接口名 |
| `map_interface_to_joint_state.effort` | `"effort"` | 映射到 `JointState.effort` 的接口名 |

---

### 其他参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `frame_id` | `"base_link"` | 发布的 `JointState` 消息中使用的 `frame_id`，用于 RViz2 中力/力矩可视化 |

---

## 示例参数文件

```yaml
joint_state_broadcaster:
  ros__parameters:
    extra_joints: '{}'
    frame_id: base_link
    interfaces: '{}'
    joints: '{}'
    map_interface_to_joint_state:
      effort: effort
      position: position
      velocity: velocity
    use_local_topics: false
```