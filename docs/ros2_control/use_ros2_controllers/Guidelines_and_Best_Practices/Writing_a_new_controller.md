# 编写一个新的控制器（Writing a New Controller）

在 `ros2_control` 框架中，**控制器是以动态库形式实现的插件**，由 Controller Manager 通过 `pluginlib` 接口动态加载。  
本文提供一份**分步指南**，帮助你创建控制器的源代码、基础测试和编译规则。

---

## 1. 准备包（Preparing package）

如果尚未创建控制器所在的 ROS 2 包，请先创建：

```bash
ros2 pkg create <controller_name_package> --build-type ament_cmake
```

> 使用 `--help` 查看更多选项。  
> 此命令支持生成库模板和 CMake 规则，可简化后续步骤。

> ✅ **注意**：包的构建类型必须为 `ament_cmake`。

---

## 2. 准备源文件目录结构（Preparing source files）

创建包后，确保包含以下文件和目录：

- `CMakeLists.txt`
- `package.xml`
- `include/<PACKAGE_NAME>/`（若不存在则手动创建）
- `src/`（若不存在则手动创建）

在对应目录中创建：
- 头文件：`include/<PACKAGE_NAME>/<controller_name>.hpp`
- 源文件：`src/<controller_name>.cpp`

**（可选）** 添加 `visibility_control.h`（用于 Windows 平台的符号导出控制）：  
可从 `ros2_controllers` 中任一控制器包复制，并将宏前缀替换为你的 `<PACKAGE_NAME>`。

---

## 3. 编写头文件（`.hpp`）

### 基本要求
- 使用 **头文件保护**（header guards），ROS 2 风格推荐使用 `#ifndef` / `#define`
- 包含必要头文件
- 使用**唯一命名空间**（通常为包名，采用 snake_case）
- 继承 `controller_interface::ControllerInterface`

### 示例模板
```cpp
#ifndef <CONTROLLER_NAME>_HPP_
#define <CONTROLLER_NAME>_HPP_

#include "controller_interface/controller_interface.hpp"
// #include "<PACKAGE_NAME>/visibility_control.h"  // 可选

namespace <controller_name>
{

class ControllerName : public controller_interface::ControllerInterface
{
public:
  ControllerName() = default;

  // 必须重写的生命周期与接口方法
  controller_interface::CallbackReturn on_init() override;
  controller_interface::CallbackReturn on_configure(const rclcpp_lifecycle::State & previous_state) override;
  controller_interface::CallbackReturn on_activate(const rclcpp_lifecycle::State & previous_state) override;
  controller_interface::CallbackReturn on_deactivate(const rclcpp_lifecycle::State & previous_state) override;
  controller_interface::return_type update(const rclcpp::Time & time, const rclcpp::Duration & period) override;

  controller_interface::InterfaceConfiguration command_interface_configuration() const override;
  controller_interface::InterfaceConfiguration state_interface_configuration() const override;

protected:
  // （可选）用于存储从参数读取的关节名和接口名
  std::vector<std::string> joint_names_;
  std::vector<std::string> command_interface_types_;
  std::vector<std::string> state_interface_types_;
};

}  // namespace <controller_name>

#endif  // <CONTROLLER_NAME>_HPP_
```

> 🔍 **提示**：具体方法签名请参考 `controller_interface/controller_interface.hpp` 或 `ros2_controllers` 中的现有控制器。

---

## 4. 编写源文件（`.cpp`）

### 实现要点

1. **包含头文件**并使用命名空间。
2. **（可选）实现构造函数**（也可在 `on_init` 中初始化成员变量）。
3. **`on_init()`**：
   - 通常先调用父类 `on_init()`
   - **声明控制器所需的节点参数**
   - 初始化变量、预分配内存（非实时部分）
   - 返回 `CallbackReturn::SUCCESS` 或 `ERROR`

4. **`on_configure()`**：
   - 读取参数（如关节列表、接口类型）
   - 验证配置合法性
   - 准备运行所需资源（非实时）

5. **接口配置方法**：
   - `command_interface_configuration()`
   - `state_interface_configuration()`
   - 返回以下之一：
     - `InterfaceConfiguration::NONE`
     - `InterfaceConfiguration::ALL`
     - `InterfaceConfiguration::INDIVIDUAL` + 接口名列表
   - 接口全名格式：`<joint_name>/<interface_type>`（如 `"joint1/position"`）

