# 构建系统



**目录**

* 构建工具
* 构建助手
* ament_package 包
* ament_cmake 仓库
* ament_lint 仓库
* 元构建工具

构建系统允许开发人员按需构建其 ROS 2 代码。ROS 2 严重依赖于将代码划分为包（package），每个包都包含一个清单文件（`package.xml`）。该清单文件包含有关包的基本元数据，包括其对其他包的依赖关系。元构建工具需要此清单才能运行。

ROS 2 构建系统由 3 个主要概念组成。

## 构建工具

这是控制单个包的编译和测试的软件。在 ROS 2 中，对于 C++ 通常是 CMake，对于 Python 通常是 setuptools，但也支持其他构建工具。

## 构建助手

这些是挂钩到构建工具中以改善开发人员体验的辅助函数。ROS 2 包通常依赖 `ament` 系列包来实现此目的。`ament` 由 GitHub 组织中的几个重要仓库组成。

### ament_package 包

位于 GitHub 的 `ament/ament_package`，此仓库包含一个单独的 `ament` Python 包，为 `ament` 包提供各种实用工具，例如环境挂钩（environment hooks）的模板。

所有 `ament` 包必须在包的根目录下包含一个 `package.xml` 文件，无论其底层构建系统如何。`package.xml` “清单”文件包含处理和操作包所需的信息。此包信息包括包名称（全局唯一）和包的依赖项等内容。`package.xml` 文件还充当标记文件，指示包在文件系统上的位置。



`package.xml` 文件的解析由 `catkin_pkg` 提供（与 ROS 1 中相同），而在文件系统中搜索这些 `package.xml` 文件以定位包的功能则由诸如 `colcon` 之类的构建工具提供。

**package.xml**
> 包清单文件，用于标记包的根目录，并包含有关包的元信息，包括其名称、版本、描述、维护者、许可证、依赖项等。清单的内容采用机器可读的 XML 格式，其内容在 REP 127 和 140 中描述，并可能在未来的 REP 中进行进一步修改。

因此，每当某个包被称为 `ament` 包时，意味着它是一个使用 `package.xml` 清单文件描述的软件单元（源代码、构建文件、测试、文档和其他资源）。

**ament package (ament 包)**
> 任何包含 `package.xml` 且遵循 `ament` 打包指南的包，无论底层构建系统如何。

由于术语 `ament` 包与构建系统无关，因此可以有不同类型的 `ament` 包，例如 ament CMake 包、ament Python 包等。

以下是您在这个软件栈中可能遇到的常见包类型的列表：

* **CMake package (CMake 包)**
    任何包含普通 CMake 项目和 `package.xml` 清单文件的包。

* **ament CMake package (ament CMake 包)**
    一个同时也遵循 `ament` 打包指南的 CMake 包。

* **Python package (Python 包)**
    任何包含基于 setuptools 的 Python 项目和 `package.xml` 清单文件的包。

* **ament Python package (ament Python 包)**
    一个同时也遵循 `ament` 打包指南的 Python 包。

### ament_cmake 仓库

位于 GitHub 的 `ament/ament_cmake`，此仓库包含许多“ament CMake”包和纯 CMake 包，它们提供了在 CMake 中创建“ament CMake”包所需的基础设施。在这种语境下，“ament CMake”包意味着：使用 CMake 构建的 `ament` 包。因此，此仓库中的包提供了必要的 CMake 函数/宏和 CMake 模块，以促进创建更多的“ament CMake”（或 `ament_cmake`）包。此类型的包在 `package.xml` 文件的 `<export>` 标签中使用 `<build_type>ament_cmake</build_type>` 标签进行标识。



该仓库中的包非常模块化，但有一个名为 `ament_cmake` 的单一“瓶颈”包。任何人都可以依赖 `ament_cmake` 包来获取此仓库中包的所有聚合功能。以下是仓库中的包列表及其简短描述：

* **ament_cmake**
    聚合此仓库中的所有其他包，用户只需依赖它。
* **ament_cmake_auto**
    提供便捷的 CMake 函数，自动处理编写包的 `CMakeLists.txt` 文件中许多繁琐的部分。
* **ament_cmake_core**
    提供 `ament` 的所有内置核心概念，例如环境挂钩、资源索引、符号链接安装等。
* **ament_cmake_gmock**
    添加用于制作基于 gmock 的单元测试的便捷函数。
* **ament_cmake_gtest**
    添加用于制作基于 gtest 的自动化测试的便捷函数。
* **ament_cmake_nose**
    添加用于制作基于 nosetests 的 Python 自动化测试的便捷函数。
* **ament_cmake_python**
    为包含 Python 代码的包提供 CMake 函数。
    *参见 ament_cmake_python 用户文档*
* **ament_cmake_test**
    使用 CTest 将不同类型的测试（例如 gtest 和 nosetests）聚合在一个目标下。

`ament_cmake_core` 包包含许多 CMake 基础设施，使得使用常规接口在包之间清晰地传递信息成为可能。这使得包与其他包具有更加解耦的构建接口，从而促进它们的重用并鼓励不同包的构建系统中的约定。例如，它提供了一种标准方法来在包之间传递包含目录、库、定义和依赖项，以便这些信息的消费者可以以常规方式访问此信息。

`ament_cmake_core` 包还提供了 `ament` 构建系统的特性，如符号链接安装，允许您将文件从源空间或构建空间符号链接到安装空间，而不是复制它们。这允许您安装一次，然后编辑非生成的资源（如 Python 代码和配置文件），而无需重新运行安装步骤即可使其生效。此功能本质上取代了 `catkin` 中的“devel space”（开发空间），因为它具有大部分优点，却几乎没有复杂性或缺点。

`ament_cmake_core` 提供的另一个功能是包资源索引，这是包指示它们包含某种类型资源的一种方式。此功能的设计使得回答简单问题（如此前缀下有哪些包，例如 `/usr/local`）更加高效，因为它只需要您列出该前缀下单个可能位置的文件。您可以在资源索引的设计文档中阅读有关此功能的更多信息。

像 `catkin` 一样，`ament_cmake_core` 也提供环境设置文件和包特定的环境挂钩。环境设置文件（通常命名为 `setup.bash` 之类的名称）是包开发人员定义使用其包所需的环境更改的地方。开发人员能够使用“环境挂钩（environment hook）”来执行此操作，这基本上是一段任意的 shell 代码，可以设置或修改环境变量、定义 shell 函数、设置自动完成规则等……例如，这就是 ROS 1 如何在 `catkin` 对 ROS 发行版一无所知的情况下设置 `ROS_DISTRO` 环境变量的方式。

### ament_lint 仓库

位于 GitHub 的 `ament/ament_lint`，此仓库提供了几个以方便和一致的方式提供 linting（代码检查）和测试服务的包。目前有的包支持使用 uncrustify 进行 C++ 风格检查，使用 cppcheck 进行静态 C++ 代码检查，检查源代码中的版权，使用 pep8 进行 Python 风格检查，以及其他事项。助手包的列表将来可能会增加。

## 元构建工具

这是一个知道如何对一组包进行拓扑排序，并按正确的依赖顺序构建或测试它们的软件。该软件将调用**构建工具**来完成编译、测试和安装包的实际工作。

在 ROS 2 中，使用名为 `colcon` 的工具来执行此操作。