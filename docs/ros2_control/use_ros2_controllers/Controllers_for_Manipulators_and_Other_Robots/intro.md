# 机械臂和其他机器人的控制器
控制器使用[通用硬件接口定义](https://github.com/ros-controls/ros2_control/blob/humble/hardware_interface/include/hardware_interface/types/hardware_interface_type_values.hpp)，并且可能会根据以下命令接口类型使用不同的命名空间：

位置控制器：hardware_interface::HW_IF_POSITION
速度控制器：hardware_interface::HW_IF_VELOCITY
力矩控制器：hardware_interface::HW_IF_ACCELERATION
力矩控制器：hardware_interface::HW_IF_EFFORT