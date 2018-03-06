# 编译Python

### 1. Python 总体架构

- 文件类别
  - **核心模块：** Python 内建的模块
  - **库：** `pip install`
  - **用户自定义模块：** 普通的`.py`
- Python运行环境
  - **对象类型系统：** 包含了 Python 中存在的各种内建对象，比如整数，list 和 dict
  - **内存分配器：** 全权负责 Python 中创建对象时对内存的申请工作，实际上它就是 Python 运行时与 C
    中 malloc 的一层接口
  - **当前运行状态：** 维护了解释器在执行字节码时在不同的状态之间切换的动作，我们可以将它视为一个巨大而复杂的有穷状态机
- 解释器
  - **Scanner：** 负责词法分析，将 Python 代码切分成一个个 token
  - **Parser：** 负责语法分析，将 Scanner 产生的 token 汇总产生抽象语法树
  - **Compiler：** 根据语法树生成指令集合，即 Python 字节码
  - **Code Evaluator：** 解释执行字节码，也被称为执行引擎

![overall_structure](https://github.com/igululu/Pyek/blob/master/image/overall_structure.png)