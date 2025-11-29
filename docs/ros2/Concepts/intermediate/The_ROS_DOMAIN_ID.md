# The ROS_DOMAIN_ID

## 概述 (Overview)

如在其他地方所解释的，ROS 2 用于通信的默认中间件是 DDS。在 DDS 中，让不同的逻辑网络共享物理网络的主要机制称为 **Domain ID（域 ID）**。处于同一域（Domain）上的 ROS 2 节点可以自由发现彼此并发送消息，而处于不同域上的 ROS 2 节点则不能。默认情况下，所有 ROS 2 节点都使用 Domain ID 0。为了避免同一网络上运行 ROS 2 的不同计算机组之间产生干扰，应为每个组设置不同的 Domain ID。

## 选择 Domain ID（简短版）

下文解释了 ROS 2 中应使用的 Domain ID 范围的推导过程。若想跳过背景知识直接选择一个安全的数字，只需选择一个 **0 到 101（含）** 之间的 Domain ID 即可。

## 选择 Domain ID（详细版）

DDS 使用 Domain ID 来计算将用于发现和通信的 UDP 端口。有关端口计算的详细信息，请参阅 [这篇文章](https://www.google.com/search?q=https://community.rti.com/content/forum-topic/statically-configure-firewall-dds)。回顾一下网络基础知识，UDP 端口是一个 **无符号 16 位整数**。因此，可以分配的最大端口号是 65535。根据上述文章中的公式进行计算，这意味着可以分配的最高 Domain ID 是 232，而最低可以分配的是 0。

### 平台特定的约束 (Platform-specific constraints)

为了获得最大的兼容性，在选择 Domain ID 时应遵循一些额外的平台特定约束。特别是，最好避免在操作系统的 **临时端口范围 (ephemeral port range)** 内分配 Domain ID。这可以避免 ROS 2 节点使用的端口与计算机上其他网络服务使用的端口发生冲突。

以下是关于临时端口的一些平台特定说明：

\=== "Linux"

```
默认情况下，Linux 内核使用端口 **32768-60999** 作为临时端口。这意味着 **Domain ID 0-101** 和 **215-232** 可以安全使用，而不会与临时端口冲突。

在 Linux 中，可以通过在 `/proc/sys/net/ipv4/ip_local_port_range` 中设置自定义值来配置临时端口范围。如果使用了自定义的临时端口范围，上述数字可能需要相应调整。
```

\=== "macOS"

```
macOS 同样有其特定的临时端口范围（通常遵循 IANA 建议或 BSD 风格），建议遵循通用的安全范围。
```

\=== "Windows"

```
Windows 也有其默认的临时端口范围。建议遵循通用的安全范围。
```

### 参与者约束 (Participant constraints)

对于计算机上运行的每个 ROS 2 进程，都会创建一个 DDS“参与者 (participant)”。由于每个 DDS 参与者要在计算机上占用**两个端口**，因此在一台计算机上运行超过 120 个 ROS 2 进程可能会溢出到其他 Domain ID 或临时端口中。

为了理解原因，请考虑 Domain ID 1 和 2：

  * **Domain ID 1** 使用端口 7650 和 7651 进行多播 (multicast)。
  * **Domain ID 2** 使用端口 7900 和 7901 进行多播。

当在 Domain ID 1 中创建**第 1 个进程**（第 0 个参与者）时，端口 7660 和 7661 用于单播 (unicast)。
当在 Domain ID 1 中创建**第 120 个进程**（第 119 个参与者）时，端口 7898 和 7899 用于单播。
当在 Domain ID 1 中创建**第 121 个进程**（第 120 个参与者）时，端口 **7900 和 7901** 用于单播，这与 **Domain ID 2 重叠**。

如果已知计算机一次只会处于一个 Domain ID 上，并且 Domain ID 足够低，那么创建比这更多的 ROS 2 进程是安全的。

当选择一个接近平台特定 Domain ID 范围上限的 ID 时，还应考虑一个额外的约束。

例如，假设一台 Linux 计算机的 Domain ID 为 101：

  * 计算机上的第 0 个 ROS 2 进程将连接到端口 32650, 32651, 32660, 和 32661。
  * 计算机上的第 1 个 ROS 2 进程将连接到端口 32650, 32651, 32662, 和 32663。
  * ...
  * 计算机上的第 53 个 ROS 2 进程将连接到端口 32650, 32651, 32766, 和 32767。
  * 计算机上的第 54 个 ROS 2 进程将连接到端口 32650, 32651, **32768 和 32769**，这闯入了**临时端口范围**。

因此，在 Linux 上使用 Domain ID 101 时，应创建的最大进程数为 54。同样，在 Linux 上使用 Domain ID 232 时，应创建的最大进程数为 63，因为最大端口号为 65535。

macOS 和 Windows 上的情况类似，只是数字不同。在 macOS 和 Windows 上，当选择 166（范围的上限）作为 Domain ID 时，在闯入临时端口范围之前，计算机上可以创建的最大 ROS 2 进程数为 120。

## Domain ID 到 UDP 端口计算器
<div style="background-color: #f5f5f5; padding: 20px; border-radius: 5px; border: 1px solid #ddd;"> <form id="dds-calculator"> <div style="margin-bottom: 10px;"> <label style="display: inline-block; width: 150px; font-weight: bold;">Domain ID:</label> <input type="number" id="domain_id" value="0" min="0" max="232" oninput="calculatePorts()" style="padding: 5px;"> </div> <div style="margin-bottom: 10px;"> <label style="display: inline-block; width: 150px; font-weight: bold;">Participant ID:</label> <input type="number" id="participant_id" value="0" min="0" oninput="calculatePorts()" style="padding: 5px;"> </div> <hr style="margin: 20px 0;"> <div style="margin-bottom: 5px;"> <span style="display: inline-block; width: 200px;">Discovery Multicast Port:</span> <code id="disco_multi" style="font-weight: bold; color: #e91e63;">7400</code> </div> <div style="margin-bottom: 5px;"> <span style="display: inline-block; width: 200px;">User Multicast Port:</span> <code id="user_multi" style="font-weight: bold; color: #e91e63;">7401</code> </div> <div style="margin-bottom: 5px;"> <span style="display: inline-block; width: 200px;">Discovery Unicast Port:</span> <code id="disco_uni" style="font-weight: bold; color: #2196f3;">7410</code> </div> <div style="margin-bottom: 5px;"> <span style="display: inline-block; width: 200px;">User Unicast Port:</span> <code id="user_uni" style="font-weight: bold; color: #2196f3;">7411</code> </div> </form> </div>

<script> function calculatePorts() { // DDS RTPS Port Calculation Constants const PB = 7400; // Port Base const DG = 250; // Domain Gain const PG = 2; // Participant Gain const d0 = 0; // Discovery Multicast Offset const d1 = 1; // User Multicast Offset const d2 = 10; // Discovery Unicast Offset const d3 = 11; // User Unicast Offset

} </script>