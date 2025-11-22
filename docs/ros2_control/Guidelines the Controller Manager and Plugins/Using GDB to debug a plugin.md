# 如何使用 GDB 调试插件（How to Use GDB to Debug a Plugin）

> 原问题发布时间：2012 年 6 月 6 日  
> 浏览量：2,000+ 次

## 问题描述

我正在尝试使用 GDB 调试一个程序的插件。之前看到一个问题提到 GDB 的 `directory` 命令可能有帮助。  
当我尝试在插件代码中设置断点时，GDB 报错：  
**“No source file named...”**（找不到名为……的源文件）。

我尝试使用 `directory` 命令指定源代码路径，但似乎没有效果。有什么解决办法吗？谢谢！

---

## 最佳答案

当你看到错误：

```
No source file named...
```

这意味着 **GDB 还无法找到插件的源文件**。原因通常有两个：

### 1. 插件尚未加载
GDB **只有在插件被实际加载到目标进程**（inferior process）之后，才能识别其符号和源文件。

✅ **解决方法**：
- 启动主程序后，在 GDB 中运行：
  ```gdb
  (gdb) info shared
  ```
  查看你的插件是否已加载。
- 如果**尚未加载**，你可以设置一个 **“延迟断点”**（deferred breakpoint）。  
  GDB 通常会提示你：“是否要设置一个将来加载时生效的断点？”（前提是 `set confirm on` 为默认开启状态）。

### 2. 插件未包含调试信息
如果 `info shared` 显示插件**已经加载**，但仍然无法设置断点，说明该插件**编译时未包含调试符号**。

✅ **解决方法**：
- 重新编译插件，添加 `-g` 编译选项：
  ```bash
  g++ -g -fPIC -shared -o my_plugin.so my_plugin.cpp
  ```
- 确保使用 **Debug 模式** 或至少包含调试信息（如 `RelWithDebInfo`）进行构建。

---

## 补充建议

- 即使指定了源码路径（`directory /path/to/source`），如果插件未加载或无调试信息，GDB 依然无法设置断点。
- 对于动态加载的插件（如 `.so` 或 `.dll`），GDB 只有在 `dlopen` 被调用后才能解析其符号表。

---

> 💡 **提示**：在 ROS 2 或 `ros2_control` 等框架中调试控制器/硬件插件时，同样适用此原则——必须等待插件被 `controller_manager` 加载后，才能在其中设置有效断点。

**[原文链接](https://stackoverflow.com/questions/10919832/how-to-use-gdb-to-debug-a-plugin)**(原文是stack overflow上的一个回答,这里笔者整理成了markdown的格式)