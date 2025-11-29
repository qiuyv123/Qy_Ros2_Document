# 组件

## 目录

- [ROS 1 - 节点与节点集](#ros-1---节点与节点集)
- [ROS 2 - 统一API](#ros-2---统一api)
- [组件容器](#组件容器)
- [编写组件](#编写组件)
- [CMake注册宏](#cmake注册宏)
  - [rclcpp_components_register_node](#rclcpp_components_register_node)
  - [rclcpp_components_register_nodes](#rclcpp_components_register_nodes)
- [使用组件](#使用组件)
- [实际应用](#实际应用)

## ROS 1 - 节点与节点集

在ROS 1中，您可以将代码编写为ROS节点或ROS节点集。ROS 1节点被编译成可执行文件。而ROS 1节点集则被编译成共享库，并在运行时由容器进程加载。

## ROS 2 - 统一API

在ROS 2中，推荐的编写代码方式类似于节点集——我们称之为组件。这使得向现有代码添加常见的概念（如生命周期）变得容易，并且避免了ROS 1中的不同API这一最大缺点，因为这两种方法使用相同的API。

**注意**

仍可以选择类似“自己编写主函数”的节点样式的编写方式，但对于常见情况，不推荐这样做。

通过将进程布局作为部署时间的决策，用户可以选择：

- 在单独的进程中运行多个节点，以获得进程/故障隔离的好处，还可以更容易地调试各个节点。
- 在单个进程中运行多个节点，以获得较低的开销和可选的更高效通信（参见[进程内通信](https://docs.ros.org/en/kilted/Tutorials/Demos/Intra-Process-Communication.html)）。

此外，可以使用ros2 launch通过专用的启动操作自动化这些操作。

## 组件容器

组件容器是一个主机进程，在同一进程空间中允许在运行时加载和管理多个组件。

目前可用的通用组件容器类型包括：

- **component_container**：使用单一`SingleThreadedExecutor`执行所有组件。

??? note "点击查看代码"

        ```cpp
        #include <memory>

        #include "rclcpp/rclcpp.hpp"

        #include "rclcpp_components/component_manager.hpp"

        int main(int argc, char * argv[])
        {
        /// Component container with a single-threaded executor.
        rclcpp::init(argc, argv);
        auto exec = std::make_shared<rclcpp::executors::SingleThreadedExecutor>();
        auto node = std::make_shared<rclcpp_components::ComponentManager>(exec);
        exec->add_node(node);
        exec->spin();
        }
        ```

- **component_container_mt**：使用单一`MultiThreadedExecutor`执行组件。

??? note "点击查看代码"

        ```cpp
        #include <memory>

        #include "rclcpp/rclcpp.hpp"

        #include "rclcpp_components/component_manager.hpp"

        int main(int argc, char * argv[])
        {
        /// Component container with a multi-threaded executor.
        rclcpp::init(argc, argv);

        auto exec = std::make_shared<rclcpp::executors::MultiThreadedExecutor>();
        auto node = std::make_shared<rclcpp_components::ComponentManager>();
        if (node->has_parameter("thread_num")) {
            const auto thread_num = node->get_parameter("thread_num").as_int();
            exec = std::make_shared<rclcpp::executors::MultiThreadedExecutor>(
            rclcpp::ExecutorOptions{}, thread_num);
            node->set_executor(exec);
        } else {
            node->set_executor(exec);
        }
        exec->add_node(node);
        exec->spin();
        }
        ```
- **component_container_isolated**：为每个组件使用专用的executor，可以是`SingleThreadedExecutor`（默认）或`MultiThreadedExecutor`。

??? note "点击查看代码"

        ```cpp
        #include <memory>
        #include <vector>
        #include <string>

        #include "rclcpp/rclcpp.hpp"
        #include "rclcpp/utilities.hpp"
        #include "rclcpp_components/component_manager_isolated.hpp"

        int main(int argc, char * argv[])
        {
        /// Component container with dedicated single-threaded executors for each components.
        rclcpp::init(argc, argv);
        // parse arguments
        bool use_multi_threaded_executor{false};
        std::vector<std::string> args = rclcpp::remove_ros_arguments(argc, argv);
        for (auto & arg : args) {
            if (arg == std::string("--use_multi_threaded_executor")) {
            use_multi_threaded_executor = true;
            }
        }
        // create executor and component manager
        auto exec = std::make_shared<rclcpp::executors::SingleThreadedExecutor>();
        rclcpp::Node::SharedPtr node;
        if (use_multi_threaded_executor) {
            using ComponentManagerIsolated =
            rclcpp_components::ComponentManagerIsolated<rclcpp::executors::MultiThreadedExecutor>;
            node = std::make_shared<ComponentManagerIsolated>(exec);
        } else {
            using ComponentManagerIsolated =
            rclcpp_components::ComponentManagerIsolated<rclcpp::executors::SingleThreadedExecutor>;
            node = std::make_shared<ComponentManagerIsolated>(exec);
        }
        exec->add_node(node);
        exec->spin();
        }
        ```

有关executor类型的更多信息，请参阅[Executor类型](https://docs.ros.org/en/kilted/Concepts/Intermediate/About-Executors.html#typesofexecutors)。有关每个组件容器的选项的更多信息，请参阅组合教程中的[组件容器类型](https://docs.ros.org/en/kilted/Tutorials/Intermediate/Composition.html#componentcontainertypes)。

## 编写组件

由于组件仅构建为共享库，因此没有main函数（参阅[Talker源代码](https://github.com/ros2/demos/blob/kilted/composition/src/talker_component.cpp)）。组件通常是`rclcpp::Node`的子类。由于无法控制线程，它不应该在其构造函数中执行长时间运行或阻塞的任务。相反，它可以使用定时器来获取周期性通知。此外，它可以创建发布者、订阅者、服务器和客户端。

使此类成为组件的一个重要因素是，该类使用`rclcpp_components`包中的宏进行注册（参见源代码的最后一行）。这使其在加载其库到正在运行的进程时可以被发现——它充当某种入口点。

此外，一旦创建组件，必须将其注册到索引中以便工具能够发现它。

```cmake
add_library(talker_component SHARED src/talker_component.cpp)
rclcpp_components_register_nodes(talker_component "composition::Talker")
# 在同一共享库中注册多个组件时，使用多次调用
# rclcpp_components_register_nodes(talker_component "composition::Talker2")
```

有关示例，请参阅[此教程](https://docs.ros.org/en/kilted/Tutorials/Intermediate/Writing-a-Composable-Node.html)。

**注意**

为了使component_container能够找到所需的组件，必须从已源化相应工作区的shell中执行或启动它。

## CMake注册宏

ROS 2提供了两个CMake宏来注册组件，每个宏都有不同的用途：

### rclcpp_components_register_node

此宏注册一个组件并生成独立的可执行文件。当您希望同时具有组合性和能够在独立进程中运行节点的能力时使用此宏。

```cmake
add_library(talker_component SHARED src/talker_component.cpp)
rclcpp_components_register_node(talker_component
  PLUGIN "composition::Talker"
  EXECUTABLE talker)
```

### rclcpp_components_register_nodes

此宏注册一个或多个组件以进行运行时组合，而不生成独立的可执行文件。当您希望在运行时加载到组件容器的纯组件库时使用此宏。

```cmake
add_library(talker_component SHARED src/talker_component.cpp)
rclcpp_components_register_nodes(talker_component "composition::Talker")
```

## 使用组件

[组件包](https://github.com/ros2/demos/tree/kilted/composition)包含几种不同方法来使用组件。最常用的方法有以下三种：

- 启动（[通用容器进程](https://github.com/ros2/rclcpp/blob/kilted/rclcpp_components/src/component_container.cpp)），并调用容器提供的ROS服务[load_node](https://github.com/ros2/rcl_interfaces/blob/kilted/composition_interfaces/srv/LoadNode.srv)。该ROS服务将加载指定的库名和包名的组件并在运行的进程中开始执行它。除了编程调用ROS服务，还可以使用[命令行工具](https://github.com/ros2/ros2cli/tree/kilted/ros2component)通过传递命令行参数来调用ROS服务。

- 创建一个[自定义可执行文件]，其中包含在编译时已知的多个节点。需要为每个组件提供头文件（对第一种情况而言严格上不需要）。

- 创建一个launch文件，并使用`ros2 launch`来创建包含多个组件的容器进程。

## 实际应用

尝试[组件demos](https://docs.ros.org/en/kilted/Tutorials/Intermediate/Composition.html)。