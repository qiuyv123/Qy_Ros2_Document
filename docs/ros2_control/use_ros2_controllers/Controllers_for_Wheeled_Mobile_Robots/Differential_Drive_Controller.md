# Diff Drive Controller（差速驱动控制器）

`diff_drive_controller` 是用于**差速驱动移动机器人**的控制器。

- **输入**：机器人本体的速度指令（线速度和角速度）  
- **输出**：转换为左右轮的驱动指令  
- **里程计**：基于硬件反馈计算并发布

> 📚 有关移动机器人运动学基础及本文使用的术语，请参阅：[轮式移动机器人运动学（Wheeled Mobile Robot Kinematics）](./mobile_robot_kinematics.html)

---

## 其他特性

- **实时安全实现**（Realtime-safe implementation）  
- **里程计发布**（Odometry publishing）  
- **任务空间的速度、加速度和加加速度**（jerk）  
- **命令超时自动停止**（Automatic stop after command time-out）

> ⚠️ **注意**：该控制器**尚未实现为链式控制器**（chainable controller）。

---

## 控制器接口说明

### 反馈接口（Feedback）

- 使用关节的 **位置**（`hardware_interface::HW_IF_POSITION`）
- 或 **速度**（`hardware_interface::HW_IF_VELOCITY`），当参数 `position_feedback=false` 时

### 输出接口（Output）

- 使用关节的 **速度指令**（`hardware_interface::HW_IF_VELOCITY`）

---

## ROS 2 接口

### 订阅者（Subscribers）

- **`~/cmd_vel`**（`geometry_msgs/msg/TwistStamped`）  
  当 `use_stamped_vel=true` 时使用。  
  控制器提取：  
  - 线速度的 **x 分量**（前进/后退）  
  - 角速度的 **z 分量**（偏航旋转）  
  其他分量被忽略。

- **`~/cmd_vel_unstamped`**（`geometry_msgs/msg/Twist`）  
  当 `use_stamped_vel=false` 时使用。  
  提取逻辑同上。

### 发布者（Publishers）

- **`~/odom`**（`nav_msgs/msg/Odometry`）  
  机器人在自由空间中的位姿和速度估计。

- **`/tf`**（`tf2_msgs/msg/TFMessage`）  
  TF 变换树。**仅当 `enable_odom_tf=true` 时发布**。

- **`~/cmd_vel_out`**（`geometry_msgs/msg/TwistStamped`）  
  经限幅后的速度指令。**仅当 `publish_limited_velocity=true` 时发布**。

---

## 参数（Parameters）

该控制器使用 `generate_parameter_library` 管理参数。完整参数定义见 `src/` 目录下的参数定义文件。

### 基础参数

| 参数 | 类型 | 默认值 | 说明 | 约束 |
|------|------|--------|------|------|
| `left_wheel_names` | `string_array` | `{}` | 左侧轮关节名称 | **不能为空** |
| `right_wheel_names` | `string_array` | `{}` | 右侧轮关节名称 | **不能为空** |
| `wheel_separation` | `double` | `0.0` | 左右轮中心距。若错误，转弯行为将异常 | > 0.0 |
| `wheels_per_side` | `int` | `0` | 每侧轮子数量。用于多轮共控场景（如 Husky 每侧 2 轮但共用 1 个控制信号，应设为 `1`） | — |
| `wheel_radius` | `double` | `0.0` | 轮子半径。若错误，移动速度将偏离预期 | > 0.0 |
| `wheel_separation_multiplier` | `double` | `1.0` | 轮距修正系数 | — |
| `left_wheel_radius_multiplier` | `double` | `1.0` | 左轮半径修正系数 | — |
| `right_wheel_radius_multiplier` | `double` | `1.0` | 右轮半径修正系数 | — |

### TF 与坐标系

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `tf_frame_prefix_enable` | `true` | 是否启用 TF 前缀 |
| `tf_frame_prefix` | `""` | TF 帧前缀。若为空，则使用控制器命名空间 |
| `odom_frame_id` | `"odom"` | 里程计坐标系名称（父帧） |
| `base_frame_id` | `"base_link"` | 机器人基座坐标系名称（子帧） |