6. **`on_activate()`**：
   - 验证并排序接口
   - 设置初始值
   - **必须实时安全**：避免动态内存分配，尽可能简短

7. **`on_deactivate()`**：
   - 执行与 `on_activate` 相反的操作（如清零命令）
   - 通常为空，但也需**实时安全**

8. **`update()`**：
   - **主控制逻辑入口**
   - 从 `state_interface` 获取最新硬件状态
   - 向 `command_interface` 写入新命令
   - **严格遵守实时约束**

### ⚠️ 重要：导出插件类

在文件**末尾**（命名空间关闭后）添加：

```cpp
#include "pluginlib/class_list_macros.hpp"

PLUGINLIB_EXPORT_CLASS(
  <controller_name>::ControllerName,
  controller_interface::ControllerInterface
)
```

---

## 5. 编写 pluginlib 插件描述文件

在包根目录创建 `<controller_name>.xml`：

```xml
<library path="<controller_name_package>">
  <class
    name="<controller_name_package>/ControllerName"
    type="<controller_name>::ControllerName"
    base_class_type="controller_interface::ControllerInterface">
    <description>Brief description of your controller</description>
  </class>
</library>
```

> - `name`：控制器类型标识（Controller Manager 通过此名称加载）
> - `type` 和 `base_class_type`：必须与 `.cpp` 中 `PLUGINLIB_EXPORT_CLASS` 一致

---

## 6. 编写加载测试（Load Test）

创建 `test/test_load_<controller_name>.cpp`：  
可直接复制 `ros2_controllers` 中任一控制器的加载测试文件，并修改最后一行为：

```cpp
EXPECT_NO_THROW(
  ctr_loader_->createSharedInstance("<controller_name_package>/ControllerName")
);
```

---

## 7. 配置 `CMakeLists.txt`

### 添加依赖
```cmake
find_package(controller_interface REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)
```

### 编译控制器库
```cmake
add_library(${PROJECT_NAME} SHARED
  src/<controller_name>.cpp
)

target_include_directories(${PROJECT_NAME} PUBLIC
  include
)

ament_target_dependencies(${PROJECT_NAME}
  controller_interface
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle
)
```

### 导出 pluginlib 描述
```cmake
pluginlib_export_plugin_description_file(controller_interface <controller_name>.xml)
```

### 安装规则
```cmake
install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/
  DESTINATION include
)
```

### 测试编译（可选但推荐）
```cmake
if(BUILD_TESTING)
  find_package(ament_cmake_gmock REQUIRED)
  find_package(controller_manager REQUIRED)
  find_package(ros2_control_test_assets REQUIRED)

  ament_add_gmock(test_load_<controller_name>
    test/test_load_<controller_name>.cpp
  )
  target_include_directories(test_load_<controller_name> PUBLIC include)
  ament_target_dependencies(test_load_<controller_name>
    controller_interface
    controller_manager
    hardware_interface
    ros2_control_test_assets
  )
endif()
```

### （可选）导出库供其他包使用
```cmake
ament_export_libraries(${PROJECT_NAME})
```

确保最后调用：
```cmake
ament_package()
```

---

## 8. 更新 `package.xml`

### 运行依赖
```xml
<depend>controller_interface</depend>
<depend>hardware_interface</depend>
<depend>pluginlib</depend>
<depend>rclcpp</depend>
<depend>rclcpp_lifecycle</depend>
```

### 测试依赖
```xml
<test_depend>ament_cmake_gmock</test_depend>
<test_depend>controller_manager</test_depend>
<test_depend>hardware_interface</test_depend>
<test_depend>ros2_control_test_assets</test_depend>
```

---

## 9. 编译与测试

### 编译
```bash
# 在工作空间根目录执行
colcon build --packages-select <controller_name_package>
```

### 运行加载测试
```bash
source install/setup.bash
colcon test --packages-select <controller_name_package>
```

若测试通过，说明 Controller Manager 能通过 `pluginlib` 正确发现并加载你的控制器。

---

## 完成！

🎉 现在你已成功创建了一个可被 `ros2_control` 框架加载和使用的控制器！  
继续实现你的控制算法吧！

---

## 附：有用外部参考

- **[控制器脚手架生成脚本](https://rtw.b-robotized.com/master/use-cases/ros2_control/setup_controller.html)**

> ⚠️ **注意**：该脚本**目前仅推荐在 ROS 2 Humble 版本使用**，与 Jazzy 及更高版本的 API **不兼容**。