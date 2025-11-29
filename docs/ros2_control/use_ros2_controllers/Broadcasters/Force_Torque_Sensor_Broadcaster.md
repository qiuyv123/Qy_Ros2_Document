# Force Torque Sensor Broadcaster（力/力矩传感器广播器）

该广播器用于发布来自机器人或传感器的**力/力矩状态接口**（force/torque state interfaces）的消息，消息类型为 `geometry_msgs/msg/WrenchStamped`。

该控制器是 `ForceTorqueSensor` **语义组件**（semantic component）的封装（详见 `controller_interface` 软件包）。

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。完整参数说明见 `src/` 目录下的参数定义文件。

> ⚠️ **注意**：接口定义有两种方式，**不可同时使用**：
> - 使用 `sensor_name`（自动推导标准接口名）
> - 使用 `interface_names`（自定义每个轴的接口名）

---

### 完整参数列表

| 参数 | 类型 | 默认值 | 说明 | 约束 |
|------|------|--------|------|------|
| `frame_id` | `string` | `""` | 发布值所用的传感器坐标系名称 | **非空** |

#### 方式一：通过传感器名称自动推导接口（推荐用于标准 6D F/T 传感器）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `sensor_name` | `""` | 传感器名称，作为接口前缀。若使用此方式，将自动采用标准 6D 接口命名：<br>`<sensor_name>/force.x`、`<sensor_name>/force.y`、…、`<sensor_name>/torque.z` |

#### 方式二：自定义接口名称（适用于非标准命名或少于 6 轴的传感器）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `interface_names.force.x` | `""` | X 轴力的状态接口名称 |
| `interface_names.force.y` | `""` | Y 轴力的状态接口名称 |
| `interface_names.force.z` | `""` | Z 轴力的状态接口名称 |
| `interface_names.torque.x` | `""` | 绕 X 轴力矩的状态接口名称 |
| `interface_names.torque.y` | `""` | 绕 Y 轴力矩的状态接口名称 |
| `interface_names.torque.z` | `""` | 绕 Z 轴力矩的状态接口名称 |

> ✅ **提示**：只需定义至少一个 `interface_names` 下的接口，即可启用自定义模式。  
> 此功能支持**少于六轴的力传感器**（如仅测 Z 向力的称重传感器）。

---

## 示例参数文件

```yaml
test_force_torque_sensor_broadcaster:
  ros__parameters:
    frame_id: "fts_sensor_frame"
```

> 📁 完整示例位于控制器包的 `test/` 目录中。