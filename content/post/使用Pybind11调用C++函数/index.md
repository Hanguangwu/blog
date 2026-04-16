---
title: 使用Pybind11调用C++函数
description: 本文介绍使用Pybind11调用C++函数的过程。
date: 2025-04-16T22:34:25-08:00
draft: false
categories:
- 编程
tags:
- C++
- Pybind
- Python
---


# 使用Pybind11调用C++函数


## pybind11 的作用

鉴于 Python 脚本的运行速度相对较慢，而 C++ 具有显著的性能优势，**Pybind11** 成为了解决 Python 性能瓶颈的关键工具。它允许 Python 代码**无缝调用**高性能的 C++ 程序，从而大幅提升脚本的执行效率。此外，Pybind11 还提供了**双向互操作性**，支持 C++ 程序反向调用 Python 脚本，实现了两种语言的灵活集成。

## 使用包管理器 (PyPI 或 Conda) 安装 pybind11

在项目中集成 Pybind11 库通常有两种方法，它们主要区别在于如何管理 Pybind11 的依赖和 CMake 的配置方式。

此方法将 Pybind11 作为环境依赖安装，通过 CMake 查找已安装的库。

### 优点

- **环境统一管理：** 版本由包管理器维护（尤其是 Conda）。
- **依赖健全：** 使用 Conda-forge安装时，能更好地处理 C/C++ 运行时库依赖（例如解决 `GLIBCXX` 冲突）。  


## 编写一个简单的 pybind11 例子

### **安装 Pybind11 包：**

```bash
# 强烈推荐在 Conda 环境中使用 conda-forge 频道
conda install -c conda-forge pybind11 

# 或者使用 pip
# pip install pybind11[global]
```

### 编写 C++ 程序

要想在 python 中调用 C++ 代码，首先我们需要有一个 C++ 程序。 简单编写一个加法程序：

```cpp
/**
* @file add.cpp
* @brief 简单的 pybind11 示例：将一个 C++ 函数导出为 Python 模块
*
* 说明：
* - 使用 pybind11 将一个原生 C++ 函数导出为名为 "add" 的 Python 扩展模块。
* - 模块导出一个函数 add(int, int)，用于返回两个整数的和。
* - 本文件展示了最小化的导出流程，便于学习 pybind11 的使用方法。
*
* 使用示例（Python）：
* import add
* result = add.add(1, 2) # result == 3
*
* 设计细节：
* - 函数签名：int add(int i, int j)
* 返回两个整数的和，函数本身不抛出异常，也不进行边界检查（简单示例）。
*
* - PYBIND11_MODULE(add, module)
* 该宏定义了模块初始化函数并将模块命名为 "add"（在 Python 中通过 import add 引入）。
* 在模块初始化体内：
* - 设置可选模块文档字符串 module.doc()
* - 使用 module.def("add", &add, "A function which adds two numbers") 将 C++ 函数绑定为 Python 可调用对象
*/

#include <pybind11/pybind11.h>

int add(int i, int j) {
	return i + j;
}

PYBIND11_MODULE(add, module) {
	module.doc() = "pybind11 example plugin"; //optional module docstring
	module.def("add", &add, "A function which adds two numbers");
}
```

### 编写 CMakeLists.txt

```cmake
# 最小 CMake 版本要求
cmake_minimum_required(VERSION 3.14)

# 定义项目名称为 'add'，并指定项目使用 C++ 语言。
project(add LANGUAGES CXX)

# 查找 Python 3 环境。
# COMPONENTS Interpreter Development: 确保找到 Python 解释器和开发头文件/库。
find_package(Python3 REQUIRED COMPONENTS Interpreter Development)

# 查找 pybind11 库。
# REQUIRED: 如果找不到 pybind11，则停止配置并报错。
find_package(pybind11 REQUIRED)

# 设置 C++ 语言标准为 C++11。pybind11 通常至少需要 C++11。
set(CMAKE_CXX_STANDARD 11)
# 强制要求编译器必须支持设置的 C++ 标准
set(CMAKE_CXX_STANDARD_REQUIRED True)
# 设置默认构建类型为 Release。这会启用优化选项。
set(CMAKE_BUILD_TYPE Release)
# 为 Release 构建类型设置高级优化标志 (-O3)
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

# 使用 pybind11 提供的便捷函数来创建 Python 扩展模块。
# 目标名: add (对应于 Python 中 import add)
# 源文件: add.cpp
# pybind11 会自动处理链接 Python 库和添加必要的包含目录。
pybind11_add_module(add add.cpp)
```

拥有这两个文件就可以开始编译了！

### 编译出可以被 python 调用的 C++ `add` 函数

在上面两个文件的同级文件夹中进行如下操作

```bash
mkdir build
cd build
cmake ../
make -j`nproc`
```

现在在 `build` 文件夹中生成了一个动态链接库文件：`add.cpython-313-aarch64-linux-gnu.so`

| 组件         | 含义          | 说明                                                                     |
| ---------- | ----------- | ---------------------------------------------------------------------- |
| add        | 模块名         | 这是在 pybind11_add_module(add ...) 中定义的模块名。在 Python 中会通过 import add 导入它。 |
| .cpython   | ABI 标签（解释器） | 表示这是为 CPython（标准 Python 解释器）构建的扩展模块。                                   |
| -313       | ABI 标签（版本）  | 表示该模块是针对 Python 3.13 的 ABI 编译的，只能被 Python 3.13 或兼容 ABI 的版本加载。          |
| -aarch64   | 平台/架构       | 表示该模块是为 ARM 64 位架构（例如 Raspberry Pi 64 位、Arm 服务器）构建的。                   |
| -linux-gnu | 操作系统/发行版    | 表示使用 GNU 工具链 在 Linux 系统上构建的。                                           |
| .so        | 扩展名         | Linux/Unix 系统中的共享库（Shared Object），类似 Windows 的 .dll、macOS 的 .dylib。    |

之后我们可以在 `python` 环境中调用这个动态链接库。 在 `build` 文件夹中创建 `add.py` 文件

```python
import add

a = 10
b = 20
c = add.add(a, b)
print(f"{a} + {b} = {c}")
```

执行这个文件就能看到使用 C++ 编写的 `add` 函数被成功调用了！

### 文件结构

```text
.
├── add.cpp
├── build
│   ├── add.cpython-313-aarch64-linux-gnu.so
│   ├── add.py
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── cmake_install.cmake
│   └── Makefile
├── CMakeLists.txt
├── extern
│   └── pybind11
└── __pycache__
    └── add.cpython-313.pyc
```

## 参考资料

[PyBind11文档](https://pybind11.readthedocs.io/en/stable/index.html)

[pybind11（一）：从零构建你的第一个 Python 可调用 C++ 模块](https://zhuanlan.zhihu.com/p/1976674574260254569)

[Cmake find_package没有找到Pybind11，即使有提示？](https://cloud.tencent.com/developer/ask/sof/107426067)

[如何从根本上解决 find_package() 找不到第三方库的问题](https://zhuanlan.zhihu.com/p/151811267)





