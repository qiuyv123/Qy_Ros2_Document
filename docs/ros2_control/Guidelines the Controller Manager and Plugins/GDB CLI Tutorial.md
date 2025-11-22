# GDB 快速参考手册（GDB Quick Reference）  
**GDB 版本 4**

> © 1991, 1992, 1993, 1998 自由软件基金会（Free Software Foundation, Inc.）  
> 本卡可依据 GNU 通用公共许可证（GPL）自由分发。  
> 作者不对本卡中的任何错误负责。

---

## 启动 GDB

| 命令 | 说明 |
|------|------|
| `gdb` | 启动 GDB，不加载调试文件 |
| `gdb program` | 开始调试 `program` |
| `gdb program core` | 使用 `core` 转储文件调试由 `program` 产生的崩溃 |
| `gdb --help` | 显示命令行选项说明 |

---

## 基本操作

| 命令 | 说明 |
|------|------|
| `quit`（或 `q`、`Ctrl-D`） | 退出 GDB |
| `Ctrl-C` | 中断当前命令，或向运行中的进程发送中断信号 |

---

## 获取帮助

| 命令 | 说明 |
|------|------|
| `help` | 列出命令类别 |
| `help class` | 显示某类别下所有命令的简要描述 |
| `help command` | 显示具体命令的详细说明 |

---

## 执行程序

| 命令 | 说明 |
|------|------|
| `run arglist` | 用 `arglist` 启动程序 |
| `run` | 用当前参数列表启动程序 |
| `run < in > out` | 启动程序并重定向输入/输出 |
| `kill` | 终止正在运行的程序 |
| `tty dev` | 下次运行时使用 `dev` 作为标准输入/输出设备 |
| `set args arglist` | 为下次运行设置参数列表 |
| `set args` | 清空参数列表 |
| `show args` | 显示当前参数列表 |
| `show env` | 显示所有环境变量 |
| `show env var` | 显示环境变量 `var` 的值 |
| `set env var string` | 设置环境变量 `var` 为 `string` |
| `unset env var` | 从环境中移除变量 `var` |

---

## Shell 命令

| 命令 | 说明 |
|------|------|
| `cd dir` | 切换工作目录 |
| `pwd` | 显示当前工作目录 |
| `make ...` | 调用 `make` 命令 |
| `shell cmd` | 执行任意 shell 命令 |

> **说明**：`[ ]` 表示可选参数，`...` 表示一个或多个参数。

---

## 断点与观察点（Breakpoints & Watchpoints）

| 命令 | 说明 |
|------|------|
| `break [file:]line`<br>`b [file:]line` | 在文件的某行设置断点（如 `break main.c:37`） |
| `break [file:]function`<br>`b [file:]function` | 在函数入口设置断点 |
| `break +offset` / `break -offset` | 在当前位置偏移若干行处设断点 |
| `break *addr` | 在地址 `addr` 处设断点 |
| `break` | 在下一条指令处设断点 |
| `break ... if expr` | 当表达式 `expr` 非零时才触发断点 |
| `cond n [expr]` | 为断点 `n` 设置新条件（无 `expr` 则取消条件） |
| `tbreak ...` | 设置**临时断点**（命中后自动删除） |
| `rbreak regex` | 为所有匹配正则表达式的函数设置断点 |
| `watch expr` | 为表达式 `expr` 设置**观察点**（值变化时中断） |
| `catch event` | 在特定事件中断（如 `throw`, `fork`, `load` 等） |
| `info break` | 显示所有已定义断点 |
| `info watch` | 显示所有观察点 |
| `clear` | 删除下一条指令处的断点 |
| `clear [file:]func` | 删除函数入口处的断点 |
| `clear [file:]line` | 删除某行上的断点 |
| `delete [n]` | 删除断点（或编号为 `n` 的断点） |
| `disable [n]` | 禁用断点 |
| `enable [n]` | 启用断点 |
| `enable once [n]` | 启用断点，命中后自动禁用 |
| `enable del [n]` | 启用断点，命中后自动删除 |
| `ignore n count` | 忽略断点 `n` 共 `count` 次 |
| `commands n`<br>`[silent]`<br>`command-list`<br>`end` | 每次命中断点 `n` 时执行指定命令列表（`silent` 抑制默认输出） |

---

## 程序调用栈（Program Stack）

