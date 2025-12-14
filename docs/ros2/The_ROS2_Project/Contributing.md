# ROS 2 开发者指南

**目录**

  * 通用原则
  * 质量实践
  * 版本控制
  * 变更控制流程
  * 文档
  * 测试
  * 通用实践
  * 问题 (Issues)
  * 分支
  * 合并请求 (Pull requests)
  * 库版本控制
  * 开发流程
  * RMW API 的变更
  * 任务追踪
  * 包命名约定
  * 度量单位和坐标系约定
  * 编程约定
  * 文件系统布局
  * 上游包
  * 开发者工作流
  * Gitconfig 优化
  * 架构开发实践
  * 软件开发生命周期
  * 构建农场 (Build Farm) 介绍
  * 关于覆盖率运行的说明
  * 如何从构建农场报告中读取覆盖率
  * 如何从构建农场报告中计算覆盖率
  * 如何在本地使用 lcov 测量覆盖率 (Ubuntu)

本页定义了我们在开发 ROS 2 时采用的实践和策略。

-----

## 通用原则

所有 ROS 2 开发都有一些共同的原则：

  * **共享所有权 (Shared ownership)**：每一位致力于 ROS 2 的人都应该感觉到对系统所有部分的所有权。代码块的原始作者没有任何特殊的许可或义务来控制或维护该代码块。每个人都可以自由地提议任何地方的变更，处理任何类型的工单，并审查任何合并请求。
  * **乐于从事任何工作**：作为共享所有权的推论，每个人都应该愿意承担任何可用的任务，并为系统的任何方面做出贡献。
  * **寻求帮助**：如果你在某件事上遇到困难，请通过工单、评论或电子邮件（视情况而定）向你的开发伙伴寻求帮助。

-----

## 质量实践