### 里程计协方差

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `pose_covariance_diagonal` | `{0,0,0,0,0,0}` | 位姿协方差对角线（6维）<br>建议起点：`[0.001, 0.001, 0.001, 0.001, 0.001, 0.01]` |
| `twist_covariance_diagonal` | `{0,0,0,0,0,0}` | 速度协方差对角线（6维）<br>建议起点同上 |

### 控制与反馈模式

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `open_loop` | `false` | 若为 `true`，里程计基于**命令值**而非反馈计算 |
| `position_feedback` | `true` | 硬件是否提供位置反馈（否则使用速度反馈） |
| `enable_odom_tf` | `true` | 是否发布 `odom` → `base_link` 的 TF 变换 |
| `cmd_vel_timeout` | `0.5` | 命令超时时间（秒），超时后自动停止 |
| `publish_limited_velocity` | `false` | 是否发布限幅后的速度 |
| `velocity_rolling_window_size` | `10` | 里程计中计算平均速度的滑动窗口大小 |
| `use_stamped_vel` | `true` | 是否使用带时间戳的速度消息计算命令时效性 |
| `publish_rate` | `50.0` | 里程计和 TF 的发布频率（Hz） |

### 速度限幅（任务空间）

#### 线速度（`linear.x`）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `has_velocity_limits` | `false` | 是否启用速度限制 |
| `has_acceleration_limits` | `false` | 是否启用加速度限制 |
| `has_jerk_limits` | `false` | 是否启用加加速度（jerk）限制 |
| `max_velocity` | `NaN` | 最大线速度 |
| `min_velocity` | `NaN` | 最小线速度 |
| `max_acceleration` | `NaN` | 最大加速度 |
| `min_acceleration` | `NaN` | 最小加速度 |
| `max_jerk` | `NaN` | 最大加加速度 |
| `min_jerk` | `NaN` | 最小加加速度 |

> ⚠️ 位置限制被忽略。

#### 角速度（`angular.z`）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `has_velocity_limits` | `false` | 是否启用角速度限制 |
| `has_acceleration_limits` | `false` | 是否启用角加速度限制 |
| `has_jerk_limits` | `false` | 是否启用角加加速度限制 |
| `max_velocity` | `NaN` | 最大角速度 |
| `min_velocity` | `NaN` | 最小角速度 |
| `max_acceleration` | `NaN` | 最大角加速度 |
| `min_acceleration` | `NaN` | 最小角加速度 |
| `max_jerk` | `NaN` | 最大角加加速度 |
| `min_jerk` | `NaN` | 最小角加加速度 |

> ⚠️ 位置限制被忽略。

---

## 示例参数文件

```yaml
test_diff_drive_controller:
  ros__parameters:
    left_wheel_names: ["left_wheels"]
    right_wheel_names: ["right_wheels"]

    wheel_separation: 0.40
    wheels_per_side: 1  # 实际有 2 个轮子，但共用 1 个控制信号
    wheel_radius: 0.02

    wheel_separation_multiplier: 1.0
    left_wheel_radius_multiplier: 1.0
    right_wheel_radius_multiplier: 1.0

    odom_frame_id: odom
    base_frame_id: base_link
    pose_covariance_diagonal: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
    twist_covariance_diagonal: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]

    position_feedback: false
    open_loop: true
    enable_odom_tf: true

    cmd_vel_timeout: 0.5  # 秒
    publish_limited_velocity: true
    velocity_rolling_window_size: 10

    linear.x.has_velocity_limits: false
    linear.x.has_acceleration_limits: false
    linear.x.has_jerk_limits: false
    linear.x.max_velocity: 0.0
    linear.x.min_velocity: 0.0
    linear.x.max_acceleration: 0.0
    linear.x.max_jerk: 0.0
    linear.x.min_jerk: 0.0

    angular.z.has_velocity_limits: false
    angular.z.has_acceleration_limits: false
    angular.z.has_jerk_limits: false
    angular.z.max_velocity: 0.0
    angular.z.min_velocity: 0.0
    angular.z.max_acceleration: 0.0
    angular.z.min_acceleration: 0.0
    angular.z.max_jerk: 0.0
    angular.z.min_jerk: 0.0
```

