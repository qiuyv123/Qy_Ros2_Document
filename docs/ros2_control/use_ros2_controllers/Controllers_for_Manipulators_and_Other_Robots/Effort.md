# effort_controllers（力控制器）

`effort_controllers` 是一组使用 **“effort”**（力/力矩）关节命令接口的控制器。  
这些控制器在**控制器层级**可接受不同类型的关节级命令，例如：通过控制某关节的**输出力**来实现**目标位置**。

该软件包包含以下控制器：

---

## `effort_controllers/JointGroupEffortController`

这是 `forward_command_controller` 的一个特化版本，专为使用 **“effort” 关节接口**而设计。

### ROS 2 接口

#### 话题（Topics）

- **`~/commands`**（输入）  
  - 类型：`std_msgs/msg/Float64MultiArray`  
  - 用途：关节力/力矩命令数组

### 参数（Parameters）

该控制器**覆盖了** `forward_command_controller` 的 `interface` 参数，  
**唯一必需的参数是 `joints`**（关节名称列表）。

---

### 示例参数文件

```yaml
controller_manager:
  ros__parameters:
    update_rate: 100  # Hz

    effort_controller:
      type: effort_controllers/JointGroupEffortController

effort_controller:
  ros__parameters:
    joints:
      - slider_to_cart
```