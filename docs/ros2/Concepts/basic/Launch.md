# 启动（Launch）

一个 ROS 2 系统通常由**多个节点**组成，这些节点可能运行在**不同的进程**（甚至不同的机器）上。虽然可以手动逐个启动这些节点，但这种方式很快就会变得繁琐。

ROS 2 的**启动系统**（launch system）旨在通过**单条命令**自动启动多个节点。它帮助用户描述整个系统的配置，并按描述执行。系统配置包括：
- 要运行哪些程序  
- 在何处运行它们  
- 传递哪些参数  
- ROS 特有的约定（例如通过不同配置复用组件）

此外，启动系统还负责**监控已启动进程的状态**，并**报告或响应**这些进程状态的变化。

上述所有内容都在一个“**启动文件**”（launch file）中定义。启动文件可以用 **XML**、**YAML** 或 **Python** 编写。编写完成后，可通过 `ros2 launch` 命令运行该文件，其中指定的所有节点将自动启动。

要开始编写和使用启动文件，请参阅 [启动教程](https://docs.ros.org/en/humble/Tutorials/Launching/Creating-Launch-Files.html)。  
如需更详细的信息，请参阅 [启动系统文档](https://docs.ros.org/en/humble/How-To-Guides/Launch-files-migration-guide.html)。