# 接口（Interfaces）

## 目录

- 背景  
- 消息（Messages）  
  - 字段（Fields）  
    - 字段类型（Field types）  
    - 字段名称（Field names）  
    - 字段默认值（Field default value）  
  - 常量（Constants）  
- 服务（Services）  
- 动作（Actions）  

---

## 背景

ROS 应用通常通过三种类型的接口进行通信：**话题**（topics）、**服务**（services）或**动作**（actions）。  
ROS 2 使用一种简化的描述语言——**接口定义语言**（Interface Definition Language, IDL）来描述这些接口。这种描述使得 ROS 工具能够自动为多种目标语言生成该接口类型的源代码。

本文档将介绍以下支持的类型：

- **msg**：`.msg` 文件是简单的文本文件，用于描述 ROS 消息的字段。它们用于为不同语言生成消息的源代码。
- **srv**：`.srv` 文件描述一个服务，由两部分组成：**请求**（request）和**响应**（response）。请求和响应均为消息声明。
- **action**：`.action` 文件描述一个动作，由三部分组成：**目标**（goal）、**结果**（result）和**反馈**（feedback）。每一部分本身都是一个消息声明。

---

## 消息（Messages）

消息是 ROS 2 节点在**无响应预期**的情况下向网络中其他 ROS 节点发送数据的方式。  
例如，若一个 ROS 2 节点从传感器读取温度数据，它可以使用 `Temperature` 消息将该数据发布到 ROS 2 网络上。网络中的其他节点可订阅该数据并接收 `Temperature` 消息。

消息在 ROS 包的 `msg/` 目录中通过 `.msg` 文件进行描述和定义。`.msg` 文件包含两部分：**字段**（fields）和**常量**（constants）。

---

## 字段（Fields）

每个字段由一个**类型**和一个**名称**组成，中间用空格分隔，例如：

```
fieldtype1 fieldname1
fieldtype2 fieldname2
fieldtype3 fieldname3
```

示例：
```
int32 my_int
string my_string
```

### 字段类型（Field types）

字段类型可以是：

- **内建类型**（built-in type）
- **独立定义的消息类型名称**，例如 `geometry_msgs/PoseStamped`

当前支持的内建类型如下：

| 类型名 | C++ | Python | DDS 类型 |
|--------|-----|--------|---------|
| `bool` | `bool` | `builtins.bool` | `boolean` |
| `byte` | `uint8_t` | `builtins.bytes*` | `octet` |
| `char` | `char` | `builtins.int*` | `char` |
| `float32` | `float` | `builtins.float*` | `float` |
| `float64` | `double` | `builtins.float*` | `double` |
| `int8` | `int8_t` | `builtins.int*` | `octet` |
| `uint8` | `uint8_t` | `builtins.int*` | `octet` |
| `int16` | `int16_t` | `builtins.int*` | `short` |
| `uint16` | `uint16_t` | `builtins.int*` | `unsigned short` |
| `int32` | `int32_t` | `builtins.int*` | `long` |
| `uint32` | `uint32_t` | `builtins.int*` | `unsigned long` |
| `int64` | `int64_t` | `builtins.int*` | `long long` |
| `uint64` | `uint64_t` | `builtins.int*` | `unsigned long long` |
| `string` | `std::string` | `builtins.str` | `string` |
| `wstring` | `std::u16string` | `builtins.str` | `wstring` |

> (*) 所有比 ROS 定义更宽松的类型（如 Python 中的 `list` 或 `str`）**通过软件强制执行** ROS 规定的范围和长度限制。

所有内建类型均可用于定义数组：

| 类型 | C++ | Python | DDS 类型 |
|------|-----|--------|---------|
| 静态数组 | `std::array<T, N>` | `builtins.list*` | `T[N]` |
| 无界动态数组 | `std::vector` | `builtins.list` | `sequence` |
| 有界动态数组 | `custom_class<T, N>` | `builtins.list*` | `sequence<T, N>` |
| 有界字符串 | `std::string` | `builtins.str*` | `string` |