根据 [REP 2004: Package Quality Categories](https://github.com/ros-infrastructure/rep/blob/master/rep-2004.rst) 中的指导原则，基于包所遵循的开发实践，包可以被归类为不同的质量等级。这些类别通过其在版本控制、测试、文档等方面的策略来区分。

以下部分是我们为确保核心包具有最高质量（“1 级”）而遵循的具体开发规则。我们建议所有 ROS 开发人员努力遵守以下策略，以确整个 ROS 生态系统的质量。

有关更具体的代码建议，请参阅[质量指南](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Quality-Guide.html)。

### 版本控制

我们将使用 **语义化版本控制 (Semantic Versioning/semver)** 指导原则进行版本控制。

我们还将遵守在 semver 完整含义之上建立的一些 ROS 特定规则：

  * 主要版本增量（即破坏性变更）不应在已发布的 ROS 发行版中进行。
  * 补丁（保持接口不变）和次要（非破坏性）版本增量不会破坏兼容性，因此这类变更**允许**在发行版中进行。
  * 主要的 ROS 发布版本是发布破坏性变更的最佳时机。如果核心包需要多个破坏性变更，它们应该合并到其集成分支（例如 `rolling`）中，以便在 CI 中快速发现问题，但要一起发布以减少 ROS 用户主要版本的发布数量。
  * 虽然主要增量需要新的发行版，但新的发行版并不一定需要主要版本号的提升（如果开发和发布可以在不破坏 API 的情况下进行）。
  * 对于编译代码，ABI 被视为公共接口的一部分。任何需要重新编译依赖代码的变更都被视为主要（破坏性）变更。
  * ABI 破坏性变更**可以**在发行版发布**之前**的次要版本提升中进行（添加到滚动发布中）。
  * 我们在 Dashing 和 Eloquent 中强制执行核心包的 API 稳定性，即使它们的主要版本部分为 `0`，这与 SemVer 关于初始开发的规范不同。
  * 随后，包应努力达到成熟状态并升级到 `1.0.0` 版本，以符合 semver 的规范。

#### 注意事项

这些规则是**尽力而为**的。在极少数极端情况下，可能需要在主要版本/发行版内破坏 API。意外的破坏是否会导致主要或次要版本的增加将视具体情况而定。

例如，考虑一种情况，涉及已发布的 X-turtle（对应于主要版本 `1.0.0`）和已发布的 Y-turtle（对应于主要版本 `2.0.0`）。

  * 如果在 X-turtle 中发现绝对必要的 API 破坏性修复，显然不能选择升级到 `2.0.0`，因为 `2.0.0` 已经存在。
  * 在这种情况下处理 X-turtle 版本的方法（都不是理想的）有：
    1.  提升 X-turtle 的次要版本：不理想，因为它违反了 SemVer 的原则，即破坏性变更必须提升主要版本。
    2.  将 X-turtle 的主要版本提升到超过 Y-turtle（至 `3.0.0`）：不理想，因为旧发行版的版本将高于新发行版中已可用的版本，这将使特定于版本的条件代码失效/崩溃。

开发人员必须决定使用哪种解决方案，或者更重要的是，他们愿意打破哪条原则。我们不能建议哪一种，但在任何一种情况下，我们都要求采取明确措施，手动向用户传达这种中断及其解释（不仅仅是通过版本增量）。

如果没有 Y-turtle，即使修复在技术上只是一个补丁，X-turtle 也必须升级到 `2.0.0`。这种情况符合 SemVer，但违反了我们自己的规则，即不应在已发布的发行版中引入主要增量。

这就是我们将版本控制规则视为**尽力而为**的原因。尽管上述示例不太可能发生，但准确定义我们的版本控制系统非常重要。

#### 公共 API 声明

根据 semver，每个包必须清楚地声明公共 API。我们将使用包质量声明中的“公共 API 声明”部分来声明哪些符号属于公共 API。

对于大多数 C 和 C++ 包，声明就是它安装的任何头文件。但是，定义一组被视为私有的符号是可以接受的。避免在头文件中使用私有符号有助于 ABI 稳定性，但并非必须。

对于 Python 等其他语言，必须明确定义公共 API，以便清楚在版本控制准则方面可以依赖哪些符号。公共 API 还可以扩展到构建产物，如配置变量、CMake 配置文件等，以及可执行文件、命令行选项和输出。公共 API 的任何元素都应在包的文档中明确说明。如果您使用的内容未在包文档中明确列为公共 API 的一部分，那么您不能依赖它在次要版本或补丁版本之间保持不变。

#### 弃用策略

在可能的情况下，我们还将对主要版本增量使用 **Tick-Tock** 弃用和迁移策略。新的弃用将出现在新的发行版发布中，并伴随编译器警告，表示该功能已被弃用。在下一个版本中，该功能将被完全移除（无警告）。

函数 `foo` 被弃用并由函数 `bar` 替换的示例：

| 版本 | API |
| :--- | :--- |
| X-turtle | `void foo();` |
| Y-turtle | `[[deprecated(“use bar()”)]] void foo();` <br> `void bar();` |
| Z-turtle | `void bar();` |

我们不得在发行版发布后添加弃用。不过，弃用并不一定需要主要版本提升。如果在发行版发布之前进行版本提升，则可以在次要版本提升中引入弃用（类似于 ABI 破坏性变更）。

例如，如果 X-turtle 作为 `2.0.0` 开始开发，则可以在 X-turtle 发布之前在 `2.1.0` 中添加弃用。

我们将尝试尽可能保持跨发行版的兼容性。然而，就像与 SemVer 相关的注意事项一样，在某些情况下可能无法完全遵守 Tick-Tock 甚至一般的弃用策略。

### 变更控制流程

  * 所有变更都必须通过合并请求 (Pull Request) 进行。
  * 我们将在 ROSCore 仓库的合并请求中强制执行 **开发者原产地证书 (DCO)**。
      * 它要求所有提交消息包含 `Signed-off-by` 行，其中的电子邮件地址需与提交作者匹配。
      * 您可以向 `git commit` 调用传递 `-s` / `--signoff`，或手动编写预期的消息（例如 `Signed-off-by: Your Name Developer <your.name@example.com>`）。
      * 对于仅涉及空白移除、拼写错误更正和其他**微不足道的变更**的合并请求，**不**要求 DCO。
  * 始终为每个合并请求运行所有**一级平台**的 CI 任务，并在合并请求中包含任务链接。（如果您没有访问 Jenkins 任务的权限，会有人为您触发任务。）
  * 至少需要 1 位非合并请求作者的开发伙伴批准，才能视为已批准。合并前必须获得批准。
      * 包可以选择增加此数量。
  * 任何所需的文档变更（API 文档、功能文档、发行说明等）必须在合并相关变更之前提出。

#### 向后移植 (Backporting) PR 的指南

更改较旧版本的 ROS 时：

1.  在打开 PR 以将更改向后移植到旧版本之前，请确保功能或修复已被接受并合并到 `rolling` 分支中。
2.  向后移植到旧版本时，也要考虑向后移植到任何其他**仍受支持的版本**，即使是非 LTS 版本。
3.  如果您要完整地向后移植单个 PR，请将向后移植 PR 命名为 `[Distro] <原 PR 名称>`。
4.  在向后移植 PR 的描述中链接到您从中移植更改的所有 PR。
5.  包维护者通常使用 **Mergify.io** 在需要时自动将 PR 向后移植到下游发行版，但在必要时，开发人员仍然可以按照上述说明执行手动向后移植操作。

### 文档

所有包均应在其 README 中包含这些文档元素或从其 README 链接到这些元素：

  * 描述和目的
  * 公共 API 的定义和描述
  * 示例
  * 如何构建和安装（应引用外部工具/工作流）
  * 如何构建和运行测试
  * 如何构建文档
  * 如何开发（用于描述诸如 `python setup.py develop` 之类的内容）

#### 许可证和版权声明

  * 每个源文件必须有许可证和版权声明，并使用自动 linter 进行检查。
  * 每个包必须有一个 `LICENSE` 文件，通常是 Apache 2.0 许可证，除非该包已有宽松的许可证（例如 rviz 使用三条款 BSD）。

#### 包描述

  * 每个包都应该描述自身及其目的，尽可能假设读者是在没有 ROS 或其他相关项目先验知识的情况下偶然发现它的。
  * 每个包都应该定义和描述其公共 API，以便用户对语义化版本控制策略涵盖的内容有合理的预期。即使在 C 和 C++ 中（可以通过 API 和 ABI 检查强制执行公共 API），描述代码布局和代码各部分的功能也是一个好机会。
  * 应该能够轻松获取任何包，并从该包的文档中了解如何构建、运行、构建和运行测试以及构建文档。显然，对于常见的工作流（如在工作区中构建包），我们应该避免重复，但基本工作流应该被描述或引用。
  * 最后，它应该包括针对开发人员的任何文档。这可能包括使用类似 `python setup.py develop` 测试代码的工作流，或者可能意味着描述如何利用您的包提供的扩展点。

**示例：**

  * **capabilities**: 这个包给出了描述公共 API 的文档示例。
  * **catkin\_tools**: 这是一个描述包扩展点的示例。

#### ROS 包的 API 文档

所有已发布 ROS 包的 API 文档可以在 [这里](https://docs.ros.org/) 找到。我们建议使用 [index.ros.org](https://index.ros.org/) 搜索可用的 ROS 包以查找其文档。

如果您是 ROS 包开发人员，正在寻找有关记录包的指导，请参阅我们关于[包级别文档](https://www.google.com/search?q=https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Quality-Guide.html%23documentation)的“操作方法”指南。所有已发布的 ROS 2 包的文档都会自动托管在 [docs.ros.org](https://docs.ros.org/) 上。

### 测试

所有包都应该具有一定程度的**系统**、**集成**和/或**单元测试**。

  * **单元测试**应始终位于被测试的包中，并应利用 Mock 等工具尝试在构建的场景中测试代码库的狭窄部分。单元测试不应引入非测试工具的测试依赖项，例如 gtest、nosetest、pytest、mock 等...
  * **集成测试**可以测试代码各部分之间或代码各部分与系统之间的交互。它们通常以我们期望用户使用软件接口的方式来测试这些接口。与单元测试一样，集成测试应位于被测试的包中，除非绝对必要，否则不应引入非工具测试依赖项，即所有非工具依赖项仅在极端审查下才被允许，因此应尽可能避免。
  * **系统测试**旨在测试包之间的端到端情况，并应位于其自己的包中，以避免膨胀或耦合包，并避免循环依赖。
      * 通常，应尽量减少外部或跨包测试依赖项，以防止循环依赖和紧密耦合的测试包。

所有包都应该有一些单元测试，可能还有集成测试，但拥有的程度取决于包的质量类别。以下小节适用于“1 级”包：

#### 代码覆盖率

我们将提供行覆盖率，并达到 95% 以上的行覆盖率。如果较低的百分比目标是合理的，则必须显着地记录下来。我们可以提供分支覆盖率，或从覆盖率中排除代码（测试代码、调试代码等）。我们要求在合并更改之前覆盖率增加或保持不变，但进行降低代码覆盖率的更改（例如删除以前覆盖的代码可能导致百分比下降）且有适当理由是可以接受的。

#### 性能

我们强烈建议进行性能测试，但也承认这对某些包没有意义。如果有性能测试，我们将选择在每次更改时检查，或在每次发布前检查，或两者兼而有之。如果合并更改或进行发布会降低性能，我们也需要理由。

#### Linters 和静态分析

我们将使用 [ROS 代码风格](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Code-Style-Language-Versions.html) 并使用来自 `ament_lint_common` 的 linter 强制执行。必须使用 `ament_lint_common` 中包含的所有 linter/静态分析。

`ament_lint_auto` 文档提供了有关运行 `ament_lint_common` 的信息。

-----

## 通用实践

有些实践是所有 ROS 2 开发共有的。
这些实践不影响 [REP 2004](https://github.com/ros-infrastructure/rep/blob/master/rep-2004.rst) 中描述的包质量级别，但仍强烈建议在开发过程中使用。

### 问题 (Issues)

提交问题时，请务必：

  * 包含足够的信息让其他人理解问题。在 ROS 2 中，缩小问题原因需要以下几点。在每个类别中测试尽可能多的替代方案将特别有帮助。
      * **操作系统和版本**。理由：ROS 2 支持多个平台，有些错误是特定于操作系统/编译器版本的。
      * **安装方法**。理由：有些问题仅在从二进制归档或 deb 安装 ROS 2 时才会出现。这有助于我们要确定问题是否出在打包过程中。
      * **ROS 2 的具体版本**。理由：有些错误可能存在于特定的 ROS 2 版本中，随后已修复。了解您的安装是否包含这些修复非常重要。
      * **正在使用的 DDS/RMW 实现**（请参阅[此页面](https://docs.ros.org/en/rolling/How-To-Guides/Working-with-multiple-RMW-implementations.html)了解如何确定）。理由：通信问题可能特定于所使用的底层 ROS 中间件。
      * **正在使用的 ROS 2 客户端库**。理由：这有助于我们缩小问题可能所在的堆栈层级。
  * 包含复现问题的步骤列表。
  * 如果是 bug，请考虑提供一个**简短、自包含、正确（可编译）的示例**。如果其他人可以轻松复现，问题更有可能得到解决。
  * 提及已经尝试过的故障排除步骤，包括：
      * 升级到最新版本的代码，其中可能包含尚未发布的错误修复。请参阅[此部分](https://www.google.com/search?q=%23branches)并按照说明获取“rolling”分支。
      * 尝试使用不同的 RMW 实现。请参阅[此页面](https://docs.ros.org/en/rolling/How-To-Guides/Working-with-multiple-RMW-implementations.html)了解如何操作。

### 分支

> **注意**
> 这些只是指导原则。包维护者可以根据自己的工作流程选择分支名称。

最好的做法是在包的源代码仓库中为它针对的每个 ROS 发行版建立**独立的分支**。这些分支通常以它们针对的发行版命名。例如，`humble` 分支专门用于针对 Humble 发行版的开发。

发布也是从这些分支进行的，针对相应的发行版。针对特定 ROS 发行版的开发可以在相应的分支上进行。例如：针对 `foxy` 的开发提交到 `foxy` 分支，`foxy` 的包发布也从该分支进行。

> **注意**
> 这要求包维护者根据需要执行向后移植或向前移植，以使所有分支的功能保持最新。维护者还必须对仍进行包发布的所有分支执行一般维护（错误修复等）。

例如，如果一个功能被合并到滚动特定分支（例如 `rolling` 或 `main`），并且该功能也适用于 Humble 发行版（不破坏 API 等），那么将该功能向后移植到 Humble 特定分支是一个好习惯。

如果有新功能或错误修复可用，维护者可以为那些较旧的发行版进行发布。

**关于 `main` 和 `rolling` 呢？**
`main` 通常针对 **Rolling**（即下一个未发布的 ROS 发行版），尽管维护者可能决定改为从 `rolling` 分支进行开发和发布。

### 合并请求 (Pull requests)

  * 一个合并请求应仅关注一个变更。单独的变更应放入单独的合并请求中。请参阅 [GitHub 关于编写完美合并请求的指南](https://github.blog/2015-01-21-how-to-write-the-perfect-pull-request/)。
  * 补丁的大小应最小化，并避免任何类型的不必要变更。
  * 合并请求必须包含最少数量的有意义的提交。
      * 您可以在合并请求审核期间创建新的提交。
      * 在合并合并请求之前，应将所有变更压缩（squash）为少量的语义提交，以保持历史记录清晰。
      * 但在合并请求审核期间**避免**压缩提交。您的审阅者可能不会注意到您进行了更改，从而导致潜在的混淆。另外，无论如何您都要在合并之前进行压缩；尽早这样做没有任何好处。
  * 欢迎任何开发人员审查和批准合并请求（请参阅[通用原则](https://www.google.com/search?q=%23general-principles)）。
  * 当您正在处理尚未准备好审查或合并的变更时，请使用**草稿合并请求**。当该变更准备好审查时，将合并请求移出草稿状态。请注意，如果您希望特定人员对草稿合并请求提供早期反馈，您可以在合并请求的描述或评论中 @ 提及他们。
  * 如果您的合并请求依赖于其他合并请求，请通过在合并请求描述的顶部添加 `- Depends on <link>` 来链接每个依赖的合并请求。这样做有助于审阅者了解合并请求的上下文。
  * 当您开始审查合并请求时，请在合并请求上发表评论，以便其他开发人员知道您正在审查它。
  * 合并请求审查不是只读的，即审阅者发表评论然后等待作者解决。作为审阅者，请随意就地进行小的改进（拼写错误、样式问题等）。作为合并请求的发起人，如果您在分叉（fork）中工作，勾选“允许上游贡献者编辑”框将有助于上述操作。作为审阅者，也可以随意进行更实质性的改进，但请考虑将它们放在单独的分支中（在评论中提及新分支，或从新分支向原始分支打开另一个合并请求）。
  * 只有维护者和提交者才能将批准的合并请求合并到主线。请参阅[当前的 ROS PMC 成员和提交者](https://docs.ros.org/en/rolling/The-ROS2-Project/Governance.html)，以获取具有合并权限的人员列表。

### 库版本控制

我们将一起对包内的所有库进行版本控制。这意味着库从包继承其版本。这可以防止库和包的版本发生偏离，并与“共享仓库的包一起发布”这一策略理由相同。如果您需要库具有不同的版本，请考虑将它们拆分为不同的包。

### 开发流程

  * 默认分支（大多数情况下是 rolling 分支）必须始终能够构建、通过所有测试且编译无警告。如果在任何时候出现回退，首要任务是至少恢复到之前的状态。
  * 始终在启用测试的情况下进行构建。
  * 始终在变更后和提议合并请求之前在本地运行测试。除了使用自动化测试外，还要手动运行修改后的代码路径，以确保补丁按预期工作。
  * 始终为每个合并请求运行所有平台的 CI 任务，并在合并请求中包含任务链接。
  * 有关推荐的软件开发工作流的更多详细信息，请参阅[软件开发生命周期](https://www.google.com/search?q=%23software-development-lifecycle)部分。

### RMW API 的变更

更新 **RMW API** 时，要求一级中间件库的 RMW 实现也必须更新。例如，RMW API 中引入的新函数 `rmw_foo()` 必须在以下包中实现（截至 ROS Galactic）：

  * rmw\_connextdds
  * rmw\_cyclonedds
  * rmw\_fastrtps

如果可行（例如取决于变更的大小），也应考虑更新非一级中间件库。有关中间件库及其层级的列表，请参阅 [REP-2000](https://github.com/ros-infrastructure/rep/blob/master/rep-2000.rst)。

### 任务追踪

为了帮助组织 ROS 2 上的工作，ROS 2 核心开发团队使用看板风格的 [GitHub 项目板](https://github.com/orgs/ros2/projects)。

但是，并非所有问题和合并请求都在项目板上进行跟踪。通常，一个板代表即将发布的版本或特定项目。可以通过浏览 [ROS 2 仓库](https://github.com/ros2)的单独 issue 页面，按仓库浏览工单。

任何给定 ROS 2 项目板中列的名称和用途各不相同，但通常遵循相同的一般结构：

  * **To do**：与项目相关的问题，准备分配
  * **In progress**：目前正在进行工作的活跃合并请求
  * **In review**：工作已完成并准备好审查的合并请求，以及当前正在积极审查的合并请求
  * **Done**：合并请求和相关问题已合并/关闭（用于信息目的）

要请求进行更改的权限，只需要在您感兴趣的工单上发表评论。根据复杂程度，描述您计划如何解决它可能会很有用。我们将更新状态（如果您没有权限），您可以开始处理合并请求。如果您定期贡献，我们可能会直接授予您自行管理标签等的权限。

-----

## 包命名约定

名称在 ROS 中起着重要作用，遵循命名约定可以简化学习和理解大型系统的过程。

  * ROS 包占据一个扁平的命名空间，因此应仔细且一致地进行命名。在 [REP-144](https://github.com/ros-infrastructure/rep/blob/master/rep-0144.rst) 中有一个包命名标准。
  * 包名称应遵循通用的 C 变量命名约定：小写，以字母开头，使用下划线分隔符，例如 `laser_viewer`。
  * 包名称应足够具体，以识别包的功能。例如，运动规划器不叫 `planner`。如果它实现了波前传播算法，它可能被称为 `wavefront_planner`。显然，在使名称具体化和避免使其过于冗长之间存在张力。
  * 应避免使用诸如 `utils` 之类的笼统名称，因为它们没有界定什么进入包或什么应该在包之外。
  * 要检查名称是否被占用，请查阅 [https://index.ros.org/packages/](https://index.ros.org/packages/)。如果您希望将您的仓库包含在该列表中，请参阅 [rosdistro 贡献指南](https://www.google.com/search?q=https://github.com/ros/rosdistro/blob/master/CONTRIBUTING.md)。
  * 我们的目标是开发一套规范的工具，让机器人做有趣的事情。包名应该告诉你包是做什么的，而不是它来自哪里。作为社区，我们应该能够做到这一点。Ubuntu 发行版提供了大约 33,000 个包，而没有在名称中插入来源或作者身份。
  * 仅当包不打算被更广泛地使用时，才建议给包名加前缀（例如，特定于 PR2 机器人的包使用 `pr2_` 前缀）。在分叉现有包时，您可能会给包名加前缀，但同样，前缀希望能传达发生了什么变化，而不是谁改变了它。
  * 对于 ROS 包，以 `ros` 为前缀是多余的。除非是非常核心的包，否则不建议这样做。

### 度量单位和坐标系约定

ROS 中使用的标准单位和坐标约定已在 [REP-0103](https://www.google.com/search?q=https://github.com/ros-infrastructure/rep/blob/master/rep-0103.rst) 中正式确定。除非有非常充分的理由并清楚地记录以避免混淆，否则所有消息都应遵循这些准则。

ROS 中距离测量内的特殊条件（如“太近”或“太远”）的表示已在 [REP-0117](https://github.com/ros-infrastructure/rep/blob/master/rep-0117.rst) 中正式确定。

### 编程约定

  * **防御性编程**：确保假设尽可能早地被验证。例如，检查每个返回码，并确保至少抛出异常，直到情况得到更优雅的处理。
  * 所有错误消息必须定向到 `stderr`。
  * 在尽可能窄的范围内声明变量。
  * 保持项目组（依赖项、导入、包含等）按字母顺序排列。

#### C++ 特定

  * 避免使用直接流 (`<<`) 到 `stdout` / `stderr`，以防止多个线程之间的交错。
  * 避免对 `std::shared_ptr` 使用引用，因为这会破坏引用计数。如果原始实例超出范围并且使用了引用，它将访问释放的内存。

### 文件系统布局

包和仓库的文件系统布局应遵循相同的约定，以便为浏览源代码的用户提供一致的体验。

#### 包布局

  * `src`: 包含所有 C 和 C++ 代码
      * 也包含未安装的 C/C++ 头文件
  * `include`: 包含所有已安装的 C 和 C++ 头文件
      * `<package_name>`: 对于所有 C 和 C++ 已安装的头文件，它们应该由包名进行文件夹命名空间隔离
  * `<package_name>`: 包含所有 Python 代码
  * `test`: 包含所有自动化测试和测试数据
  * `config`: 包含配置文件，例如 YAML 参数文件和 RViz 配置文件
  * `doc`: 包含所有文档
  * `launch`: 包含所有启动文件
  * `msg`: 包含所有 ROS 消息定义
  * `srv`: 包含所有 ROS 服务定义
  * `action`: 包含所有 ROS 动作定义
  * `package.xml`: 按照 [REP-0140](https://github.com/ros-infrastructure/rep/blob/master/rep-0140.rst) 定义（可能会为了原型设计而更新）
  * `CMakeLists.txt`: 仅限使用 CMake 的 ROS 包
  * `setup.py`: 仅限只使用 Python 代码的 ROS 包
  * `README`: 可以在 GitHub 上渲染为项目的着陆页
      * 这可以根据方便程度简短或详细，但它至少应该链接到项目文档
      * 考虑在这个 README 中放置 CI 或代码覆盖率标签
      * 也可以是 `.rst` 或 GitHub 支持的任何其他格式
  * `CONTRIBUTING`: 描述贡献指南
      * 这可能包括许可证含义，例如在使用 Apache 2 许可证时。
  * `LICENSE`: 此包的一个或多个许可证的副本
  * `CHANGELOG.rst`: 符合 [REP-0132](https://github.com/ros-infrastructure/rep/blob/master/rep-0132.rst) 的变更日志

#### 仓库布局

  * 每个包应该位于与其包名相同的子文件夹中。如果仓库仅包含单个包，则它可以选择位于仓库的根目录中。

-----

## 上游包

### Debian 和 Ubuntu 上游中的包

由于 Jochen Sprickerhof 和 Leopold Palomo-Avellaneda 的辛勤努力，现在可以从 Debian 和 Ubuntu 主仓库中[获取一些 ROS 2 包](https://www.google.com/search?q=https://packages.debian.org/search%3Fkeywords%3Dros2)。[这是 Jochen 在 ROSCon 2015 上的过程简短概述](https://www.google.com/search?q=https://roscon.ros.org/2015/presentations/ROS_on_Debian.pdf)。原始 ROS 包已被修改以遵循 Debian 指南，其中包括将包拆分为多个部分、在某些情况下更改名称、根据 FHS 指南安装到 `/usr` 以及在共享库上使用 soversion。

此外，一些引导依赖项，如命令行工具 `vcstool` 和 `colcon`，以及一些库如 `osrf-pycommon` 和 `ament` 也在上游打包。

与来自 [http://packages.ros.org](http://packages.ros.org) 的 OSRF 提供的 ROS 包不同，上游仓库中的包不依附于特定的 **ROS 发行版**。相反，它们代表了时间快照，将在 Debian unstable 中定期更新，然后锁定到下游 Debian 和 Ubuntu 发行版的各个点。

### 不要混合使用源 (Don’t mix the streams)

我们强烈建议不要在同一系统上混合使用来自上游 Debian/Ubuntu 的 ROS 包和来自 [http://packages.ros.org](http://packages.ros.org) 的 ROS 包。在某些情况下，这种混合系统可以正常工作，但两组包之间可能会产生负面交互。我们正在与 Jochen 及其朋友合作，通过文档和包冲突规范来尽量减少出现问题的机会，但我们预计仍会存在一些风险，包括一些相当微妙的问题。

因此，我们建议您选择从上游安装包**或**从 [http://packages.ros.org](http://packages.ros.org) 安装，但**不要**两者都选。不仅不应同时安装两者的包，而且如果您打算使用上游包，甚至不应在您的 apt 源中（即 `/etc/apt/sources*` 中的任何文件）包含 [http://packages.ros.org](http://packages.ros.org) 条目。同时启用它们可能会导致名称重叠的包在两个源之间混合，例如 `python3-rospkg`。

### 已知差异

与来自 packages.ros.org 的 ROS 包相比，上游 ROS 包存在一些人们应该注意的差异：

  * 包集不完整。
  * 包可能有不同的名称和不同的分区方式。

-----

## 开发者工作流

我们使用 [GitHub 项目板](https://github.com/orgs/ros2/projects)跟踪与即将发布的版本和较大型项目相关的未解决工单和活跃 PR。

通常的工作流是：

1.  讨论设计（在相应仓库上的 GitHub工单，如果需要，向 [https://github.com/ros2/design](https://github.com/ros2/design) 提交设计 PR）
2.  在分叉（fork）的功能分支上编写实现
      * 请查看[开发者指南](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Developer-Guide.html)以获取指南和最佳实践
3.  编写测试
4.  启用并运行 linter
5.  使用 `colcon test` 在本地运行测试（请参阅 [colcon 教程](https://colcon.readthedocs.io/en/released/user/quick-start.html)）
6.  一旦一切在本地构建无警告并且所有测试都通过，就在您的功能分支上运行 CI：
    1.  转到 [ci.ros2.org](https://ci.ros2.org/)
    2.  登录（右上角）
    3.  点击 `ci_launcher` 任务
    4.  点击“Build with Parameters”（左栏）
    5.  在第一个框“CI\_BRANCH\_TO\_TEST”中输入您的功能分支名称
    6.  点击 **build** 按钮
    7.  （如果您不是 ROS 2 提交者，您无权访问 CI 农场。在这种情况下，请联系您的 PR 审阅者为您运行 CI）
    8.  如果您的用例需要运行代码覆盖率：
          * 转到 [ci.ros2.org](https://ci.ros2.org/)
          * 登录（右上角）
          * 点击 `ci_linux_coverage` 任务
          * 点击“Build with Parameters”（左栏）
          * 确保“CI\_BUILD\_ARGS”和“CI\_TEST\_ARGS”保留默认值
          * 点击 **build** 按钮
          * 文档末尾有关于如何[解释报告结果](https://www.google.com/search?q=%23how-to-read-the-coverage-rate-from-the-buildfarm-report)和[计算覆盖率](https://www.google.com/search?q=%23how-to-calculate-the-coverage-rate-from-the-buildfarm-report)的说明
7.  如果 CI 任务构建无警告、错误和测试失败，请将任务链接发布在您的 PR 或汇总所有 PR 的高级工单上（参见[此处](https://www.google.com/search?q=https://github.com/ros2/ros2/issues/1090)示例）
      * 注意，这些徽章的 markdown 位于 `ci_launcher` 任务的控制台输出中
8.  当 PR 获得批准后：
      * 提交 PR 的人使用“Squash and Merge”（压缩并合并）选项将其合并，以便我们保持清晰的历史记录
      * 如果提交值得保持分离：将所有 nitpick/linter/typo 提交压缩在一起，然后合并剩余的集
      * 注意：每个 PR 应针对特定功能，因此“Squash and Merge”在 99% 的时间里应该是合理的
      * 合并后删除分支

### Gitconfig 优化

为了能够推送到仓库，您需要在系统上设置 ssh 密钥。但是，我们仓库的默认 url 模式是使用 https，因为它是可匿名访问的。在您的系统上，您可以使用 `gitconfig` 选项 `insteadOf` 让 `git` 自动使用您的 ssh 密钥，即使远程声明为 https。

将以下内容添加到您的 `~/.gitconfig`：

```ini
[url "ssh://git@github.com/"]
  insteadOf = https://github.com/
```

如果您在 GitLab 或 Bitbucket 上的仓库工作，您可以做同样的事情。

-----

## 架构开发实践

本节描述了在对 ROS 2 进行大型架构更改时应采用的理想生命周期。

### 软件开发生命周期

本节逐步描述如何规划、设计和实现新功能：

1.  任务创建
2.  创建设计文档
3.  设计评审
4.  实现
5.  代码审查

#### 任务创建

需要更改 ROS 2 关键部分的任务应在发布周期的早期阶段进行设计评审。如果在后期阶段进行设计评审，这些更改将成为未来发布的一部分。

  * 应在适当的 [ros2 仓库](https://github.com/ros2)中创建一个 issue，清楚地描述正在处理的任务。
  * 它应该有明确的成功标准，并强调预期的具体改进。
  * 如果该功能针对某个 ROS 发行版，请确保在 ROS 发布工单中对其进行跟踪（[示例](https://www.google.com/search?q=https://github.com/ros2/ros2/issues/1090)）。

#### 编写设计文档

设计文档绝不能包含机密信息。您的更改是否需要设计文档取决于任务的大小。

**您正在进行小的更改或修复错误：**
不需要设计文档，但应在适当的仓库中打开一个 issue 来跟踪工作并避免重复工作。

**您正在实现新功能或希望为 OSRF 拥有的基础设施（如 Jenkins CI）做出贡献：**
需要设计文档，并应将其贡献给 [ros2/design](https://github.com/ros2/design)，以便在 [https://design.ros2.org/](https://design.ros2.org/) 上访问。

1.  您应该分叉 (fork) 仓库并提交详细说明设计的合并请求。
2.  在合并请求或提交消息中提及相关的 ros2 issue（例如，`Design doc for task ros2/ros2#<issue id>`）。详细说明在 [ROS 2 贡献](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing.html)页面上。设计评论将直接在合并请求上进行。
3.  如果该任务计划与特定版本的 ROS 一起发布，则应在合并请求中包含此信息。

#### 设计文档评审

设计准备好审核后，应打开合并请求并分配适当的审阅者。建议将项目所有者 - 所有受影响包的维护者（由 `package.xml` 维护者字段定义，参见 [REP-140](https://github.com/ros-infrastructure/rep/blob/master/rep-0140.rst)） - 作为审阅者。

如果设计文档很复杂或审阅者的时间表冲突，可以安排可选的设计评审会议。在这种情况下，

**会议前**

  * 至少提前一周发送会议邀请
  * 建议会议时长一小时
  * 会议邀请应列出评审期间要做的所有决定（需要包维护者批准的决定）
  * 会议必须参加者：设计合并请求审阅者
  * 会议可选参加者：所有 OSRF 工程师（如果适用）

**会议期间**

  * 任务所有者推动会议，陈述他们的想法并管理讨论，以确保按时达成协议

**会议后**

  * 任务所有者应将会议记录发送给所有与会者
  * 如果对设计提出了小问题：
      * 任务所有者应根据反馈更新设计文档合并请求
      * 不需要额外的审核
  * 如果对设计提出了重大问题：
      * 可以接受删除没有明确协议的部分
      * 设计中可争议的部分可以作为单独的任务在将来重新提交
      * 如果删除可争议部分不是选项，请直接与包所有者合作达成协议

**一旦达成共识：**

  * 确保 `ros2/design` 合并请求已合并（如果适用）
  * 更新并关闭与此设计任务关联的 GitHub issue

#### 实现

开始之前，请浏览[合并请求](https://www.google.com/search?q=%23pull-requests)部分以获取最佳实践。

对于每个要修改的 repo：

1.  修改代码，如果完成则转到下一步，或者定期备份您的工作。
2.  使用 `git add -i` **自我审查**您的更改。
3.  使用 `git commit -s` 创建一个新的签名提交。
4.  一个合并请求应包含最少的语义上有意义的提交（例如，大量 1 行提交是不可接受的）。在迭代反馈时创建新的 fixup 提交，或者如果您不想每次都创建新提交，可以选择使用 `git commit --amend` 修改现有提交。
5.  每个提交必须有一个正确编写的、有意义的提交消息。更多说明[在此](https://cbea.ms/git-commit/)。
6.  移动文件必须在单独的提交中完成，否则 git 可能无法准确跟踪文件历史记录。
7.  合并请求描述或提交消息必须包含对相关 ros2 issue 的引用，以便在合并合并请求时自动关闭它。有关更多详细信息，请参阅此[文档](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue)。
8.  推送新提交。

#### 代码审查

一旦更改准备好进行代码审查：

1.  为每个修改的仓库打开一个合并请求。
2.  记得遵循[合并请求](https://www.google.com/search?q=%23pull-requests)最佳实践。
3.  [GitHub](https://cli.github.com/) 可用于从命令行创建合并请求。
4.  如果任务计划与特定版本的 ROS 一起发布，则应在每个合并请求中包含此信息。
5.  应在合并请求中提及审查设计文档的包所有者。
6.  **代码审查 SLO**：虽然审查合并请求是尽力而为，但审阅者在一周内对合并请求发表评论，代码作者在一周内回复评论是有帮助的，这样就不会丢失上下文。
7.  照常迭代反馈，根据需要修改和更新开发分支。
8.  一旦 PR 获得批准，包维护者将合并更改。

-----

## 构建农场 (Build Farm) 介绍

构建农场位于 [ci.ros2.org](https://ci.ros2.org/)。
每天晚上，我们都会运行夜间任务，在各种平台上以各种场景构建并运行所有测试。此外，我们在合并之前针对这些平台测试所有合并请求。
检查[当前的目标平台和架构集](https://www.ros.org/reps/rep-2000.html)，尽管它会随着时间推移而演变。

构建农场上有几种类型的任务：

  * **manual jobs** (由开发者手动触发):
      * `ci_linux`: 在 Ubuntu 上构建 + 测试代码
      * `ci_linux-aarch64`: 在 ARM 64 位机器 (aarch64) 上的 Ubuntu 上构建 + 测试代码
      * `ci_linux_coverage`: 构建 + 测试 + 生成测试覆盖率
      * `ci_linux-rhel`: 在 Red Hat Enterprise Linux 上构建 + 测试代码
      * `ci_windows`: 在 Windows 上构建 + 测试代码
      * `ci_launcher`: 触发上面列出的所有任务
  * **nightly** (每晚运行):
      * Debug: 使用 `CMAKE_BUILD_TYPE=Debug` 构建 + 测试代码
          * `nightly_linux_debug`
          * `nightly_linux-aarch64_debug`
          * `nightly_linux-rhel_debug`
          * `nightly_win_deb`
      * Release: 使用 `CMAKE_BUILD_TYPE=Release` 构建 + 测试代码
          * `nightly_linux_release`
          * `nightly_linux-aarch64_release`
          * `nightly_linux-rhel_release`
          * `nightly_win_rel`
      * Repeated: 构建然后运行每个测试最多 20 次或直到失败（也称为 flakiness hunter）
          * `nightly_linux_repeated`
          * `nightly_linux-aarch64_repeated`
          * `nightly_linux-rhel_repeated`
          * `nightly_win_rep`
      * Coverage:
          * `nightly_linux_coverage`: 构建 + 测试代码 + 分析 c/c++ 和 python 的覆盖率
              * 结果导出为 cobertura 报告
  * **packaging** (每晚运行；结果打包成归档):
      * `packaging_linux`
      * `packaging_linux-rhel`
      * `packaging_windows`

另外两个构建农场通过提供源码包和二进制包的构建、持续集成、测试和分析来支持 ROS / ROS 2 生态系统。
有关详细信息、常见问题解答和故障排除，请参阅[构建农场](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Build-Farms.html)。

### 关于覆盖率运行的说明

ROS 2 包的组织方式使得给定包的测试代码不仅包含在包内，而且也可能存在于不同的包中。换句话说：包可以在测试阶段执行属于其他包的代码。
为了达到 ROS 2 核心包中所有可用代码达到的覆盖率，建议使用一组固定的建议仓库运行构建。该集在 Jenkins 中覆盖率任务的默认参数中定义。

### 如何从构建农场报告中读取覆盖率

要查看给定包的覆盖率报告：

1.  当 `ci_linux_coverage` 构建完成后，点击 **Coverage Report**
2.  向下滚动到 **Coverage Breakdown by Package** 表
3.  在表中，查看名为“Name”的第一列

构建农场中的覆盖率报告包括 ROS 工作区中使用的所有包。覆盖率报告包括对应于同一包的不同路径：

  * 形式为 `src.*.<repository_name>.<package_name>.*` 的 Name 条目：这些对应于包中针对其自身源代码可用的单元测试运行。
  * 形式为 `build.<repository_name>.<package_name>.*` 的 Name 条目：这些对应于包中针对其在构建或配置时生成的文件可用的单元测试运行。
  * 形式为 `install.<package_name>.*` 的 Name 条目：这些对应于来自其他包测试运行的系统/集成测试。

### 如何从构建农场报告中计算覆盖率

使用自动脚本获取合并的单元覆盖率：

1.  从 `ci_linux_coverage` Jenkins 构建中复制构建的 URL
2.  下载 [`get_coverage_ros2_pkg`](https://www.google.com/search?q=%5Bhttps://github.com/ros2/ros2/blob/rolling/scripts/get_coverage_ros2_pkg.py%5D\(https://github.com/ros2/ros2/blob/rolling/scripts/get_coverage_ros2_pkg.py\)) 脚本
3.  执行脚本：`./get_coverage_ros2_pkg.py <jenkins_build_url> <ros2_package_name>` ([README](https://www.google.com/search?q=https://github.com/ros2/ros2/blob/rolling/scripts/README.md))
4.  从脚本输出的“Combined unit testing”最后一行获取结果

**替代方案**：从覆盖率报告中获取合并的单元覆盖率（需要手动计算）：

1.  当 `ci_linux_coverage` 构建完成后，点击 **Cobertura Coverage Report**
2.  向下滚动到 **Coverage Breakdown by Package** 表
3.  在表中，在第一列“Name”下，寻找（其中 `<package_name>` 是您正在测试的包）：
      * 模式 `src.*.<repository_name>.<package_name>.*` 下的所有目录，抓取“Lines”列中的两个绝对值。
      * 模式 `build/.<repository_name>.*` 下的所有目录，抓取“Lines”列中的两个绝对值。
4.  对于之前的选择：对于每个单元格，第一个值是测试的行数，第二个是代码的总行数。汇总所有行以获得测试行总数和测试下代码行总数。相除以获得覆盖率。

### 如何在本地使用 lcov 测量覆盖率 (Ubuntu)

要在您自己的机器上测量覆盖率，请安装 `lcov`。

```bash
sudo apt install -y lcov
```

本节的其余部分假设您正在从 colcon 工作区工作。使用覆盖率标志在 debug 模式下编译。随意使用 colcon 标志来针对特定包。

```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="${CMAKE_CXX_FLAGS} --coverage" -DCMAKE_C_FLAGS="${CMAKE_C_FLAGS} --coverage"
```

`lcov` 需要一个初始基线，您可以使用以下命令生成。根据您的需要更新输出文件位置。

```bash
lcov --no-external --capture --initial --directory . --output-file ~/ros2_base.info
```

为您覆盖率测量所关注的包运行测试。例如，如果测量 `rclcpp` 以及 `test_rclcpp`：

```bash
colcon test --packages-select rclcpp test_rclcpp
```

捕获 lcov 结果，这次使用类似的命令但去掉 `--initial` 标志。

```bash
lcov --no-external --capture --directory . --output-file ~/ros2.info
```

合并追踪 `.info` 文件：

```bash
lcov --add-tracefile ~/ros2_base.info --add-tracefile ~/ros2.info --output-file ~/ros2_coverage.info
```

生成 html 以便轻松可视化和注释覆盖的行。

```bash
mkdir -p coverage
genhtml ~/ros2_coverage.info --output-directory coverage
```