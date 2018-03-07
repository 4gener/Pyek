# 编译 Python

### 1. Python 总体架构

- 文件类别
  - **核心模块：** Python 内建的模块
  - **库：** `pip install`
  - **用户自定义模块：** 普通的`.py`
- Python运行环境
  - **对象类型系统：** 包含了 Python 中存在的各种***内建对象***，比如整数，list 和 dict
  - **内存分配器：** 全权负责 Python 中创建对象时对***内存的申请工作***，实际上它就是 Python 运行时与 C
    中 malloc 的一层接口
  - **当前运行状态：** 维护了解释器在***执行字节码时在不同的状态之间切换***的动作，我们可以将它视为一个巨大而复杂的有穷状态机
- 解释器
  - **Scanner：** 负责词法分析，将 Python 代码切分成一个个 token
  - **Parser：** 负责语法分析，将 Scanner 产生的 token 汇总产生抽象语法树
  - **Compiler：** 根据语法树生成指令集合，即 Python 字节码
  - **Code Evaluator：** 解释执行字节码，也被称为执行引擎

![overall_structure](https://github.com/igululu/Pyek/blob/master/image/overall_structure.png?raw=true)

### 2. Python 源代码组织 

![directory_structure](https://github.com/igululu/Pyek/blob/master/image/directory_structure.png?raw=true)

- :file_folder: **Include:** 包含了 Python 提供的 ***所有头文件*** `.h`，如果用户需要自己用 C 或 C++ 来编写自定义模块来扩展 Python ，那么就需要用到这里提供的头文件
- 📁 **Lib:** 包含了 Python 自带的 ***所有标准库*** ，这些库都是 `.py` 且对速度没有严格要求
- 📁 **Modules:** 包含了所有用 C 编写的模块，这些模块都是对速度要求严格的模块，所以都是 `.c`


- 📁 **Parser:** 包含了 Python 解释器中的 ***Scanner*** 和 ***Parser*** 部分，即对 Python 源代码进行 ***词法分析*** 和 ***语法分析*** 的部分。除此之外也包括一些有用的工具，它们可以 ***根据 Python 的语法自动生成 Python 的词法和语法分析器*** ，类似 YACC

- 📁 **Objects:** 包含了 Python ***所有的内建对象***，包括整数，list 和 dict 等，同时包括了 Python 在运行时需要的 ***所有内部使用对象的实现***


- 📁 **Python:** 包含了 Python 解释器中 ***Compiler 和执行引擎部分***


- 📁 **PCBuild:** 包含了 ***Visual Studio 的工程文件***