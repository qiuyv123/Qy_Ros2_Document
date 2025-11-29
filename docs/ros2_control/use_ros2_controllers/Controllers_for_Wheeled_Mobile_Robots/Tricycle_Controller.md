# Tricycle Controller（三轮控制器）

`tricycle_controller` 适用于具有**单个双驱动轮**（同时具备牵引与转向功能）的移动机器人。  
典型示例：**前轮驱动并转向**的三轮机器人，后轴配备两个被动随动轮。

- **输入**：机器人 `base_link` 的速度指令（Twist）
- **输出**：转换为牵引（速度）和转向（角度）指令
- **里程计**：基于硬件反馈计算并发布

> 📚 有关移动机器人运动学基础及本文使用的术语，请参阅：[轮式移动机器人运动学（Wheeled Mobile Robot Kinematics）](./mobile_robot_kinematics.html)

---

## 其他特性

- **实时安全实现**（Realtime-safe implementation）  
- **里程计发布**（Odometry publishing）  
- **任务空间的速度、加速度和加加速度**（jerk）  
- **命令超时自动停止**（Automatic stop after command timeout）

---

## ROS 2 接口

### 订阅者（Subscribers）

- **`~/cmd_vel`**（`geometry_msgs/msg/TwistStamped`）  
  控制器速度指令输入。  
  控制器仅提取：
  - 线速度的 **x 分量**（前进/后退）
  - 角速度的 **z 分量**（偏航旋转）  
  其他分量被忽略。