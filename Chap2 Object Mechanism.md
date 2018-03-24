# 对象机制

### 1. 对象

- 在 Python 中， ***一切都是对象*** ：

  - 整数，字符串
  - **类型：** 整数类型，字符串类型

- 不同场景下对于对象的定义：

  - **OOP：** 对象是数据以及基于这些数据的操作的集合

  - **计算机：** 对象是一片被分配的 ***连续性不确定*** 的内存空间，但是该空间在更高层次上可以作为一个整体来考虑，因为它们存储了 ***一系列数据*** 以及可以 ***对这些数据进行一系列操作的代码*** 。

  - **Python：** ***除类型对象以外*** ， Python 对象是在 ***堆上申请的结构体*** 且不能被静态初始化以及在栈空间上生存。而类型对象则全部是静态初始化的。

    > 栈内存中存放函数体中定义的局部变量，由编译器自动分配释放。而堆内存中存放动态分配等分配内存的函数，若代码中不进行回收，该部分内存将一直被占用或被操作系统回收。

- Python 对对象的控制：

  - **规则：**一个对象 *(1)* 一旦被创建，它在内存中的大小就不再发生变化。这就代表着那些需要容纳可变长度的数据的对象只能在对象中维护一个指针 *(2)* ，它指向一个存放着对象取值的可变大小的内存区域 *(3)* 。

    ![variable_length_object](https://github.com/igululu/Pyek/blob/master/image/variable_length_object.png?raw=truehttps://github.com/igululu/Pyek/blob/master/image/variable_length_object.png?raw=true)

- 代码分析

  ```c
  # object.h
  typedef struct _object {
      pyObject_HEAD
  } PyObject;
  ```

  ​