**消息定义示例**（使用数组和有界类型）：
```ros
int32[] unbounded_integer_array
int32[5] five_integers_array
int32[<=5] up_to_five_integers_array

string string_of_unbounded_size
string<=10 up_to_ten_characters_string

string[<=5] up_to_five_unbounded_strings
string<=10[] unbounded_array_of_strings_up_to_ten_characters_each
string<=10[<=5] up_to_five_strings_up_to_ten_characters_each
```

### 字段名称（Field names）

字段名称必须满足：
- 由小写字母、数字和下划线组成
- 必须以字母开头
- 不能以下划线结尾
- 不能包含两个连续的下划线

### 字段默认值（Field default value）

可为消息中的任意字段设置默认值。**当前不支持**为字符串数组和复杂类型（即非内建类型，包括嵌套消息）设置默认值。

通过在字段定义行中添加第三个元素来设置默认值：
```
fieldtype fieldname fielddefaultvalue
```

示例：
```
uint8 x 42
int16 y -2000
string full_name "John Doe"
int32[] samples [-200, -100, 0, 100, 200]
```

> **注意**：  
> - 字符串值必须使用单引号 `'` 或双引号 `"` 定义  
> - 当前**不支持**字符串转义

---

## 常量（Constants）

常量定义类似于带默认值的字段描述，但其值**在程序中不可更改**。使用等号 `=` 表示赋值：

```
constanttype CONSTANTNAME=constantvalue
```

示例：
```
int32 X=123
int32 Y=-123
string FOO="foo"
string EXAMPLE='bar'
```

> **注意**：常量名称必须为**大写字母**（UPPERCASE）

---

## 服务（Services）

服务是一种**请求/响应式通信**，客户端（请求方）等待服务端（响应方）执行一个**短时计算**并返回结果。

服务在 ROS 包的 `srv/` 目录中通过 `.srv` 文件定义。

服务描述文件由**请求消息类型**和**响应消息类型**组成，中间用 `---` 分隔。任意两个 `.msg` 文件用 `---` 连接即构成合法的服务描述。

**简单示例**（输入一个字符串，返回一个字符串）：
```ros
string str
---
string str
```

**复杂示例**：
```ros
# 请求常量
int8 FOO=1
int8 BAR=2
# 请求字段
int8 foobar
another_pkg/AnotherMessage msg
---
# 响应常量
uint32 SECRET=123456
# 响应字段
another_pkg/YetAnotherMessage val
CustomMessageDefinedInThisPackage value
uint32 an_integer
```

> **注意**：服务中**不能嵌套其他服务**。  
> 若引用同一包中的消息，**不得包含包名**。

---

## 动作（Actions）

动作是一种**长时间运行的请求/响应通信**。动作客户端（请求方）等待动作服务器（响应方）执行某项操作并返回结果。与服务不同，动作具有以下特点：
- 可长时间运行（数秒至数分钟）
- 执行过程中可提供**反馈**（feedback）
- 可被**中断**

动作定义格式如下：
```
<request_type> <request_fieldname>
---
<response_type> <response_fieldname>
---
<feedback_type> <feedback_fieldname>
```

- 第一个 `---` 之前为**请求字段**
- 第一个与第二个 `---` 之间为**响应字段**
- 第二个 `---` 之后为**反馈字段**

请求、响应和反馈字段的数量均可为零或任意多个。

各部分的类型和字段名遵循与消息相同的规则。

**示例**：Fibonacci 动作定义
```ros
int32 order
---
int32[] sequence
---
int32[] sequence
```

- 客户端发送一个 `int32` 字段 `order`，表示要计算的斐波那契数列项数
- 服务器返回一个 `int32[]` 数组 `sequence`，包含完整的数列
- 执行过程中，服务器可发送中间结果（已计算的部分数列）作为反馈