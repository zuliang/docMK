# 泛型编程
## C语言泛型
C语言可以让程序员在微观层面写出非常精细和精确的编程操作，让程序员在底层和系统细节上非常自由、灵活和精准地控制代码。
然而在代码组织和功能编程上，C语言的上述特性，却不那么美妙。以int swap函数做例子：
```c
void swap(int* x, int* y)
{
    int temp = *x;
    *x = *y;
    *y = temp;
}
```
如果希望swap更类型通用，泛型版本swap如下
```c
void swap(void* x, void* y, size_t size)
{
    char tmp[size];
    memcpy(tmp, y, size);
    memcpy(y, x, size);
    memcpy(x, tmp, size);
}
```
有三个点值得注意：
1. **size参数**：由于void\* 把类型抽象了，编译器不能通过类型获得类型长度，所以需要手动加类型长度标识。
2. **memcpy()函数**：因为类型抽象了，赋值表达式不能使用，传过来的参数有可能是一个结构体，所以我们只能用内存复制方法。
3. **temp[size]数组**：临时存储空间buffer。
由此可见，C语言在类型抽象会带来编程复杂度的提升。而且，在提升复杂度的同时，如果我们发现还有问题，如果想交换字符串数组，类型是char\*，那么，我们的swap()函数的x, y是不是要用void\*\*？这样一来，借口无法定义。
这是我们可以用宏替代泛型：
```c
# define swap(x, y, size) 
{
    char temp[size];
    memcpy(tmp, &y, size);
    memcpy(&y, &x, size);
    memcpy(&x, tmp, size);
}
```
宏带来的问题是编译器做字符串替代，带着代码膨胀，编译出来的执行文件会比较大。
另外还有一个更严重的问题，例如min()函数：
```c
# define min(x, y) ((x)>(y) ? (y) : (x))
```
如果调用min(foo(), bar())，我们本意是调用foo()和bar()函数的返回值，然而宏替换后，foo()和bar()会被调用两次，就会带来很多问题。
回到刚刚的swap()泛型函数，你应该会觉得泛型太宽松了，不做类型检查就直接内存对拷，感觉就像定时炸弹一样，不知道什么时候就炸了。
而且泛型提升了接口复杂度，将类型长度交给了调用者。
## python源码泛型
python是动态类型语言，而C语言静态类型语言，在C基础上实现动态类型，python源码通过PyObject实现不同类型变量与对象之间转换。例如float类型的加法实现如下：
```c
static PyObject *
float_add(PyObject *v, PyObject *w)
{
    double a,b;
    CONVERT_TO_DOUBLE(v, a);
    CONVERT_TO_DOUBLE(w, b);
    PyFPE_START_PROTECT("add", return 0)
    a = a + b;
    PyFPE_END_PROTECT(a)
    return PyFloat_FromDouble(a);
}
```
python的swap()实现十分简单：
```python
a, b = b, a
```
这句话在python源码中执行就比较复杂了，首先需要了解的是python中的变量是以frame->f_locals字典形式保存变量名和具体变量值的，而且变量名和变量值都是PyObject。大体流程是：
1. LOAD_NAME\*2(读取变量值压栈两次)
2. ROT_TWO(交换栈里面的两个值)
3. STORE_NAME\*2(将栈顶变量弹出保存到新的变量名中两次)
4. LOAD_CONST(load 1个None返回值)
5. RETURN_VALUE(返回None值)

> 以下是一些代码片段可以自行感受一下：

