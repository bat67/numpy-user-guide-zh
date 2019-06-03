<!-- TOC -->

- [数据类型](#数据类型)
  - [数组类型和类型之间的转换](#数组类型和类型之间的转换)
  - [数组标量](#数组标量)
  - [扩展精度](#扩展精度)

<!-- /TOC -->

# 数据类型

> 原文：[Data types](https://docs.scipy.org/doc/numpy/user/basics.types.html)

> 另见：[数据类型对象](https://docs.scipy.org/doc/numpy/reference/arrays.dtypes.html#arrays-dtypes)

## 数组类型和类型之间的转换

NumPy支持的数值类型比Python多得多。这一节会讲述所有可用的类型，以及如何改变数组的数据类型。

所支持的基本类型与C语言中的基本类型紧密相连:

| Numpy类型 | C中的类型 |描述 |
| --- | --- | --- |
| np.bool | bool | 以字节存储的布尔值（True 或 False） |
| np.byte | signed char | 由平台定义
| np.ubyte | unsigned char | 由平台定义
| np.short | short | 由平台定义
|np.ushort|unsigned short| 由平台定义
|np.intc|int| 由平台定义
|np.uintc|unsigned int| 由平台定义
|np.int_|long| 由平台定义
|np.uint|unsigned long| 由平台定义
|np.longlong|long long| 由平台定义
|np.ulonglong|unsigned long long| 由平台定义
|np.half / np.float16|-|Half precision float: sign bit, 5 bits exponent, 10 bits mantissa|
|np.single|float|Platform-defined single precision float: typically sign bit, 8 bits exponent, 23 bits mantissa|
|np.double|double|Platform-defined double precision float: typically sign bit, 11 bits exponent, 52 bits mantissa.
|np.longdouble|long double|Platform-defined extended-precision float
|np.csingle|float complex|Complex number, represented by two single-precision floats (real and imaginary components)
|np.cdouble|double complex|Complex number, represented by two double-precision floats (real and imaginary components).
|np.clongdouble|long double complex|Complex number, represented by two extended-precision floats (real and imaginary components).

由于其中许多具有平台相关的定义，因此提供了一组固定大小的别名:

| Numpy类型 | C中的类型 |描述 |
| --- | --- | --- |
np.int8|int8_t|Byte (-128 to 127)
np.int16|int16_t|Integer (-32768 to 32767)
np.int32|int32_t|Integer (-2147483648 to 2147483647)
np.int64|int64_t|Integer (-9223372036854775808 to 9223372036854775807)
np.uint8|uint8_t|Unsigned integer (0 to 255)
np.uint16|uint16_t|Unsigned integer (0 to 65535)
np.uint32|uint32_t|Unsigned integer (0 to 4294967295)
np.uint64|uint64_t|Unsigned integer (0 to 18446744073709551615)
np.intp|intptr_t|Integer used for indexing, typically the same as ssize_t
np.uintp|uintptr_t|Integer large enough to hold a pointer
np.float32|float|-
np.float64 / np.float_|double|Note that this matches the precision of the builtin python float.
np.complex64|float complex|Complex number, represented by two 32-bit floats (real and imaginary components)
np.complex128 / np.complex_|double complex|Note that this matches the precision of the builtin python complex.



NumPy数值类型是`dtype`对象的实例，每个都有独特的特点。一旦导入了NumPy：

```python
>>> import numpy as np
```

这些 dtype 都可以通过 `np.bool_` ， `np.float32` 以及其它的形式访问。

更高级的类型不在表中给出，请见[结构化数组](https://docs.scipy.org/doc/numpy/user/basics.rec.html#structured-arrays)一节。

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