# 执行器

## 概述
在ROS 2中，执行管理由执行器处理。该执行器使用一个或多个操作系统线程来调用订阅、定时器、服务服务器、动作服务器等的回调函数，以响应传入的消息和事件。尽管`spin`机制与ROS 1的基本API非常相似，但显式的执行器类提供了对执行管理的更多控制（在rclcpp中的`executor.hpp`，rclpy中的`executors.py`，或rclc中的`executor.h`）。

本文档将主要关注C++客户端库`rclcpp`。

## 基本用法
最简单的情况下，可以通过调用`rclcpp::spin(..)`让主线程处理节点的传入消息和事件：

```cpp
int main(int argc, char* argv[])
{
   // 部分初始化代码。
   rclcpp::init(argc, argv);
   ...

   // 实例化一个节点。
   rclcpp::Node::SharedPtr node = ...

   // 运行执行器。
   rclcpp::spin(node);

   // 关闭并退出。
   ...
   return 0;
}
```

调用`spin(node)`基本上是实例化并调用单线程执行器，这是最简单的执行器：

```cpp
rclcpp::executors::SingleThreadedExecutor executor;
executor.add_node(node);
executor.spin();
```
通过调用执行器实例的`spin()`方法，当前线程开始查询rcl和中间件层的传入消息和其他事件，并调用相应的回调函数直到节点关闭。为了不干扰中间件的QoS设置，传入消息不会存储在客户端库层的队列中，而是保留在中间件中，直到被回调函数取走处理。（这是与ROS 1的关键区别）。等待集用于通知执行器关于中间件层中可用的消息，每个队列对应一个二进制标志。等待集还用于检测计时器到期的情况。

![](../../_images/executors_basic_principle.png)

单线程执行器还由组件的容器进程使用，即在没有显式主函数的情况创建和执行节点的所有情况。

## 执行器类型
目前，rclcpp提供了三种从公共基类派生的执行器类型：

```dot
digraph Flatland {
    Executor -> SingleThreadedExecutor [dir = back, arrowtail = empty];
    Executor -> MultiThreadedExecutor [dir = back, arrowtail = empty];
    Executor -> StaticSingleThreadedExecutor [dir = back, arrowtail = empty];
    Executor [shape=polygon,sides=4];
    SingleThreadedExecutor [shape=polygon,sides=4];
    MultiThreadedExecutor [shape=polygon,sides=4];
    StaticSingleThreadedExecutor [shape=polygon,sides=4];
}
```

多线程执行器创建可配置数量的线程以允许多个消息或事件并行处理。

注意：
静态单线程执行器已弃用，建议使用单线程执行器。静态单线程执行器旨在减少扫描节点实体（如订阅、定时器、服务服务器、动作服务器等）方面的运行时开销。这些运行时改进现在也适用于其他所有执行器。此外，静态单线程执行程序存在一些问题，例如，在`spin_some`中不尊重最大持续时间。由于这些问题不稳定，针对静态单线程执行器跳过了某些单元测试。您可以在ROS Discourse上查看该主题的更多细节：《The ROS 2 C++ Executors》。

可以通过为每个节点调用 add_node(..) 使用这三种执行器中的任何一个来处理多个节点。

```cpp
rclcpp::Node::SharedPtr node1 = ...
rclcpp::Node::SharedPtr node2 = ...
rclcpp::Node::SharedPtr node3 = ...

rclcpp::executors::SingleThreadedExecutor executor;
executor.add_node(node1);
executor.add_node(node2);
executor.add_node(node3);
executor.spin();
```

在上述示例中，单线程执行器的一个线程用于同时为三个节点提供服务。而在多线程执行器的情况下，实际的并行性取决于回调组。

## 回调组
ROS 2 允许将一个节点的回调组织成组。在rclcpp中，可以使用Node类的create_callback_group函数创建这样一个回调组。在rclpy中，同样的操作可以通过调用特定类型的回调组构造函数来实现。必须在整个节点的执行过程中保存回调组（例如作为类成员），否则执行器将无法触发回调。然后可以在创建订阅、定时器等时指定这个回调组。例如，可以通过订阅选项进行设置：