| 命令 | 说明 |
|------|------|
| `backtrace [n]`<br>`bt [n]` | 打印调用栈（`n>0`：显示最内层 `n` 帧；`n<0`：显示最外层 `|n|` 帧） |
| `frame [n]` | 切换到第 `n` 帧（或地址 `n` 处的帧）；无参数则显示当前帧 |
| `up n` | 向上调 `n` 帧 |
| `down n` | 向下调 `n` 帧 |
| `info frame [addr]` | 显示当前帧（或地址 `addr` 处帧）的详细信息 |
| `info args` | 显示当前帧的函数参数 |
| `info locals` | 显示当前帧的局部变量 |
| `info reg [rn]`<br>`info all-reg [rn]` | 显示寄存器值（`all-reg` 包含浮点寄存器） |

---

## 执行控制（Execution Control）

| 命令 | 说明 |
|------|------|
| `continue [count]`<br>`c [count]` | 继续运行（若指定 `count`，则忽略当前断点 `count` 次） |
| `step [count]`<br>`s [count]` | 单步执行（进入函数），可重复 `count` 次 |
| `stepi [count]`<br>`si [count]` | 按机器指令单步（进入函数） |
| `next [count]`<br>`n [count]` | 单步执行（**不进入**函数） |
| `nexti [count]`<br>`ni [count]` | 按机器指令单步（不进入函数） |
| `until [location]` | 运行至下一行（或指定位置） |
| `finish` | 运行至当前函数返回 |
| `return [expr]` | 弹出当前栈帧，不执行剩余代码（可指定返回值） |
| `signal num` | 以信号 `num` 恢复执行（`0` 表示无信号） |
| `jump line`<br>`jump *address` | 跳转到指定行或地址继续执行 |
| `set var=expr` | 计算表达式并赋值给变量（用于修改程序变量） |

---

## 显示与查看（Display）

| 命令 | 说明 |
|------|------|
| `print [/f] expr`<br>`p [/f] expr` | 按格式 `f` 显示表达式值：<br>`x`：十六进制<br>`d`：有符号十进制<br>`u`：无符号十进制<br>`o`：八进制<br>`t`：二进制<br>`a`：地址<br>`c`：字符<br>`f`：浮点数 |
| `call [/f] expr` | 调用函数（类似 `print`，但不显示 `void` 返回值） |
| `x [/Nuf] addr` | 查看内存内容：<br>`N`：显示单元数量<br>`u`：单元大小（`b`=字节，`h`=半字，`w`=字，`g`=巨字）<br>`f`：显示格式（同 `print`，或 `s`=字符串，`i`=指令） |
| `disassem [addr]` | 将内存反汇编为机器指令 |
| `display [/f] expr` | 每次程序暂停时自动显示表达式值 |
| `display` | 显示所有自动显示项 |
| `undisplay n` | 从自动显示列表中移除编号 `n` 的项 |
| `disable disp n` | 禁用自动显示项 `n` |
| `enable disp n` | 启用自动显示项 `n` |
| `info display` | 列出所有自动显示项及其编号 |

---

## 表达式（Expressions）

- 支持 C、C++、Modula-2 语法（包括函数调用）
- `addr @ len`：从地址 `addr` 开始的 `len` 个元素数组
- `file::nm`：文件 `file` 中定义的变量或函数 `nm`
- `{type}addr`：将地址 `addr` 按指定类型 `type` 解释
- `$`：最近一次显示的值
- `$n`：第 `n` 次显示的值
- `$$`：`$` 的前一个值
- `$n`（在 `x` 命令后）：最后检查的地址
- `$value`：地址 `$` 处的值
- `$var`：便利变量（可赋任意值）
- `show values [n]`：显示最近 10 个值（或围绕 `$n` 的值）
- `show conv`：显示所有便利变量

---

## 符号表（Symbol Table）

| 命令 | 说明 |
|------|------|
| `info address s` | 显示符号 `s` 的存储位置 |
| `info func [regex]` | 显示所有函数（或匹配正则的函数）名称与类型 |
| `info var [regex]` | 显示所有全局变量（或匹配正则的变量）名称与类型 |
| `whatis [expr]` | 显示表达式的数据类型（不求值） |
| `ptype [expr]` | 显示表达式的详细类型信息（比 `whatis` 更详细） |
| `ptype type` | 描述结构体、联合、枚举等类型的定义 |

---

## GDB 脚本（GDB Scripts）

| 命令 | 说明 |
|------|------|
| `source script` | 从文件读取并执行 GDB 命令 |
| `define cmd`<br>`command-list`<br>`end` | 定义新 GDB 命令 `cmd` |
| `document cmd`<br>`help-text`<br>`end` | 为新命令 `cmd` 编写帮助文档 |

---

