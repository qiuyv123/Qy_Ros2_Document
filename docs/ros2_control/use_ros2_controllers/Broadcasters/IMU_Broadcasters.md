# IMU Sensor Broadcaster（IMU 传感器广播器）

该广播器用于发布来自 **IMU 传感器**（惯性测量单元）的状态数据，发布消息类型为 `sensor_msgs/msg/Imu`。

该控制器是 `IMUSensor` **语义组件**（semantic component）的封装（详见 `controller_interface` 软件包）。

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。完整参数说明见 `src/` 目录下的参数定义文件。

---

### 参数列表

| 参数 | 类型 | 默认值 | 说明 | 约束 |
|------|------|--------|------|------|
| `sensor_name` | `string` | `""` | 传感器名称，用作所有接口的前缀。自动生成的标准接口包括：<br>`<sensor_name>/orientation.x` 到 `.z`<br>`<sensor_name>/angular_velocity.x` 到 `.z`<br>`<sensor_name>/linear_acceleration.x` 到 `.z` | **非空** |
| `frame_id` | `string` | `""` | 发布数据所用的传感器坐标系名称 | **非空** |

#### 协方差参数（静态）

| 参数 | 默认值 | 说明 | 约束 |
|------|--------|------|------|
| `static_covariance_orientation` | `{0.0, ..., 0.0}`（9 个） | 方向（四元数）协方差矩阵（行优先，3×3） | 长度必须为 9 |
| `static_covariance_angular_velocity` | `{0.0, ..., 0.0}`（9 个） | 角速度协方差矩阵（行优先，3×3） | 长度必须为 9 |
| `static_covariance_linear_acceleration` | `{0.0, ..., 0.0}`（9 个） | 线加速度协方差矩阵（行优先，3×3） | 长度必须为 9 |

> 💡 **提示**：协方差矩阵按行展开，例如 `[σₓₓ, σₓᵧ, σₓ_z, σᵧₓ, ..., σ_zz]`。

---

## 示例参数文件

```yaml
test_imu_sensor_broadcaster:
  ros__parameters:
    sensor_name: "imu_sensor"
    frame_id: "imu_sensor_frame"
```

> 📁 完整示例位于控制器包的 `test/` 目录中。