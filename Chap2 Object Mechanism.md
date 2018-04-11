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

  - **规则：** 一个对象 *(1)* 一旦被创建，它在内存中的大小就不再发生变化。这就代表着那些需要容纳可变长度的数据的对象只能在对象中维护一个指针 *(2)* ，它指向一个存放着对象取值的可变大小的内存区域 *(3)* 。这样可以使通过指针维护对象的工作变得简单。

    ![variable_length_object](https://github.com/igululu/Pyek/blob/master/image/variable_length_object.png?raw=truehttps://github.com/igululu/Pyek/blob/master/image/variable_length_object.png?raw=true)

  - **原因：** 假设内存中有大小可变的对象 A ，其后紧跟着对象 B 。若允许对象大小在运行期改变，则假如运行期某个时刻 A 的大小发生了变化，这就意味着 ***必须将对象 A 移动至内存中的其他位置*** ，否则 A 增大的部分将覆盖属于对象 B 的内存。但是移动 A 到内存中的其他位置代表 ***所有指向 A 的指针必须立即得到更新*** ，这样的工作量是无法被接受的。

- 代码分析  [**object.h**](https://github.com/python/cpython/blob/master/Include/object.h)

  - `PyObject` 

    Python 中，所有的东西都是对象，它们也都拥有一些相同的内容，这些内容在 `PyObject` 中定义，它是 Python 中 ***不包含可变长度数据对象的基石*** 。

    ```c++
    typedef struct _object {
        _PyObject_HEAD_EXTRA	// 宏
        Py_ssize_t ob_refcnt;	// 引用计数
        struct _typeobject *ob_type;	// 对象类型
    } PyObject;
    ```

  - `PyVarObject`

    对于 ***包含可变长度数据的对象*** ，它们的基石是`PyVarObject`。可以看出，它们与不包含可变长度数据对象 ***拥有相同的对象头部*** ，这也使得 Python 中对对象的引用变得统一。

    ```c++
    typedef struct {
        PyObject ob_base;
        Py_ssize_t ob_size; // 变长变量中的元素个数
    } PyVarObject;
    ```

  - `_PyObject_HEAD_EXTRA`

    这个宏定义了`next`和`prev`指针，用来支持一个 ***将堆中所有对象串起来的双向链表*** 。

    ```c++
    #define _PyObject_HEAD_EXTRA            \
        struct _object *_ob_next;           \
        struct _object *_ob_prev;
    ```


  > Python3 与 Python2 相比，去掉了对`PyObject`和`PyVarObject`中的变量进一步的封装。 Python2 中它们分别只含宏`PyObject_HEAD`和`PyObject_VAR_HEAD`，对象中的几个具体变量也是在这两个宏中定义的。

### 2. 类型对象

- 代码分析  [**object.h**](https://github.com/python/cpython/blob/master/Include/object.h)

  - `PyTypeObject`

    `_typeobject`中包含了大量的 ***函数指针*** ，这些函数指针用来 ***指定不同类型的操作信息*** 。主要分为标准操作（`dealloc` `print` `compare`），标准操作族（`numbers` `sequences` `mappings`），以及其他操作（`hash` `buffer` `call`）。

    ```c++
    typedef struct _typeobject {
        PyObject_VAR_HEAD
        const char *tp_name; // 类型名，主要用于Python内部以及调试
        Py_ssize_t tp_basicsize, tp_itemsize; // 创建该类型对象时分配内存空间的大小信息

        // 与该类型关联对象相关联的操作信息
        hashfunc tp_hash;	// 指向该类型对象生成hash值的函数的函数指针
        ternaryfunc tp_call;	
        /* ... */
    } PyTypeObject;
    ```

  - `PyTypeObject`

    从`PyTypeObject`中可以看到，它的头部有`PyObject_VAR_HEAD`，这意味着 ***类型实际上也是一个对象*** 。

    ```c++
    #define PyObject_VAR_HEAD      PyVarObject ob_base;	// 包含可变长度数据的对象
    ```

- 代码分析 [**typeobject.c**](https://github.com/python/cpython/blob/master/Objects/typeobject.c)

  - `PyType_Type`

    Python 中，每一个对象都可以通过与其关联的类型对象确定其类型，而对于类型对象来说，它的类型就是`PyType_Type`。

    ```c++
    PyTypeObject PyType_Type = {	// 结构体变量初始化
        PyVarObject_HEAD_INIT(&PyType_Type, 0)		// 生成宏PyObject_VAR_HEAD
        "type",                                     /* tp_name */
        sizeof(PyHeapTypeObject),                   /* tp_basicsize */
        sizeof(PyMemberDef),                        /* tp_itemsize */
        /* ... */
    };
    ```

  - 一些宏起到的作用

    在`PyType_Type`中，它的头部由如下宏函数表示：

    ```c++
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    ```

    根据给定的一些宏定义：

    ```c++
    #ifdef Py_TRACE_REFS
    #define _PyObject_HEAD_EXTRA            \
        struct _object *_ob_next;           \
        struct _object *_ob_prev;

    #define _PyObject_EXTRA_INIT 0, 0,

    #else
    #define _PyObject_HEAD_EXTRA
    #define _PyObject_EXTRA_INIT
    #endif

    #define PyObject_HEAD                   PyObject ob_base;

    #define PyObject_HEAD_INIT(type)        \
        { _PyObject_EXTRA_INIT              \
        1, type },

    #define PyVarObject_HEAD_INIT(type, size)       \
        { PyObject_HEAD_INIT(type) size },
    ```

    可将该宏函数 ***逐级转化*** 为：

    ```c++
    { PyObject_HEAD_INIT(&PyType_Type) 0 },
    ```

    ```c++
    { { _PyObject_EXTRA_INIT              \
        1, &PyType_Type }, 0 },
    ```

    通过类型定义可将其类型 ***逐级规约*** 为：

    ```c++
    { { _PyObject_HEAD_EXTRA              \
        1, &PyType_Type }, 0 },
    ```

    ```c++
    { PyObject, 0},
    ```

    ```c++
    PyVarObject,
    ```

    `PyTypeObject`的头部`PyObject_VAR_HEAD`宏恰好也被定义为`PyVarObject`类型。这说明，如上宏函数协同将`PyVarObject_HEAD_INIT(&PyType_Type, 0)`层层转化或规约使其与`PyObject_VAR_HEAD`类型相同， ***完成`PyType_Type`作为`PyTypeObject`类型的实体化*** 。

- 代码分析 [**floatobject.c**](https://github.com/python/cpython/blob/master/Objects/floatobject.c)

  - `PyFloat_Type`

    ```c++
    PyTypeObject PyFloat_Type = {
        PyVarObject_HEAD_INIT(&PyType_Type, 0)	// 初始化对象头
        "float",
        sizeof(PyFloatObject),
        /* ... */
    };
    ```

    这是 Python 中最简单的一个类型对象，不过我们可以通过它大概得出一个浮点数对象在运行时的抽象表示，下图中的箭头表示属性中指向对象类型的指针`ob_type`。

    ![object_type_relation](image/object_type_relation.png?raw=true)

    ​