`LOAD_NAME`:
```c
case TARGET(LOAD_NAME): {
    PyObject *name = GETITEM(names, oparg);
    PyObject *locals = f->f_locals;
    PyObject *v;
    if (locals == NULL) {
        PyErr_Format(PyExc_SystemError,
                     "no locals when loading %R", name);
        goto error;
    }
    if (PyDict_CheckExact(locals)) {
        v = PyDict_GetItem(locals, name);
        Py_XINCREF(v);
    }
    else {
        v = PyObject_GetItem(locals, name);
        if (v == NULL) {
            if (!PyErr_ExceptionMatches(PyExc_KeyError))
                goto error;
            PyErr_Clear();
        }
    }
    if (v == NULL) {
        v = PyDict_GetItem(f->f_globals, name);
        Py_XINCREF(v);
        if (v == NULL) {
            if (PyDict_CheckExact(f->f_builtins)) {
                v = PyDict_GetItem(f->f_builtins, name);
                if (v == NULL) {
                    format_exc_check_arg(
                                PyExc_NameError,
                                NAME_ERROR_MSG, name);
                    goto error;
                }
                Py_INCREF(v);
            }
            else {
                v = PyObject_GetItem(f->f_builtins, name);
                if (v == NULL) {
                    if (PyErr_ExceptionMatches(PyExc_KeyError))
                        format_exc_check_arg(
                                    PyExc_NameError,
                                    NAME_ERROR_MSG, name);
                    goto error;
                }
            }
        }
    }
    PUSH(v);
    DISPATCH();
}
```
`ROT_TWO`
```c
case TARGET(ROT_TWO): {
    PyObject *top = TOP();
    PyObject *second = SECOND();
    SET_TOP(second);
    SET_SECOND(top);
    FAST_DISPATCH();
}
```
`STORE_NAME`:
```c
case TARGET(STORE_NAME): {
    PyObject *name = GETITEM(names, oparg);
    PyObject *v = POP();
    PyObject *ns = f->f_locals;
    int err;
    if (ns == NULL) {
        PyErr_Format(PyExc_SystemError,
                     "no locals found when storing %R", name);
        Py_DECREF(v);
        goto error;
    }
    if (PyDict_CheckExact(ns))
        err = PyDict_SetItem(ns, name, v);
    else
        err = PyObject_SetItem(ns, name, v);
    Py_DECREF(v);
    if (err != 0)
        goto error;
    DISPATCH();
}
```
`LOAD_CONST`
```c
case TARGET(LOAD_CONST): {
    PREDICTED(LOAD_CONST);
    PyObject *value = GETITEM(consts, oparg);
    Py_INCREF(value);
    PUSH(value);
    FAST_DISPATCH();
}
```
`RETURN_VALUE`
```c
case TARGET(RETURN_VALUE): {
    retval = POP();
    assert(f->f_iblock == 0);
    goto return_or_yield;
}
```
## 小结
总体来说，C语言把非常底层的控制权交给了程序员，它的设计理念为：
1. 相信程序员；
2. 不会阻止程序员做任何底层的事
3. 保存语言的最小最简特征；
4. 保证C语言的最快运行速度，哪怕牺牲移植性。

C语言伟大之处在于程序员在高级语言的特性之上还能简单地做任何底层上的微观控制。不过，这只是针对底层指令控制和过程式编程方式，而对于更高阶更抽象的编程模型来说，C语言基于过程和底层的初衷设计方式就成为它的短板。所以，我们需要更贴近业务的抽象语言。

在C语言之后，C++、java、C#、python等语言前仆后继，都在试图解决那个时代的特定问题。再回过头来说，编程范式其实是程序的指导思想，它也代表了这门语言的设计方向。我们不能说哪种范式更优越，只能说各有千秋。

这里也顺便举一个动态类型语言的坑，如下代码，在不同的编程语言会有不一样的结果：
```
x = 5
y = "3"
z = x + y
```
如果是Visual Basic的结果是8，JavaScript是"53"，python和go就会运行时异常。这里想带出的一个概念就是任何一门语言都有特定的类型系统。动态类型的类型检查是在运行时完成，这个在方便的同时也会在某些情况下增加编程的复杂度，这里也印证了为何pep484会提倡用type hint把变量的类型和函数声明重新规范起来。

