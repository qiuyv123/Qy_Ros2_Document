# Joint Trajectory Controller（关节轨迹控制器）

用于在**一组关节上执行关节空间轨迹**的控制器。  
控制器会对轨迹点进行**时间插值**，因此点与点之间的距离可以任意。即使轨迹仅包含一个点，控制器也能接受。

轨迹由一系列**路径点**（waypoints）组成，每个路径点包含：
- **位置**（必选）
- **速度**（可选）
- **加速度**（可选）

这些路径点需在指定时间到达，控制器会尽其所能地执行该轨迹。

> 📝 本部分内容部分源自 ROS 1 Wiki（CC BY 3.0 许可），已根据 ROS 2 实现进行适配。  
> 引用来源：Adolfo Rodriguez: `joint_trajectory_controller` [^1]

---

## 硬件接口类型（Hardware Interface Types）

### 支持的命令接口组合

控制器支持以下**命令接口**（command interfaces）组合：

- `position`
- `position`, `velocity`
- `position`, `velocity`, `acceleration`
- `velocity`
- `effort`

### 控制律说明

- **`position`**：目标位置直接转发至硬件
- **`acceleration`**：目标加速度直接转发至硬件
- **`velocity` / `effort`**：通过 **PID 控制器** 将 **位置+速度跟踪误差** 映射为速度或力命令（需配置 PID 参数）

### 合法的状态接口组合

| 命令接口 | 所需状态接口 |
|---------|-------------|
| `position` | 无限制 |
| `velocity`（单独使用） | 必须包含 `position`, `velocity` |
| `effort` | 必须包含 `position`, `velocity` |
| `acceleration` | 必须包含 `position`, `velocity` |

### 状态接口依赖关系

- 若**无 `position`**，则**不能使用 `velocity`** 状态接口
- 若**无 `position` 和 `velocity`**，则**不能使用 `acceleration`** 状态接口

---

## 其他特性

- **实时安全实现**（Realtime-safe）
- **正确处理连续关节**（如旋转无限的关节）
- **对系统时钟跳变鲁棒**：即使系统时间发生不连续变化，已排队的轨迹段仍能平滑执行

---

## 使用 Joint Trajectory Controller

- **至少需要位置反馈**（position feedback）
- 速度和加速度反馈为可选
- **控制器不会内部积分**：若硬件仅提供加速度或速度状态，必须在**硬件接口层**完成积分（加速度 → 速度 → 位置）

### 通用控制器配置示例

```yaml
controller_manager:
  ros__parameters:
    joint_trajectory_controller:
      type: "joint_trajectory_controller/JointTrajectoryController"

joint_trajectory_controller:
  ros__parameters:
    joints:
      - joint1
      - joint2
      - joint3
      - joint4
      - joint5
      - joint6

    command_interfaces:
      - position

    state_interfaces:
      - position
      - velocity

    state_publish_rate: 50.0
    action_monitor_rate: 20.0

    allow_partial_joints_goal: false
    open_loop_control: true
    constraints:
      stopped_velocity_tolerance: 0.01
      goal_time: 0.0
      joint1:
        trajectory: 0.05
        goal: 0.03
```

---

## 抢占策略（Preemption Policy）

- **同一时间仅允许一个动作目标激活**（或无目标，若使用话题接口）
- 路径与目标容差**仅对当前激活目标**的轨迹段进行检查
- **动作接口抢占**：新目标到达时，当前目标**被取消**，客户端收到通知，轨迹按[轨迹替换](#trajectory-replacement)规则替换
- **话题接口发送空轨迹**：会**覆盖当前动作目标**，但**不会中止动作**

> ⚠️ **注意**：该控制器**尚未实现为链式控制器**（chainable controller）。

---

## 控制器接口说明

### 状态接口（States）
格式：`<joint>/<state_interface>`  
合法组合见上文「硬件接口类型」部分。

### 命令接口（Commands）
支持两种轨迹输入方式：
1. **动作接口**（推荐，支持执行监控）
2. **话题接口**（fire-and-forget，无反馈）

> ✅ 两种方式均使用 `trajectory_msgs/msg/JointTrajectory` 消息格式  
> ❗ 若 `allow_partial_joints_goal=false`，则必须为**所有控制器关节**提供值

---

## 动作接口（Actions）

- **`<controller_name>/follow_joint_trajectory`**  
  - 类型：`control_msgs/action/FollowJointTrajectory`
  - **主要轨迹输入方式**，支持执行监控

#### 容差（Tolerances）
- 可为每个关节指定 **路径容差** 和 **目标容差**
- 容差值含义：
  - `0`：使用默认容差
  - `-1`：**无限制**（忽略容差）
- 若未指定容差，则使用参数中定义的默认值
- **容差超限** → 动作中止，客户端通知，保持当前位置
- **目标达成**（在容差内）→ 动作返回成功，保持最后指令点

---

## 订阅者（Subscriber）

- **`<controller_name>/joint_trajectory`**  
  - 类型：`trajectory_msgs/msg/JointTrajectory`
  - **无监控的轻量级接口**
  - **不使用目标容差**（无通知机制）
  - 容差超限时轨迹中止，保持当前位置
  - 可通过 `~/query_state` 服务或 `~/state` 话题获取状态（但不如动作接口便捷）

---

## 发布者（Publishers）

- **`<controller_name>/controller_state`**  
  - 类型：`control_msgs/msg/JointTrajectoryControllerState`
  - 以 Controller Manager 的更新频率发布控制器内部状态

---

## 服务（Services）

- **`<controller_name>/query_state`**  
  - 类型：`control_msgs/srv/QueryTrajectoryState`
  - 可查询**任意未来时刻**的控制器状态

---

## 进一步信息

- [轨迹表示](https://control.ros.org/humble/doc/ros2_controllers/joint_trajectory_controller/doc/trajectory.html)（Trajectory Representation）
- [轨迹替换](https://control.ros.org/humble/doc/ros2_controllers/joint_trajectory_controller/doc/trajectory.html#trajectory-replacement)（Trajectory Replacement）
- [Joint Trajectory Controller 参数详解](https://control.ros.org/humble/doc/ros2_controllers/joint_trajectory_controller/doc/parameters.html)
- [`rqt_joint_trajectory_controller`](https://control.ros.org/humble/doc/ros2_controllers/rqt_joint_trajectory_controller/doc/userdoc.html)（GUI 调试工具）

[^1]: Adolfo Rodriguez: **[joint_trajectory_controller](http://wiki.ros.org/joint_trajectory_controller)**