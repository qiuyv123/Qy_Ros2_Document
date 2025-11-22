# 调试（Debugging）

所有控制器和硬件组件都是作为插件加载到 `controller_manager` 中的。因此，**调试器必须附加到 `controller_manager` 进程上**。  
如果你的机器人或机器上运行了多个 `controller_manager` 实例，则需要将调试器附加到**与你要调试的硬件组件或控制器相关联的那个 `controller_manager`** 上。

---

## 操作指南（How-To）

### 1. 安装所需工具
在系统中安装 `xterm`、`gdb` 和 `gdbserver`：

```bash
sudo apt install xterm gdb gdbserver
```

### 2. 使用带调试信息的构建类型
确保你使用的是 **“Debug”** 或 **“带调试信息的 Release”**（RelWithDebInfo）构建：

```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
```

> **注意**：在纯 Release 构建中，编译器优化可能导致断点行为异常（例如代码行被优化掉）。  
> 若需精确调试，建议使用完整 Debug 构建：
> ```bash
> colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug
> ```

### 3. 修改启动文件以附加调试器

#### 方式 A：直接使用 gdb 命令行界面（CLI）

在启动文件中的 `controller_manager` 节点定义中添加 `prefix`：

```python
prefix=['xterm -e gdb -ex run --args']
```

> 由于 `ros2launch` 的工作机制，必须在**单独的终端窗口**中运行该节点。

#### 方式 B：使用 gdbserver（推荐用于远程或 IDE 调试）

添加：

```python
prefix=['gdbserver localhost:3000']
```

随后，你可以通过 gdb CLI 或任意 IDE（如 VS Code、CLion）连接到 `localhost:3000`。  
**注意**：启动调试器的终端必须已 `source` 你的工作空间，以确保路径正确解析。

---

### 示例启动文件节点定义

```python
# 获取 controller_manager 的配置文件
controller_config_file = get_package_file("<package name>", "config/controllers.yaml")

controller_manager = Node(
    package="controller_manager",
    executable="ros2_control_node",
    parameters=[controller_config_file],
    output="both",
    emulate_tty=True,
    remappings=[
        ("~/robot_description", "/robot_description")
    ],
    prefix=['xterm -e gdb -ex run --args']  # 或使用 prefix=['gdbserver localhost:3000']
)

ld.add_action(controller_manager)
```

---

## 附加说明（Additional notes）

### 调试插件（Debugging plugins）

你**只能在插件加载后**设置断点。在 `ros2_control` 上下文中，这意味着必须在**控制器或硬件组件被加载之后**。

### 针对特定包构建调试版本

通常只需为要调试的特定包生成调试信息：

```bash
colcon build --packages-select [package_name] --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
# 或
colcon build --packages-select [package_name] --cmake-args -DCMAKE_BUILD_TYPE=Debug
```

### 实时上下文中的调试警告（Realtime）

> ⚠️ **警告**：  
> 控制器的 `update`/`on_activate`/`on_deactivate` 方法，以及硬件组件的 `read`/`write`/`on_activate`/`perform_command_mode_switch` 方法，**均运行在实时更新循环上下文中**。  
> 在这些函数中设置断点**可能导致严重问题**，最坏情况下甚至会**损坏硬件**。

**建议**：
- 在实时上下文中**优先使用有意义的日志**（需谨慎，避免影响实时性）
- 或添加额外的**调试用状态接口**（对于控制器，也可使用发布者）

不过，**使用 gdb 运行 `controller_manager` 和插件**对于调试**段错误**（segfault）等崩溃问题仍然非常有用，因为它能提供完整的**回溯栈**（backtrace）。