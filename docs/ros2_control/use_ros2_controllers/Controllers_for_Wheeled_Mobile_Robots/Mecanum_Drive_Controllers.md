# Mecanum Drive Controller（麦克纳姆轮驱动控制器）

`mecanum_drive_controller` 是一个面向**四麦克纳姆轮移动机器人**的控制器库。  
它实现了**通用的里程计计算**与**更新逻辑**，并定义了主要的接口。

---

## 控制器执行逻辑

- **输入**：带时间戳的 `Twist` 消息（`geometry_msgs/msg/TwistStamped`）
  - 使用分量：线速度 **x**、**y**，角速度 **z**
  - 其他分量被忽略
- **链式模式**（chained mode）下，提供三个**参考接口**（reference interfaces）
- **其他特性**：
  - 发布 `Odometry` 和 `TF` 消息
  - 基于参数的**输入命令超时机制**

> 💡 **关于里程计计算的说明**：  
> 与 `DiffDriveController` 不同，本控制器**不进行速度滤波**，而是返回原始值，允许用户按需进行后处理。  
> **原因**：滤波会引入延迟，使得行为曲线难以解释和对比。

---

## 控制器接口说明

### 参考接口（References，来自前置控制器）

当控制器处于 **链式模式**（`in_chained_mode == true`）时，向前置控制器暴露以下参考接口：

| 接口名称 | 单位 | 说明 |
|---------|------|------|
| `<controller_name>/linear/x/velocity` | m/s | X 方向线速度 |
| `<controller_name>/linear/y/velocity` | m/s | Y 方向线速度 |
| `<controller_name>/angular/z/velocity` | rad/s | Z 轴角速度 |

---

### 命令接口（Commands）

- `<*_wheel_command_joint_name>/velocity`（单位：rad/s）  
  向四个麦克纳姆轮关节发送角速度指令。

---

### 状态接口（States）

- `<joint_name>/velocity`（单位：rad/s）  
  读取轮子角速度。

> **注意**：  
> `joint_name` 优先使用 `*_wheel_state_joint_name` 参数（若指定），否则使用 `*_wheel_command_joint_name`。

---

## ROS 2 接口

### 订阅者（Subscribers）

**仅在非链式模式下使用**（`in_chained_mode == false`）：

- **`<controller_name>/reference`**（`geometry_msgs/msg/TwistStamped`）  
  当 `use_stamped_vel == true` 时使用。
- **`<controller_name>/reference_unstamped`**（`geometry_msgs/msg/Twist`）  
  当 `use_stamped_vel == false` 时使用。

---

### 发布者（Publishers）

- **`<controller_name>/odometry`**（`nav_msgs/msg/Odometry`）  
  机器人位姿与速度估计。
- **`<controller_name>/tf_odometry`**（`tf2_msgs/msg/TFMessage`）  
  里程计 TF 变换。
- **`<controller_name>/controller_state`**（`control_msgs/msg/MecanumDriveControllerState`）  
  控制器内部状态（如轮速、本体速度等）。

---

## 参数（Parameters）

该控制器使用 `generate_parameter_library` 管理参数。完整定义见 `src/` 目录下的参数文件。

### 超时与关节命名

| 参数 | 类型 | 默认值 | 说明 | 约束 |
|------|------|--------|------|------|
| `reference_timeout` | `double` | `0.0` | 参考命令超时时间（秒）。超时后重置参考值。设为 `0` 表示每次运行后重置。 | — |
| `front_left_wheel_command_joint_name` | `string` | `""` | 前左轮命令关节名 | **非空，只读** |
| `front_right_wheel_command_joint_name` | `string` | `""` | 前右轮命令关节名 | **非空，只读** |
| `rear_right_wheel_command_joint_name` | `string` | `""` | 后右轮命令关节名 | **非空，只读** |
| `rear_left_wheel_command_joint_name` | `string` | `""` | 后左轮命令关节名 | **非空，只读** |

