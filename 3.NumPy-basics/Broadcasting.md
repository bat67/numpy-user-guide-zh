
<!-- TOC -->

- [广播](#广播)
    - [一般的广播规则](#一般的广播规则)

<!-- /TOC -->


# 广播

> 原文：[Broadcasting](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)

> 另见：[numpy.broadcast](https://docs.scipy.org/doc/numpy/reference/generated/numpy.broadcast.html#numpy.broadcast)

术语广播描述了NumPy在算术运算时如何处理不同形状的数组。 在某些条件下，较小的数组“广播”成较大的数组以便有相同的形状。 广播提供了一种矢量化操作数组的方法，这样可以在C而不是Python中进行循环。 它可以在不制作不必要的数据副本的情况下实现这一点，并且通常可以实现高效 然而，有些情况下广播是一个坏主意，因为它会导致内存使用效率低下，从而减慢计算速度。

NumPy的运算通常是在数组对里逐元素执行的。在最简单的情况下，两个数组的形状必须完全相同，如下例所示：

```python
>>> a = np.array([1.0, 2.0, 3.0])
>>> b = np.array([2.0, 2.0, 2.0])
>>> a * b
array([ 2.,  4.,  6.])
```

NumPy的广播在数组形状满足某些条件时放宽了这个规则。最简单的广播发生在数组和一个标量值组合的操作中：

```python
>>> a = np.array([1.0, 2.0, 3.0])
>>> b = 2.0
>>> a * b
array([ 2.,  4.,  6.])
```

结果与前面的例子中`b`是一个数组是相等的。我们可以把标量b在算术运算中拉伸成与`a`形状相同的数组。`b`中的新元素只是原始标量的拷贝。拉伸类比只是概念上的。NumPy足够聪明，可以使用原始的标量值，而无需实际进行复制，因此广播操作能具有较低内存和较高的计算效率。

第二个示例中的代码比第一个示例中的代码更高效，因为在乘法计算中，广播移动的内存更少（`b`是标量而不是数组）。

## 一般的广播规则

在操作两个数组时，NumPy逐元素比较它们的形状。它从后面的维度开始，然后继续前进。两个维度在如下情况叫兼容的：

* 它们相等，或者
* 其中一个是1
  
如果不满足这些条件，就会抛出一个异常`ValueError: frame are not aligned exception`，表示数组具有不兼容的形状。结果数组的大小是沿输入数组的每个维度的最大值。

数组的维数不需要相同。例如，如果你有一个包含RGB值的`256x256x3`数组，并且你想用一个不同的值来缩放图像中的每种颜色，你可以用一个一维数组3去乘。根据广播规则排列这些阵列的尾轴的大小，表明它们是兼容的:

```
Image  (3d array): 256 x 256 x 3
Scale  (1d array):             3
Result (3d array): 256 x 256 x 3
```

当比较的两个维度之一时，就使用另一个。换句话说，大小为1的维度被拉伸或“复制”以匹配另一个维度。
在下面的例子中，A和B数组都有一个长度为1的轴，在广播操作中，轴被放大到更大的尺寸:

```
A      (4d array):  8 x 1 x 6 x 1
B      (3d array):      7 x 1 x 5
Result (4d array):  8 x 7 x 6 x 5
```

下面是更多的例子:

```
A      (2d array):  5 x 4
B      (1d array):      1
Result (2d array):  5 x 4

A      (2d array):  5 x 4
B      (1d array):      4
Result (2d array):  5 x 4

A      (3d array):  15 x 3 x 5
B      (3d array):  15 x 1 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 1
Result (3d array):  15 x 3 x 5
```

下面是一些形状不能广播的例子:

```
A      (1d array):  3
B      (1d array):  4 # trailing dimensions do not match

A      (2d array):      2 x 1
B      (3d array):  8 x 4 x 3 # second from last dimensions mismatched
```

实际中一个广播的例子:

```python
>>> x = np.arange(4)
>>> xx = x.reshape(4,1)
>>> y = np.ones(5)
>>> z = np.ones((3,4))

>>> x.shape
(4,)

>>> y.shape
(5,)

>>> x + y
<type 'exceptions.ValueError'>: shape mismatch: objects cannot be broadcast to a single shape

>>> xx.shape
(4, 1)

>>> y.shape
(5,)

>>> (xx + y).shape
(4, 5)

>>> xx + y
array([[ 1.,  1.,  1.,  1.,  1.],
       [ 2.,  2.,  2.,  2.,  2.],
       [ 3.,  3.,  3.,  3.,  3.],
       [ 4.,  4.,  4.,  4.,  4.]])

>>> x.shape
(4,)

>>> z.shape
(3, 4)

>>> (x + z).shape
(3, 4)

>>> x + z
array([[ 1.,  2.,  3.,  4.],
       [ 1.,  2.,  3.,  4.],
       [ 1.,  2.,  3.,  4.]])
```

广播提供了一种获取两个阵列的外部产品(或任何其他外部操作)的方便方法。下面的例子展示了两个一维数组的外积操作:

```python
>>> a = np.array([0.0, 10.0, 20.0, 30.0])
>>> b = np.array([1.0, 2.0, 3.0])
>>> a[:, np.newaxis] + b
array([[  1.,   2.,   3.],
       [ 11.,  12.,  13.],
       [ 21.,  22.,  23.],
       [ 31.,  32.,  33.]])
```

这里``newaxis``索引操作符将一个新轴插入到``a``中，使其成为一个二维``4x1``数组。将``4x1``数组与形状为``(3,)``的``b``组合，产生一个``4x3``数组。

有关广播概念的说明，请参阅[这篇文章](http://wiki.scipy.org/EricsBroadcastingDoc)