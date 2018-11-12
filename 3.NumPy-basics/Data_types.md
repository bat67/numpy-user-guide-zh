<!-- TOC -->

- [数据类型](#数据类型)
    - [数组类型和类型之间的转换](#数组类型和类型之间的转换)
    - [数组标量](#数组标量)
    - [扩展精度](#扩展精度)

<!-- /TOC -->

# 数据类型

> 原文：[Data types](http://docs.scipy.org/doc/numpy-dev/user/basics.types.html)

> 另见：
[数据类型对象](http://docs.scipy.org/doc/numpy-dev/reference/arrays.dtypes.html#arrays-dtypes)

## 数组类型和类型之间的转换

NumPy支持的数值类型比Python更多。这一节会讲述所有可用的类型，以及如何改变数组的数据类型。

| 数据类型 | 描述 |
| --- | --- |
| bool_ | 以字节存储的布尔值（True 或 False） |
| int_ | 默认的整数类型（和 C 的 long 一样，是 int64 或者 int32） |
| intc | 和 C 的 int 相同（一般为 int64 或 int32） |
| intp | 用于下标的整数（和 C 的 ssize_t 相同，一般为int64 或者 int32） |
| int8 | 字节（-128 到 127） |
| int16 | 整数（-32768 到 32767） |
| int32 | 整数（-2147483648 到 2147483647） |
| int64 | 整数（-9223372036854775808 到 9223372036854775807） |
| uint8 | 无符号整数（0 到 255） |
| uint16 | 无符号整数（0 到 65535） |
| uint32 | 无符号整数（0 到 4294967295） |
| uint64 | 无符号整数（0 到 18446744073709551615） |
| float_ | float64 的简写 |
| float16 | 半精度浮点：1位符号，5位指数，10位尾数 |
| float32 | 单精度浮点：1位符号，8位指数，23位尾数 |
| float64 | 双精度浮点：1位符号，11位指数，52位尾数 |
| complex_ | complex128 的简写 |
| complex64 | 由两个32位浮点（实部和虚部）组成的复数 |
| complex128 | 由两个64位浮点（实部和虚部）组成的复数 |

此外，Intel平台相关的C整数类型 `short`、`long`，`long long` 和它们的无符号版本是有定义的。

NumPy数值类型是dtype对象的实例，每个都有独特的特点。一旦你导入了NumPy：

```python
>>> import numpy as np
```

这些 dtype 都可以通过 `np.bool_` ， `np.float32` 以及其它的形式访问。

更高级的类型不在表中给出，请见[结构化数组](http://docs.scipy.org/doc/numpy-dev/user/basics.rec.html#structured-arrays)一节。

有5种基本的数值类型：布尔（`bool`），整数（`int`），无符号整数（`uint`），浮点（`float`）和复数。其中的数字表示类型所占的位数（即需要多少位代表内存中的一个值）。有些类型，如`int`和`intp`，依赖于平台（例如32位和64位机）有不同的位数。在与低级别的代码（如C或Fortran）交互和在原始内存中寻址时应该考虑到这些。

数据类型可以用做函数，来将Python类型转换为数组标量（详细解释请见数组标量一节），或者将Python的数值序列转换为同类型的NumPy数组，或者作为参数传入接受dtype的关键词的NumPy函数或方法中，例如：

```python
>>> import numpy as np
>>> x = np.float32(1.0)
>>> x
1.0
>>> y = np.int_([1,2,4])
>>> y
array([1, 2, 4])
>>> z = np.arange(3, dtype=np.uint8)
>>> z
array([0, 1, 2], dtype=uint8)
```

数组类型也可以由字符代码指定，这主要是为了保留旧的包的向后兼容，如Numeric。一些文档仍旧可能这样写，例如：

```python
>>> np.array([1, 2, 3], dtype='f')
array([ 1.,  2.,  3.], dtype=float32)
```

我们推荐用 dtype 对象来取代。

要转换数组类型，使用 `.astype()` 方法（推荐），或者将类型自身用作函数，例如：

```python
>>> z.astype(float)
array([  0.,  1.,  2.])
>>> np.int8(z)
array([0, 1, 2], dtype=int8)
```

需要注意的是，上面我们使用Python的浮点对象作为 dtype。NumPy知道`int`是指`np.int_`，`bool`指`np.bool_`，`float`指`np.float_`，`complex`指`np.complex_ `。其他数据类型在Python中没有对应。

通过查看 dtype 属性来确定数组的类型：

```python
>>> z.dtype
dtype('uint8')
```

dtype 对象还包含有关类型的信息，如它的位宽和字节顺序。数据类型也可以间接用于类型的查询属性，例如检查是否是整数：

```python
>>> d = np.dtype(int)
>>> d
dtype('int32')

>>> np.issubdtype(d, int)
True

>>> np.issubdtype(d, float)
False
```

## 数组标量

NumPy一般以数组标量返回数组元素（带有相关dtype的标量）。数组标量不同于Python标量，但他们中的大部分可以互换使用（一个主要的例外是2.x之前的Python，其中整数数组标量不能作为列表和元组的下标）。也有一些例外，比如当代码需要标量的一个非常特定的属性，或检查一个值是否是特定的Python标量时。一般来说，总是可以使用相应的Python类型函数（如`int`，`float`，`complex`，`str`，`unicode`），将数组标量显式转换为Python标量来解决问题。

使用数组标量的主要优点是，它们保留了数组的类型（Python可能没有匹配的标量类型，如`int16`）。因此，使用数组标量确保了数组和标量之间具有相同的行为，无论值在不在数组中。NumPy标量也有许多和数组相同的方法。

## 扩展精度

Python 的浮点数通常都是64位的，几乎相当于 `np.float64` 。在一些不常见的情况下，更精确的浮点数可能更好。是否可以这样做取决于硬件和开发环境：具体来说，x86 机器提供了80位精度的硬件浮点支持，虽然大多数 C 编译器都以 `long double` 类型来提供这个功能，但 MSVC （标准的Windows版本）中 `long double` 和 `double` 一致。NumPy中可以通过 `np.longdouble` 来使用编译器的 `long double` （复数为 `np.clongdouble` ）。你可以通过 `np.finfo(np.longdouble)` 来了解你的 numpy 提供了什么。

NumPy 不提供比 C 的 `long double` 精度更高的 dtype；特别是128位 IEEE 四精度数据类型（Fortran 的 `REAL*16`）是不能用的。

为了高效的内存对齐，`np.longdouble`通常填充零位来存储，共96位或128位。哪个更有效取决于硬件环境；通常在32位系统中，他们被填充到96位，而在64位系统，他们通常是填充到128位。`np.longdouble` 以系统默认的方式填充；而 `np.float96` 和 `np.float128` 为那些需要特定填充位的用户提供。尽管名字不同，`np.float96` 和 `np.float128`都只提供和`np.longdouble`相同的精度，也就是说，大多数 x86 机器上面只有80位，标准Windows版本上只有64位。

注意，即使`np.longdouble`比Python的`float`精度更高，也很容易失去额外的精度，因为Python经常强行以`float`来传值。例如，`%`格式化运算符要求其参数转换成标准的Python类型，因此它不可能保留额外的精度，即使要求更多的小数位数。可以使用`1 + np.finfo(np.longdouble).eps`来测试你的代码。