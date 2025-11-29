# PID Controller（PID 控制器）

本控制器基于 `control_toolbox` 软件包中的 `PidROS` 实现。  
它支持两种使用方式：
- **直接通过话题发送参考值**
- **作为链式控制器**（可被前置或后置控制器调用）

此外，控制器支持使用**参考值的一阶导数及其反馈**，实现**二阶 PID 控制**。

---

## 控制器类型与接口匹配

根据硬件的 **参考/状态接口** 与 **命令接口** 类型，PID 控制器可配置为不同控制形式：

| 参考/状态接口 | 命令接口 | 控制类型 |
|--------------|--------|--------|
| `POSITION` | `VELOCITY` | PI |
| `VELOCITY` | `ACCELERATION` | PI |
| `VELOCITY` | `POSITION` | PD |
| `ACCELERATION` | `VELOCITY` | PD |
| `POSITION` | `POSITION` | PID |
| `VELOCITY` | `VELOCITY` | PID |
| `ACCELERATION` | `ACCELERATION` | PID |
| `EFFORT` | `EFFORT` | PID |

> ⚠️ **注意**：  
> 理论上可用 `Joint Trajectory Controller`（JTC）发送单点轨迹实现类似效果，但**不推荐**。  
> JTC 应用于**多点轨迹插值**（线性/三次/五次），而 PID 控制器无插值功能。  
> JTC 内部的 PID 项仅用于**通过 velocity/effort 接口控制硬件**，目的不同。

---

## 控制器执行逻辑

- 支持 **前馈模式**（feed-forward mode），通过 `feedforward_gain` 提升动态性能
- 若仅使用**一种参考/状态接口**，则仅计算**瞬时误差**
- 若使用**两种接口**，则第二接口视为第一接口的**一阶导数**（如 `position` + `velocity`）

---

## 使用方式

- **Pluginlib 库**：`pid_controller`  
- **插件名称**：`pid_controller/PidController`

---

## 控制器接口说明

### 参考接口（References，来自前置控制器）
```
<reference_and_state_dof_names[i]>/<reference_and_state_interfaces[j]> [double]
```
> 若 `reference_and_state_dof_names` 未定义，则使用 `dof_names`

### 命令接口（Commands）
```
<dof_names[i]>/<command_interface> [double]
```

### 状态接口（States）
```
<reference_and_state_dof_names[i]>/<reference_and_state_interfaces[j]> [double]
```
> 若 `reference_and_state_dof_names` 未定义，则使用 `dof_names`

---

## ROS 2 接口

### 订阅者（Subscribers）

- **`<controller_name>/reference`**（`control_msgs/msg/MultiDOFCommand`）  
  当 `in_chained_mode == false` 时使用

- **`<controller_name>/measured_state`**（`control_msgs/msg/MultiDOFCommand`）  
  仅当 `use_external_measured_states == true` 时启用

### 服务（Services）

- **`<controller_name>/set_feedforward_control`**（`std_srvs/srv/SetBool`）  
  启用/禁用前馈控制

### 发布者（Publishers）

- **`<controller_name>/controller_state`**（`control_msgs/msg/MultiDOFStateStamped`）  
  发布控制器内部状态

---

## 参数（Parameters）

使用 `generate_parameter_library` 管理参数。

### 核心参数

| 参数 | 类型 | 默认值 | 说明 | 约束 | 只读 |
|------|------|--------|------|------|------|
| `dof_names` | `string_array` | `{}` | 控制器使用的自由度/轴名称 | 非空、无重复 | ✅ |
| `reference_and_state_dof_names` | `string_array` | `{}` | 参考/状态接口对应的自由度名（可选） | 无重复 | ✅ |
| `command_interface` | `string` | `""` | 命令接口类型（如 `"position"`） | 非空 | ✅ |
| `reference_and_state_interfaces` | `string_array` | `{}` | 参考/状态接口列表（第二项应为第一项的导数） | 长度 < 3、非空 | ✅ |
| `use_external_measured_states` | `bool` | `false` | 是否从话题而非状态接口读取测量值 | — | — |

### 增益参数（按自由度配置）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `gains.<dof>.p` | `0.0` | 比例增益 |
| `gains.<dof>.i` | `0.0` | 积分增益 |
| `gains.<dof>.d` | `0.0` | 微分增益 |
| `gains.<dof>.antiwindup` | `false` | 是否启用抗积分饱和（启用时限制积分误差，否则限制积分输出） |
| `gains.<dof>.i_clamp_max` | `0.0` | 积分上限 |
| `gains.<dof>.i_clamp_min` | `0.0` | 积分下限 |
| `gains.<dof>.feedforward_gain` | `0.0` | 前馈增益 |
| `gains.<dof>.angle_wraparound` | `false` | 是否启用角度环绕处理（连续关节，误差归一化到 [-π, π]） |

---

## 示例参数文件

### 基础 PID 控制（独立模式）
```yaml
test_pid_controller:
  ros__parameters:
    dof_names: [joint1]
    command_interface: position
    reference_and_state_interfaces: ["position"]
    gains:
      joint1: {p: 1.0, i: 2.0, d: 3.0, i_clamp_max: 5.0, i_clamp_min: -5.0}
```

### 启用角度环绕
```yaml
test_pid_controller_angle_wraparound_on:
  ros__parameters:
    gains:
      joint1: {p: 1.0, i: 2.0, d: 3.0, i_clamp_max: 5.0, i_clamp_min: -5.0, angle_wraparound: true}
```

### 带前馈增益（单接口）
```yaml
test_pid_controller_with_feedforward_gain:
  ros__parameters:
    gains:
      joint1: {p: 0.5, i: 0.0, d: 0.0, i_clamp_max: 5.0, i_clamp_min: -5.0, feedforward_gain: 1.0}
```

### 带前馈增益（双接口：position + velocity）
```yaml
test_pid_controller_with_feedforward_gain_dual_interface:
  ros__parameters:
    dof_names: [joint1, joint2]
    command_interface: velocity
    reference_and_state_interfaces: ["position", "velocity"]
    gains:
      joint1: {p: 0.5, i: 0.3, d: 0.4, i_clamp_max: 5.0, i_clamp_min: -5.0, feedforward_gain: 1.0}
      joint2: {p: 0.5, i: 0.3, d: 0.4, i_clamp_max: 5.0, i_clamp_min: -5.0, feedforward_gain: 1.0}
```

### 作为前置控制器（自定义状态接口名）
```yaml
test_pid_controller:
  ros__parameters:
    dof_names: [joint1]
    reference_and_state_dof_names: [joint1state]  # 状态接口名为 joint1state
    command_interface: position
    reference_and_state_interfaces: ["position"]
```