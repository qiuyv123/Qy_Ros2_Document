# Range Sensor Broadcaster（距离传感器广播器）

该广播器用于发布来自 **距离传感器**（Range Sensor）的状态数据，发布消息类型为 `sensor_msgs/msg/Range`。

该控制器是 `RangeSensor` **语义组件**（semantic component）的封装（详见 `controller_interface` 软件包）。

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。完整参数说明见 `src/` 目录下的参数定义文件。

---

### 参数列表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `sensor_name` | `string` | `""` | 传感器名称，用作接口前缀（若未单独定义接口名） |
| `frame_id` | `string` | `""` | 发布数据所用的传感器坐标系名称 |
| `radiation_type` | `int` | `0` | 传感器使用的辐射类型：<br>`0` = 超声波（Ultrasonic）<br>`1` = 红外线（Infrared） |
| `field_of_view` | `double` | `0.52` | 有效测距的弧度角（单位：弧度） |
| `min_range` | `double` | `0.52` | 最小测距范围（单位：米） |
| `max_range` | `double` | `4.0` | 最大测距范围（单位：米） |

---

## 示例参数文件

### 基础示例
```yaml
range_sensor_broadcaster:
  ros__parameters:
    field_of_view: 0.52
    frame_id: ''
    max_range: 4.0
    min_range: 0.52
    radiation_type: 0
    sensor_name: ''
```

### 完整测试示例（位于 `test/` 目录）
```yaml
test_range_sensor_broadcaster:
  ros__parameters:
    # 必填参数
    sensor_name: "range_sensor"
    frame_id: "range_sensor_frame"

    # 覆盖默认值的参数（用于验证配置生效）
    radiation_type: 1          # 红外
    field_of_view: 0.1         # 0.1 弧度
    min_range: 0.10            # 10 厘米
    max_range: 7.0             # 7 米
```