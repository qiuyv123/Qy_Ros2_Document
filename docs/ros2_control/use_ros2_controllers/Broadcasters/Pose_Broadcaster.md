# Pose Broadcaster（位姿广播器）

该广播器用于发布由**机器人或传感器测量的位姿**（pose），消息类型为 `geometry_msgs/msg/PoseStamped`，并可**选择性地发布为 TF 变换**。

该控制器是 `PoseSensor` **语义组件**（semantic component）的封装（详见 `controller_interface` 软件包）。

---

## 参数（Parameters）

本控制器使用 `generate_parameter_library` 管理参数。完整参数说明见 `src/` 目录下的参数定义文件。

---

### 参数列表

| 参数 | 类型 | 默认值 | 说明 | 约束 |
|------|------|--------|------|------|
| `frame_id` | `string` | `""` | 发布位姿所用的**父坐标系名称** | **非空** |
| `pose_name` | `string` | `""` | 用作状态接口前缀的基础名称。<br>自动生成的接口包括：<br>`<pose_name>/position.x` 到 `.z`<br>`<pose_name>/orientation.x` 到 `.w` | **非空** |

### TF 发布参数（可选）

| 参数 | 默认值 | 说明 | 约束 |
|------|--------|------|------|
| `tf.enable` | `true` | 是否将位姿发布为 TF 变换 | — |
| `tf.child_frame_id` | `""` | TF 变换的子坐标系名称。<br>若为空，则默认使用 `pose_name` | — |
| `tf.publish_rate` | `0.0` | TF 发布频率限制（Hz）。<br>若设为 `0`，则**不限制发布速率** | ≥ 0.0 |

---

## 示例参数文件

```yaml
test_pose_broadcaster:
  ros__parameters:
    pose_name: "test_pose"
    frame_id: "pose_frame"
```

> 📁 完整示例位于控制器包的 `test/` 目录中。