# Admittance Controller（导纳控制器）

导纳控制器使您能够**基于末端执行器**（TCP），实现**零力控制**（zero-force control）。

该控制器实现了 `ChainedControllerInterface`，因此**可被其他控制器链式调用**（如 `JointTrajectoryController`）。

> ⚠️ **依赖项**：控制器需要**外部运动学插件**才能运行。  
> `kinematics_interface` 仓库提供了该控制器可使用的接口与实现。

---

## ROS 2 接口

### 话题（Topics）

- **`~/joint_references`**（输入）  
  - 类型：`trajectory_msgs/msg/JointTrajectoryPoint`  
  - 用途：**非链式模式**下的目标关节命令

- **`~/state`**（输出）  
  - 类型：`control_msgs/msg/AdmittanceControllerState`  
  - 用途：发布控制器内部状态

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。完整说明见 `src/` 目录下的参数定义文件。

### 关节与接口配置

| 参数 | 类型 | 默认值 | 说明 | 只读 |
|------|------|--------|------|------|
| `joints` | `string_array` | — | 控制器使用的关节列表 | ✅ |
| `command_joints` | `string_array` | `{}` | 用于向**下游控制器**写入参考值的关节（链式模式专用） | ✅ |
| `command_interfaces` | `string_array` | — | 控制器申请的命令接口（如 `position`、`velocity`） | ✅ |
| `state_interfaces` | `string_array` | — | 控制器申请的状态接口 | ✅ |
| `chainable_command_interfaces` | `string_array` | `{"position", "velocity"}` | 控制器导出的**参考接口**（供上游控制器使用） | ✅ |

### 运动学插件

| 参数 | 说明 |
|------|------|
| `kinematics.plugin_name` | 插件类名（如 `kinematics_interface_kdl/KinematicsInterfaceKDL`） |
| `kinematics.plugin_package` | 插件所在包名 |
| `kinematics.base` | 机器人基座连杆（用于运动学计算） |
| `kinematics.tip` | 末端执行器连杆 |
| `kinematics.alpha` | 雅可比伪逆的阻尼系数（默认：`0.01`） |

### 力/力矩传感器

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `ft_sensor.name` | — | URDF 中 F/T 传感器名称 |
| `ft_sensor.frame.id` | — | F/T 传感器所在坐标系 |
| `ft_sensor.filter_coefficient` | `0.05` | 传感器指数滤波系数 |

### 控制与坐标系

| 参数 | 说明 |
|------|------|
| `control.frame.id` | 导纳计算所用的控制坐标系（通常为 `tool0`） |
| `fixed_world_frame.frame.id` | 世界坐标系（重力必须沿 **-Z 方向**） |
| `gravity_compensation.frame.id` | 重心（CoG）定义坐标系（通常为 F/T 传感器坐标系） |

### 重力补偿

| 参数 | 约束 | 说明 |
|------|------|------|
| `gravity_compensation.CoG.pos` | 长度 = 3 | CoG 在补偿坐标系中的位置 `[x, y, z]` |
| `gravity_compensation.CoG.force` | ≥ 0.0 | 末端执行器重量（`mass × 9.81`，默认 `0.0`） |

### 导纳参数（6 自由度）

| 参数 | 约束 | 说明 |
|------|------|------|
| `admittance.selected_axes` | 长度 = 6 | 是否启用各轴（`[x, y, z, rx, ry, rz]`） |
| `admittance.mass` | 长度 = 6<br>每个 ≥ 0.0001 | 各轴质量 |
| `admittance.damping_ratio` | 长度 = 6 | 各轴阻尼比（\( \zeta = D / (2 \sqrt{M S}) \)） |
| `admittance.stiffness` | 长度 = 6<br>每个 ≥ 0.0 | 各轴刚度 |
| `admittance.joint_damping` | ≥ 0.0 | 关节阻尼（默认 `5.0`） |

### 其他

| 参数 | 默认值 | 说明 | 只读 |
|------|--------|------|------|
| `robot_description` | — | URDF 格式的机器人描述（用于运动学） | ✅ |
| `enable_parameter_update_without_reactivation` | `true` | 是否允许**运行时动态更新参数** | — |

---

## 示例参数文件

### 最小配置（仅加载）
```yaml
load_admittance_controller:
  ros__parameters:
    joints: [joint1, joint2]
    command_interfaces: [velocity]
    state_interfaces: [position, velocity]
    chainable_command_interfaces: [position, velocity]
```

### 完整配置（KUKA KR6 示例）
```yaml
test_admittance_controller:
  ros__parameters:
    joints: [joint1, joint2, joint3, joint4, joint5, joint6]
    command_interfaces: [position]
    state_interfaces: [position]
    chainable_command_interfaces: [position, velocity]

    kinematics:
      plugin_name: kinematics_interface_kdl/KinematicsInterfaceKDL
      plugin_package: kinematics_interface
      base: base_link
      tip: tool0
      alpha: 0.0005

    ft_sensor:
      name: ft_sensor_name
      frame:
        id: link_6
        external: false
      filter_coefficient: 0.005

    control:
      frame:
        id: tool0
        external: false

    fixed_world_frame:
      frame:
        id: base_link
        external: false

    gravity_compensation:
      frame:
        id: tool0
        external: false
      CoG:
        pos: [0.1, 0.0, 0.0]  # x, y, z
        force: 23.0            # 重量

    admittance:
      selected_axes: [true, true, true, true, true, true]
      mass: [5.5, 6.6, 7.7, 8.8, 9.9, 10.10]
      damping_ratio: [2.828427, 2.828427, 2.828427, 2.828427, 2.828427, 2.828427]
      stiffness: [214.1, 214.2, 214.3, 214.4, 214.5, 214.6]
```

> 📁 示例位于 `test/` 目录。

---

## ros2_control 接口

### 参考接口（References，供上游控制器使用）
```
<controller_name>/<joint_name>/position
<controller_name>/<joint_name>/velocity
```

### 状态接口（States）
- 格式：`<joint>/<state_interface>`
- 支持：`position`、`velocity`、`acceleration`
- 若某状态接口未提供，则使用**上一周期的命令值**进行计算

### 力/力矩接口（通过语义组件）
- 基于 `ft_sensor.name` 构建：
  ```
  <sensor_name>/force.x
  <sensor_name>/force.y
  <sensor_name>/force.z
  <sensor_name>/torque.x
  <sensor_name>/torque.y
  <sensor_name>/torque.z
  ```

### 命令接口（Commands）
- 格式：`<joint>/<command_interface>`
- 支持：`position`、`velocity`、`acceleration`（定义见 `hardware_interface_type_values.hpp`）