## 信号处理（Signals）

| 命令 | 说明 |
|------|------|
| `handle signal act` | 设置对信号的处理方式：<br>`print`/`noprint`：是否报告<br>`stop`/`nostop`：是否中断<br>`pass`/`nopass`：是否传递给程序 |
| `info signals` | 显示所有信号及其 GDB 处理方式 |

---

## 调试目标（Debugging Targets）

| 命令 | 说明 |
|------|------|
| `target type param` | 连接到目标机器、进程或文件 |
| `help target` | 显示可用目标类型 |
| `attach param` | 附加到另一个进程 |
| `detach` | 从目标进程分离 |

---

## 控制 GDB 自身（Controlling GDB）

| 命令 | 说明 |
|------|------|
| `set param value` | 设置 GDB 内部参数 |
| `show param` | 显示参数当前值 |

### 常用参数：
- `complaints`：限制非常规符号的警告数量
- `confirm`：启用/禁用确认提示
- `editing`：启用/禁用命令行编辑
- `height`：设置显示暂停的行数
- `language`：设置表达式语言（`auto`, `c`, `modula-2`）
- `listsize`：`list` 命令显示的行数
- `prompt`：设置 GDB 提示符
- `radix`：数字显示进制（8/10/16）
- `verbose`：加载符号时是否显示详细信息
- `width`：行折叠的字符数
- `write`：是否允许修改二进制或 core 文件

### 历史与打印选项：
- `history ...`：控制命令历史（文件、大小、保存等）
- `print ...`：控制输出格式（地址、数组、C++ 符号、联合体等）

| 其他命令 | 说明 |
|--------|------|
| `show commands` | 显示最近 10 条命令 |
| `show commands n` | 显示围绕第 `n` 条的 10 条命令 |
| `show commands +` | 显示下 10 条命令 |

---

## 工作文件（Working Files）

| 命令 | 说明 |
|------|------|
| `file [file]` | 设置可执行文件和符号表（无参数则清空） |
| `core [file]` | 加载 core 转储文件 |
| `exec [file]` | 仅设置可执行文件 |
| `symbol [file]` | 仅设置符号表 |
| `load file` | 动态链接文件并添加其符号 |
| `add-sym file addr` | 从动态加载到 `addr` 的文件中加载额外符号 |
| `info files` | 显示当前使用的工作文件和目标 |
| `path dirs` | 将 `dirs` 添加到可执行文件和符号文件搜索路径开头 |
| `show path` | 显示当前搜索路径 |
| `info share` | 列出当前加载的共享库 |

---

## 源文件（Source Files）

| 命令 | 说明 |
|------|------|
| `dir names` | 将目录 `names` 添加到源码搜索路径开头 |
| `dir clear` | 清空源码路径 |
| `show dir` | 显示当前源码路径 |
| `list` | 显示接下来 10 行源码 |
| `list -` | 显示前 10 行 |
| `list lines` | 显示指定位置周围的源码：<br>`[file:]num`：行号<br>`[file:]function`：函数起始<br>`+off` / `-off`：相对偏移<br>`*address`：包含该地址的行 |
| `list f,l` | 显示从行 `f` 到行 `l` 的源码 |
| `info line num` | 显示源码行 `num` 对应的编译后地址范围 |
| `info source` | 显示当前源文件名 |
| `info sources` | 列出所有已使用的源文件 |
| `forw regex` | 向后搜索匹配正则的源码行 |
| `rev regex` | 向前搜索匹配正则的源码行 |

---

## 在 GNU Emacs 中使用 GDB

| 命令 | 说明 |
|------|------|
| `M-x gdb` | 在 Emacs 中启动 GDB |
| `C-h m` | 显示 GDB 模式帮助 |
| `M-s` | 单步（`step`） |
| `M-n` | 下一行（`next`） |
| `M-i` | 单条指令（`stepi`） |
| `C-c C-f` | 完成当前栈帧（`finish`） |
| `M-c` | 继续（`continue`） |
| `M-u` | 向上 `arg` 帧（`up`） |
| `M-d` | 向下 `arg` 帧（`down`） |
| `C-x &` | 复制光标处数字并插入到命令末尾 |
| `C-x SPC` | 在源码文件光标处设置断点 |

---

## 许可与担保

- `show copying`：显示 GNU GPL 许可证全文  
- `show warranty`：显示“**GDB 不提供任何担保**”声明

> GDB 是自由软件，欢迎在 GNU GPL 条款下分发。  
> 对 GDB 有任何改进建议，请发送至：`bug-gdb@gnu.org`