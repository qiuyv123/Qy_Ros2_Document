# Gripper Action Controller（夹爪动作控制器）

该控制器用于执行**简单单自由度夹爪**（single-DOF gripper）的 `GripperCommand` 动作，提供两种实现：

- `position_controllers/GripperActionController`  
- `effort_controllers/GripperActionController`

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。完整参数说明见 `src/` 目录下的参数定义文件。

### 参数列表

| 参数 | 类型 | 默认值 | 说明 | 约束 |
|------|------|--------|------|------|
| `action_monitor_rate` | `double` | `20.0` | 动作监控频率（Hz） | ≥ 0.1 |
| `joint` | `string` | `""` | 夹爪关节名称 | — |
| `goal_tolerance` | `double` | `0.01` | 目标位置容差（单位：弧度或米） | ≥ 0.0 |
| `max_effort` | `double` | `0.0` | 允许的最大力/力矩 | ≥ 0.0 |
| `allow_stalling` | `bool` | `false` | 若启用，当夹爪在运动中**卡住**（stall），动作服务器仍返回 **成功** | — |
| `stall_velocity_threshold` | `double` | `0.001` | 判定卡住的**速度阈值**（低于此值视为停滞） | — |
| `stall_timeout` | `double` | `1.0` | 卡住判定的**超时时间**（秒） | — |

---

## 示例参数文件

```yaml
gripper_action_controller:
  ros__parameters:
    action_monitor_rate: 20.0
    allow_stalling: false
    goal_tolerance: 0.01
    joint: ''
    max_effort: 0.0
    stall_timeout: 1.0
    stall_velocity_threshold: 0.001
```