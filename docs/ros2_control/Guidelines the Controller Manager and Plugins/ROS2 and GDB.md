# 在 ROS2 中使用 GDB：一份参考指南

**关键词**：#ros #ros2 #gdb #debugging

自从我全职转向 ROS2 开发至今已将近一年。就像我之前写的那篇 **[在 ROS1 中使用 GDB](https://juraph.com/miscellaneous/ros_and_gdb/)**的文章一样，我希望为自己（也可能为其他人）整理一份关于在 ROS2 中构建高效调试工作流的参考指南。与 ROS1 指南不同，本文将重点介绍**启动文件**（launch files）的调试方法，因为目前关于 ROS2 调试的信息相对较少。

---

## 步骤 1：使用调试标志编译

```bash
colcon build [flags] --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
```

你**不需要为整个工作空间启用调试标志**，只需为你要调试的包编译即可。因此，你可以使用 `--packages-select` 仅编译单个包：

```bash
colcon build --packages-select [package_name] --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
```

---

## 步骤 2：启动带 GDB 的节点

将 GDB 附加到进程的两种最常见方法是：

- **方式 1**：直接在 GDB 内部执行进程  
- **方式 2**：启动一个 `gdbserver` 实例，并在另一个终端中连接它  

后者在**远程开发**场景中尤其有用（例如无法轻松创建新终端会话时）。

---

### 情况 1：独立启动节点

#### 方式 1：通过 GDB 直接启动
```bash
$ gdb
(gdb) exec-file [workspace]/build/lib/[ros_package_name]/[node_name]
(gdb) start
```

#### 方式 2：使用 `ros2 run` 启动并附加 `gdbserver`
```bash
$ ros2 run --prefix 'gdbserver localhost:3000' [package_name] [executable_name]
```

在**第二个终端**中连接：
```bash
$ gdb
(gdb) target remote localhost:3000
```

---

### 情况 2：在启动文件中配置 GDB

你可以在启动文件中通过 **`launch prefix`** 的形式启动 GDB，这样调试会话就成为标准启动流程的一部分。当你需要为节点设置大量参数时，这种方式尤其方便。

#### Python 启动文件
```python
your_node = Node(
    package='package',
    executable='executable',
    name='name',
    prefix=['gdbserver localhost:3000'],
    output='screen')
```

#### XML 启动文件
```xml
<launch>
  <node pkg="pkg" exec="node" name="name" launch-prefix="gdbserver localhost:3000">
  </node>
</launch>
```

#### 启动终端窗口
如果你不想使用 `gdbserver`，也可以直接用终端启动 GDB：
```python
prefix=['xterm -e gdb -ex run --args']
```
（可将 `xterm` 替换为你喜欢的终端）

---

## 优化工作流：添加一个 `.bashrc` 函数

将以下函数添加到你的 `~/.bashrc` 中，即可通过简单命令启动 GDB 调试：

```bash
debug_node()
{
    # 在指定节点上启动 GDB 会话
    # 用法: debug_node {node_name}
    local full_name="$(ros2 pkg executables | grep -i $*)"
    local node_name=$(echo "$full_name" | cut -d' ' -f2)
    local pkg_name=$(echo "$full_name" | cut -d' ' -f1)
    if [ -z "$node_name" ]
    then
        echo "ERROR! 找不到节点: $*，可用选项如下："
        ros2 pkg executables
    else
        local pkg_dir=$(ros2 pkg prefix $pkg_name)
        gdb --args $pkg_dir/lib/$pkg_name/$node_name
    fi
}
```

使用方式：
```bash
$ debug_node your_node_name arg1 arg2
```

如果找不到指定节点，它还会列出所有可用的可执行文件供你参考。

---

## 推荐工具

### GDB Dashboard
毫无疑问，我最喜欢的 GDB 扩展是 **[GDB Dashboard](https://github.com/cyrus-and/gdb-dashboard)**。它会彻底改变你使用 GDB 的方式，强烈推荐！
![alt text](Screenshot.png)
（图示：GDB Dashboard 界面，包含源码、寄存器、堆栈、内存等面板）

### Beej’s GDB 快速指南
再次推荐：**[Beej’s Quick Guide to GDB](https://beej.us/guide/bggdb/)** 是学习 GDB 的最佳入门资料。如果你刚接触 GDB，强烈建议阅读。

---

## 结语

未来，我希望能尝试使用 **`rr`**（Record and Replay Debugger）进行调试。我听到的关于它的评价都非常好。