### 状态关节名（可选）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `front_left_wheel_state_joint_name` | `""` | 前左轮状态关节名（当状态与命令关节不同时使用） |
| `front_right_wheel_state_joint_name` | `""` | 前右轮状态关节名 |
| `rear_right_wheel_state_joint_name` | `""` | 后右轮状态关节名 |
| `rear_left_wheel_state_joint_name` | `""` | 后左轮状态关节名 |

> ✅ **只读参数**

---

### 运动学参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `kinematics.base_frame_offset.x` | `0.0` | 机器人基座在 `base_link` 坐标系中的 X 偏移 |
| `kinematics.base_frame_offset.y` | `0.0` | Y 偏移 |
| `kinematics.base_frame_offset.theta` | `0.0` | 旋转偏移（弧度） |
| `kinematics.wheels_radius` | `0.0` | 轮子半径（米） | **> 0.0** |
| `kinematics.sum_of_robot_center_projection_on_X_Y_axis` | `0.0` | 麦克纳姆轮逆运动学几何参数：<br>`lx + ly`（机器人中心到轮子在 X/Y 轴投影距离之和） |

---

### 其他参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `use_stamped_vel` | `true` | 是否使用带时间戳的速度消息计算命令时效性 |
| `base_frame_id` | `"base_link"` | 机器人基座坐标系名称 |
| `odom_frame_id` | `"odom"` | 里程计坐标系名称 |
| `enable_odom_tf` | `true` | 是否发布里程计 TF |
| `twist_covariance_diagonal` | `{0.1, 0.1, 0.1, 0.1, 0.1, 0.1}` | 速度协方差对角线（6维） |
| `pose_covariance_diagonal` | `{0.1, 0.1, 0.1, 0.1, 0.1, 0.1}` | 位姿协方差对角线（6维） |

---

## 示例参数文件

### 基础配置
```yaml
test_mecanum_drive_controller:
  ros__parameters:
    reference_timeout: 0.9

    front_left_wheel_command_joint_name: "front_left_wheel_joint"
    front_right_wheel_command_joint_name: "front_right_wheel_joint"
    rear_right_wheel_command_joint_name: "back_right_wheel_joint"
    rear_left_wheel_command_joint_name: "back_left_wheel_joint"

    kinematics:
      base_frame_offset: { x: 0.0, y: 0.0, theta: 0.0 }
      wheels_radius: 0.5
      sum_of_robot_center_projection_on_X_Y_axis: 1.0

    base_frame_id: "base_link"
    odom_frame_id: "odom"
    enable_odom_tf: true
    twist_covariance_diagonal: [0.0, 7.0, 14.0, 21.0, 28.0, 35.0]
    pose_covariance_diagonal: [0.0, 6.0, 12.0, 18.0, 24.0, 30.0]
```

### 带旋转偏移的配置
```yaml
test_mecanum_drive_controller_with_rotation:
  ros__parameters:
    reference_timeout: 5.0

    front_left_wheel_command_joint_name: "front_left_wheel_joint"
    front_right_wheel_command_joint_name: "front_right_wheel_joint"
    rear_right_wheel_command_joint_name: "rear_right_wheel_joint"
    rear_left_wheel_command_joint_name: "rear_left_wheel_joint"

    kinematics:
      base_frame_offset: { x: 1.0, y: 2.0, theta: 3.0 }
      wheels_radius: 0.05
      sum_of_robot_center_projection_on_X_Y_axis: 0.2925

    base_frame_id: "base_link"
    odom_frame_id: "odom"
    enable_odom_tf: true
    pose_covariance_diagonal: [0.001, 0.001, 0.001, 0.001, 0.001, 0.001]
    twist_covariance_diagonal: [0.001, 0.001, 0.001, 0.001, 0.001, 0.001]
```

> 📁 示例文件位于 `test/` 目录下。