```cpp
C++Python
my_callback_group = create_callback_group(rclcpp::CallbackGroupType::MutuallyExclusive);

rclcpp::SubscriptionOptions options;
options.callback_group = my_callback_group;

my_subscription = create_subscription<Int32>("/topic", rclcpp::SensorDataQoS(), callback, options);
```

未指定回调组创建的所有订阅、定时器等会被分配到默认的回调组中。该默认的回调组可以通过NodeBaseInterface::get_default_callback_group()在rclcpp以及通过Node.default_callback_group在rclpy中获取。

回调组有两种类型，需要在实例化时指定：

- 互斥：此组的回调不能并行执行。
- 可重入：此组的回调可以并行执行。

来自不同回调组的回调总是可以并行执行。多线程执行器利用其线程作为线程池，根据条件尽可能地并行处理回调。有关如何高效使用回调组的提示，请参阅《使用回调组》。

rclcpp 中的执行器基类也具有 add_callback_group(..) 函数，允许将不同的回调组分配给不同的执行器。通过配置底层线程，可以使用操作系统调度优先执行特定回调。例如，可以优先执行控制循环的订阅和定时器，而不是节点的所有其他订阅和标准服务。examples_rclcpp_cbg_executor 包提供的演示了此机制。

## 调度语义
如果回调的处理时间短于消息和事件发生的周期，执行器基本上按先进先出的顺序处理它们。然而，如果有回调的处理时间较长，消息和事件将在堆栈的较低层排队。等待集机制向执行器报告关于这些队列的非常少量的信息。具体来说，它仅报告是否存在特定主题的任何消息。执行器借助此信息以循环的方式但不是以先进先出的方式处理消息（包括服务和动作）。

下图可视化了此调度语义：

![](../../_images/executors_scheduling_semantics.png)

此语义首先由 Casini 等人在 2019 年的 ECRTS 会议上的一篇论文描述。（注意：该论文还解释了计时器事件优先于所有其他消息。这个优先级在 Eloquent 中已被移除。）

## 展望

虽然 rclcpp 的三个执行器适合多数应用，但在对于需要严格的执行时间和确定性以及自定义执行顺序的实时应用中，这些执行器存在一些限制。以下是对这些问题的一些总结：

- 复杂且混合的调度语义。理想情况下，你希望有确定的调度语义以便执行正式的时序分析。
- 回调可能受到优先级倒置的影响。高优先级/callback 可能会被低优先级/callback 阻塞。
- 对回调执行顺序没有明确的控制。
- 没有内建触发特定话题的能力。
- 此外，执行器在CPU和内存使用方面的开销相当大。

这些问题是部分通过以下发展得到解决：

- **rclcpp 等待集**：rclcpp 的等待集类允许直接等待订阅、定时器、服务服务器、动作服务器等，而不是使用执行器。它可以用来实现确定性的、用户定义的处理序列，即使是从不同的订阅一起处理多个消息。examples_rclcpp_wait_set 包提供了用户级等待集机制使用的几个例子。
- **rclc 执行器**：这是为 micro-ROS 开发的 C 客户端库 rclc 中的执行器，提供了对回调执行顺序的细粒度控制，并允许自定义触发条件来激活回调。此外，它实现了逻辑执行时间(LET)语义的想法。

## 进一步信息

- Michael Pöhnl et al.: "ROS 2 Executor: How to make it efficient, real-time and deterministic?". 工作坊，ROS World 2021. 虚拟活动. 2021年10月19日。
- Ralph Lange: "高级执行管理与 ROS 2". ROS 工业会议. 虚拟活动. 2020年12月16日。
- Daniel Casini, Tobias Blass, Ingo Lütkebohle, 和 Björn Brandenburg: "基于预留调度下的ROS 2处理链的响应时间分析"，《第31届ECRTS会议论文集》，德国斯图加特，2019年7月。