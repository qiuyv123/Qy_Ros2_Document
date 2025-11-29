# forward_command_controller（前馈命令控制器）

`forward_command_controller` 是一个实现**前馈控制器**（feedforward controller）的**基类**。  
其具体实现可在以下软件包中找到：

- `position_controllers`
- `velocity_controllers`
- `effort_controllers`

---

## 硬件接口类型（Hardware interface type）

该控制器**支持任意类型的命令接口**（如 position、velocity、effort 等）。

---

## ROS 2 接口

### 话题（Topics）

- **`~/commands`**（输入）  
  - 类型：`std_msgs/msg/Float64MultiArray`  
  - 用途：目标关节命令数组

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。

> ⚠️ 注意：存在两个相关控制器：
> - `forward_command_controller`
> - `multi_interface_forward_command_controller`

### 公共参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `joints` | `string_array` | `{}` | 要控制的关节名称列表 |
| `interface_name` | `string` | `""` | 要命令的接口名称（如 `"position"`、`"velocity"` 等） |