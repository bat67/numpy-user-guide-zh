<!-- TOC -->

- [和MATLAB的对比](#和matlab的对比)
  - [介绍](#介绍)
  - [一些关键的差异](#一些关键的差异)
  - [“array” 还是 “matrix”? 我应该选谁?](#array-还是-matrix-我应该选谁)
    - [简要的回答](#简要的回答)
    - [详细的回答](#详细的回答)
  - [MATLAB和NumPy粗略的功能对应表](#matlab和numpy粗略的功能对应表)
    - [一般功能的对应表](#一般功能的对应表)
    - [线性代数功能对应表](#线性代数功能对应表)
  - [注释](#注释)
  - [自定义环境](#自定义环境)
  - [链接](#链接)

<!-- /TOC -->
# 和MATLAB的对比

> 原文：[NumPy for Matlab users](https://docs.scipy.org/doc/numpy/user/numpy-for-matlab-users.html)

## 介绍

MATLAB®和 NumPy/SciPy 有很多共同之处。但是也有很多不同之处。创建NumPy和SciPy是为了用Python以最自然的方式进行数值和科学计算，而不是 MATLAB® 的克隆版。本章节旨在收集有关两者的差异，主要是为了帮助熟练的MATLAB®用户成为熟练的NumPy和SciPy用户。

## 一些关键的差异

MATLAB® | NumPy
---|---
在MATLAB®中，基本数据类型是双精度浮点数的多维数组。大多数表达式采用这样的数组并且也返回这样的数据类型，操作这些数组的2-D实例的的方式被设计成或多或少地像线性代数中的矩阵运算一样。 | 在NumPy中，基本类型是多维`数组(array)`。在所有维度(包括2D)上对这些数组的操作都是元素级的操作。对于线性代数，需要使用特定的函数（尽管对于矩阵乘法，可以在python 3.5及以上版本中使用`@`运算符）。
MATLAB®是使用基于1的索引。 使用`a(1)`下标作为元素的初始位置。请参阅[备注](#备注) | Python使用基于0的索引。序列的初始元素使用`a[0]`来查找。
MATLAB的脚本语言是为做线性代数而创建的。基本矩阵操作的语法很好也很简洁，但是用于添加GUI和制作成熟应用程序的API或多或少是事后才想到的。| NumPy基于Python，它从一开始就被设计为一种优秀的通用编程语言。 虽然Matlab的一些数组操作的语法比NumPy更紧凑，但NumPy（由于是Python的附加组件）可以做许多Matlab所不能做的事情，例如正确处理矩阵的堆叠操作。
在 MATLAB® 中，数组具有按值传递的语义，采用延迟的“写时复制”(copy-on-write)模式，以防止在实际需要时创建副本。
使用切片操作是复制数组的部分。 | 在NumPy数组，在NumPy数组中按引用传递语义。切片操作是数组的视图。

## “array” 还是 “matrix”? 我应该选谁?

除了`np.ndarray`之外，NumPy还提供了一种额外的矩阵类型，你可以在一些现有代码中看到这种类型。想用哪一个呢？

由于历史原因，NumPy提供了一种特殊的矩阵类型`np.matrix`，它是ndarray的一个子类，用来做二元运算线性代数运算。您可能会看到一些现有代码中使用它，而不是`np.array`。那么该用哪一个呢？

### 简要的回答

**使用 arrays.**

* 它们是numpy的标准向量/矩阵/张量类型。许多numpy函数返回的是数组，而不是矩阵。
* 元素级运算和线性代数运算之间有着明确的区别。
* 如果你愿意，可以使用标准向量或行/列向量。

直到Python3.5，使用数组类型的唯一缺点是你必须使用`dot`而不是`*`来将两个张量(标量乘积、矩阵向量乘法等)相乘。从 Python3.5 之后，你就可以使用矩阵乘法运算`@`。

综上所述，我们打算最终弃用`matrix`类型。

### 详细的回答

NumPy包含`array`类和`Matrix`类。`array`类旨在成为用于多种数值计算的通用n维数组，而`matrix`类则专门用于促进线性代数计算。实际上，两者之间只有少数几个关键区别。

* 操作符`*`、`@`，函数`dot()`，和`multiply()`：
    * 对于数组，`*`表示元素乘，而`@`表示矩阵乘；它们有相关的函数`multiply()`和`dot()`。（在python 3.5之前，@不存在，必须使用点`dot()`来进行矩阵乘法）。
    * 对于矩阵，`*`表示矩阵乘法，`multiply()`函数用于逐元素乘法。
* 矢量处理（一维数组）
    * 对于数组，向量形状1xN，Nx1和N都是不同的东西。像`A [:,1]`这样的操作返回形状N的一维数组，而不是形状Nx1的二维数组。一维数组上的转置不起任何作用。
    * 对于矩阵，一维数组总是向上转换为1xN或Nx1矩阵（行或列向量）。`A[:,1]`返回形状为Nx1的二维矩阵。
* 处理更高维数组（ndim> 2）
    * 数组对象的维数可以 >2；
    * 矩阵对象总是具有两个维度。
* 便捷的属性
    * 数组有一个.T属性，它返回数据的转置。
    * 矩阵还具有.H，.I和.A属性，分别返回矩阵的共轭转置，反转和`asarray()`。
* 便捷的构造器
    * 数组构造函数接受(嵌套的)Pythonseqxues作为初始化器。如`array([1,2,3], [4,5,6])`。
    * 矩阵构造器另外采用方便的字符串初始化器（传入的参数是字符串）。如`matrix("[1 2 3; 4 5 6]")`。

使用这两种方法有好处也有坏处：

* 数组
    * `:)` 元素乘法很简单：`A*B`
    * `:(` 你必须记住矩阵乘法有它自己的运算符:`@`
    * `:)` 你可以将一维数组视为行或列向量。`A @ v`将`v`视为列向量，`v @ A`将`v`视为行向量。这可以节省输入大量的转置操作。
    * `:)` `array`是默认的NumPy类型，因此它得到的测试最多，并且是使用NumPy的第三方代码最有可能返回的类型
    * `:)` 能处理任意数量维度的数据
    * `:)` 它的语义更接近张量代数（如果读者对此很熟悉的话）
    * `:)` 所有的运算符 (``*``, ``/``, ``+``, ``-``) 都是元素级别的。
    * `:(` `scipy.sparse`中的稀疏矩阵不能很好地与数组交互。

* 矩阵
    * `:\\` 行为更像MATLAB®矩阵。
    * `<:(` 它的维度最大为二维，如果你想保存三维数据，你需要数组或者一个矩阵列表。
    * `<:(` 它的维度最少也是二维，你不能有向量。它们必须将转换为单列或单行矩阵。
    * `<:(` 由于`array`类型是NumPy中的默认类型，因此即使你将矩阵作为参数传入，某些函数也可能返回一个 `array`类型。这种情况不应该发生在NumPy函数中(如果发生了，那就是一个bug)，但是基于NumPy的第三方代码可能不像NumPy那样遵守类型保存。
    * `:)` `A*B`是矩阵乘法，因此线性代数更方便(对于Python >= 3.5 的版本，普通数组与`@`运算符具有同样的方便性)。
    * `<:(` 元素级乘法要求调用乘法函数：`multiply(A,B)`。
    * `<:(` 操作符重载的使用有点不合逻辑：元素级别中`*`运算符并不会生效， 但是`/`运算符却可以。
    * `:)` 与`scipy.sparse`交互更简介一点

因此，使用数组要明智得多。事实上，我们打算最终弃用矩阵类型。


## MATLAB和NumPy粗略的功能对应表

下表粗略的反应了一些常见MATLAB表达式的大致对应关系。这些并不是完全等价的，而是应该作为提示来引导您走向正确的方向。有关更多详细信息，请参阅NumPy函数的内置文档。


在下表中，假设你已经在Python中执行了以下命令：

```python
from numpy import *
import scipy.linalg
```

另外如果下表中的`注释`这一列的内容是和`matrix`有关的话，那么参数一定是二维的形式。

### 一般功能的对应表

MATLAB | NumPy | 注释
---|---|---
help func | info(func) 或 help(func) 或 func? (在 Ipython 中) | 获得函数func的帮助。
which func | see note HELP（译者注：在里的原链接已经失效。） | 找出func定义的位置。
type func | source(func) 或 func?? (在 Ipython 中) | 打印func的源代码(如果不是原生函数的话)。
a && b | a and b | 短路逻辑 AND 运算符 (Python 原生运算符); 仅限标量参数。
a \|\| b | a or b | 短路逻辑 OR 运算符 (Python 原生运算符); 仅限标量参数。
1\*i, 1\*j, 1i, 1j | 1j | 复数。
eps | np.spacing(1) | 数字1和最近的浮点数之间的距离。
ode45 | scipy.integrate.solve_ivp(f) | 将ODE与Runge-Kutta 4,5整合在一起。
ode15s | scipy.integrate.solve_ivp(f, method='BDF') | 用BDF方法整合ODE。

### 线性代数功能对应表

MATLAB | NumPy | 注释
---|---|---
ndims(a) | ndim(a) 或 a.ndim | 获取数组的维数。
numel(a) | size(a) 或 a.size | 获取数组的元素个数。
size(a) | shape(a) 或 a.shape | 求矩阵的“大小”
size(a,n) | a.shape[n-1] | 获取数组a的n维元素数量。(请注意，MATLAB使用基于1的索引，而Python使用基于0的索引，请参见注释索引)。
[ 1 2 3; 4 5 6 ] | array([[1.,2.,3.], [4.,5.,6.]]) | 一个 2x3 矩阵的字面量。
[ a b; c d ] | vstack([hstack([a,b]), hstack([c,d])]) 或 bmat('a b; c d').A | 从快a、b、c和d构造矩阵。
a(end) | a[-1] | 访问1xn矩阵a中的最后一个元素。
a(2,5) | a[1,4] | 访问第二行，第五列中的元素。
a(2,:) | a[1] 或 a[1,:] | 取得a数组第二个元素全部（译者注：第二个元素如果是数组，则返回这个数组）
a(1:5,:) | a[0:5] 或 a[:5] 或 a[0:5,:] | 取得a数组的前五行。
a(end-4:end,:) | a[-5:] | 取得a数组的后五行。
a(1:3,5:9) | a[0:3][:,4:9] | a数组的第1行到第3行和第5到第9列，这种方式只允许读取。
a([2,4,5],[1,3]) | a[ix_([1,3,4],[0,2])] | 行2,4和5以及第1列和第3列。这允许修改矩阵，并且不需要常规切片方式。
a(3:2:21,:) | a[ 2:21:2,:] | a数组每隔一行，从第三行开始，一直到第二十一行。
a(1:2:end,:) | a[ ::2,:] | a数组从第一行开始，每隔一行。
a(end\: -1\:1,:) 或 flipud(a) | a[ ::-1,:] | 反转a数组的顺序。
a([1:end 1],:) | a[r_[:len(a),0]] | 将a数组的第一行的副本添加到数组末尾。
a.' | a.transpose() 或 a.T | a数组的转置。
a' | a.conj().transpose() 或 a.conj().T | a数组的共轭转置。
a * b | a.dot(b) | 矩阵乘法。
a .* b | a * b | 元素相乘。
a./b | a/b | 元素相除。
a.^3 | a**3 | 元素指数运算。
(a>0.5) | (a>0.5) | 其i, th元素为(a_ij > 0.5)的矩阵。Matlab的结果是一个0和1的数组。NumPy结果是一个布尔值false和True的数组。
find(a>0.5) | nonzero(a>0.5) | 找到条件满足 (a > 0.5) 的索引。
a(:,find(v>0.5)) | a[:,nonzero(v>0.5)[0]] | 找到满足条件 向量v > 0.5 的列。
a(:,find(v>0.5)) | a[:,v.T>0.5] |  找到满足条件 列向量v > 0.5 的列。
a(a<0.5)=0 | a[a<0.5]=0 | 元素小于0.5 赋为 0。
a .* (a>0.5) | a * (a>0.5) | （译者注：应该是a乘上a中大于0.5的值的矩阵）
a(:) = 3 | a[:] = 3 | 将所有值设置为相同的标量值
y=x | y = x.copy() | numpy 通过拷贝引用来赋值。
y=x(2,:) | y = x[1,:].copy() | numpy 通过拷贝引用来切片操作。
y=x(:) | y = x.flatten() | 将数组转换为向量(请注意，这将强制拷贝)。
1:10 | arange(1.,11.) 或 r_[1.:11.] 或 r_[1:10:10j] | 创建一个可增长的向量 (参见下面的[备注](#备注)章节)
0:9 | arange(10.) 或 r_[:10.] 或 r_[:9:10j] | 创建一个可增长的向量 (参见下面的[备注](#备注)章节)
[1:10]' | arange(1.,11.)[:, newaxis] | 创建一个列向量。
zeros(3,4) | zeros((3,4)) | 创建一个全是0的填充的 3x4 的64位浮点类型的二维数组。
zeros(3,4,5) | zeros((3,4,5)) | 创建一个全是0的填充的 3x4x5 的64位浮点类型的三维数组。
ones(3,4) | ones((3,4)) | 创建一个全是 1 的填充的 3x4 的64位浮点类型的二维数组。
eye(3) | eye(3) | 创建一个3x3恒等矩阵。
diag(a) | diag(a) | 创建a数组的对角元素向量。
diag(a,0) | diag(a,0) | 创建方形对角矩阵，其非零值是a的所有元素。
rand(3,4) | random.rand(3,4) | 创建一个随机的 3x4 矩阵
linspace(1,3,4) | linspace(1,3,4) | 创建4个等间距的样本，介于1和3之间。
[x,y]=meshgrid(0:8,0:5) | mgrid[0:9.,0:6.] 或 meshgrid(r_[0:9.],r_[0:6.] | 两个2维数组：一个是x值，另一个是y值。
 \- | ogrid[0:9.,0:6.] 或 ix_(r_[0:9.],r_[0:6.] | 最好的方法是在一个网格上执行函数。
[x,y]=meshgrid([1,2,4],[2,4,5]) | meshgrid([1,2,4],[2,4,5]) |  - 
 - | ix_([1,2,4],[2,4,5]) | 最好的方法是在网格上执行函数。
repmat(a, m, n) | tile(a, (m, n)) | 通过n份a的拷贝创建m。
[a b] | concatenate((a,b),1) 或 hstack((a,b)) 或 column_stack((a,b)) or c_[a,b] | 连接a和b的列。
[a; b] | concatenate((a,b)) 或 vstack((a,b)) 或 r_[a,b] | 连接a和b的行。
max(max(a)) | a.max() | 取a数组的中的最大元素（对于matlab来说，ndims(a) <= 2）
max(a) | a.max(0) | 求各列的最大值。
max(a,[],2) | a.max(1) | 求各行最大值。
max(a,b) | maximum(a, b) | 比较a和b元素，并返回每对中的最大值。
norm(v) | sqrt(dot(v,v)) 或 np.linalg.norm(v) | 向量v的L2范数
a & b | logical_and(a,b) | 逐元素使用 AND 运算符 (NumPy ufunc) (参见下面的[注释](#note)章节)
a | b | logical_or(a,b) | 逐元素使用 OR 运算符 (NumPy ufunc) (参见下面的[注释](#note)章节)
bitand(a,b) | a & b | 按位AND运算符  (Python原生 和 NumPy ufunc)
bitor(a,b) | a | b | 按位OR运算符 (Python原生 和 NumPy ufunc)
inv(a) | linalg.inv(a) | 矩阵a的逆运算
pinv(a) | linalg.pinv(a) | 矩阵a的反逆运算
rank(a) | linalg.matrix_rank(a) | 二维数组或者矩阵的矩阵rank。
a\b | 如果a是方形矩阵 linalg.solve(a,b) ，否则：linalg.lstsq(a,b)  | 对于x，x = b的解
b/a | Solve a.T x.T = b.T instead | 对于x，x a = b的解
[U,S,V]=svd(a) | U, S, Vh = linalg.svd(a), V = Vh.T | a数组的奇值分解
chol(a) | linalg.cholesky(a).T | 矩阵的cholesky分解（matlab中的chol(a)返回一个上三角矩阵，但linalg.cholesky(a)返回一个下三角矩阵）
[V,D]=eig(a) | D,V = linalg.eig(a) | a数组的特征值和特征向量
[V,D]=eig(a,b) | V,D = np.linalg.eig(a,b) | a，b数组的特征值和特征向量
[V,D]=eigs(a,k) | - | 找到a的k个最大特征值和特征向量
[Q,R,P]=qr(a,0) | Q,R = scipy.linalg.qr(a) | Q，R 的分解
[L,U,P]=lu(a) | L,U = scipy.linalg.lu(a) or LU,P=scipy.linalg.lu_factor(a) | LU 分解 (注意: P(Matlab) == transpose(P(numpy)) )
conjgrad | scipy.sparse.linalg.cg | 共轭渐变求解器
fft(a) | fft(a) | a数组的傅立叶变换
ifft(a) | ifft(a) | a的逆逆傅里叶变换
sort(a) | sort(a) or a.sort() | 对矩阵或者数组进行排序
[b,I] = sortrows(a,i) | I = argsort(a[:,i]), b=a[I,:] | 对矩阵或数组的行进行排序
regress(y,X) | linalg.lstsq(X,y) | 多线性回归
decimate(x, q) | scipy.signal.resample(x, len(x)/q) | 采用低通滤波的下采样
unique(a) | unique(a) |  -
squeeze(a) | a.squeeze() | -  

## 注释

**子矩阵**: 可以使用ix_命令使用索引列表完成对子矩阵的分配。例如，对于2d阵列a，有人可能会这样做： `ind=[1,3]; a[np.ix_(ind,ind)]+=100`.

**帮助方面**: 没有直接等同于MATLAB的`which`的命令，但是命令`help`和`source`通常会列出函数所在的文件名。Python还提供了一个检查模块(do `import inspect`)，该模块提供了一个经常被使用到的`getfile``方法。

**索引方面**: MATLAB® 使用基于1的索引，因此序列的初始元素具有索引1。Python使用基于0的索引，因此数组的初始元素的索引为0。之所以有如此天壤之别，是因为它们各有优缺点。使用基于1的索引是和常见的人类语言用法是一致的，前者是序列的 “第一个” 元素的索引是1。而后者基于0的索引简化了索引。另见pro.dr的文本。

**范围**: 在 MATLAB® 中，`0:5`既可以用作范围文字索引，也可以用作“片”索引(括号内)；然而，在Python中，像`0:5`这样的结构只能用作片索引(方括号内)。因此，创建了有点奇怪的`r_`对象，以允许numpy具有类似的简洁范围构造机制。请注意，`r_`不像函数或构造函数那样被调用，而是使用方括号进行索引，这允许在参数中使用Python的切片语法。

**逻辑运算**: & 或者 | 在NumPy中是按位的 AND/OR，而在Matlab中 & 和 | 是逻辑 AND/OR。对于任何有丰富编程经验的人来说，这一区别应该是显而易见的。这两种方法表面上看起来是一样的，但也有重要的区别。如果你要使用Matlab的& 或 | 操作符，则应该使用NumPy函数的logicaland/logicalor。Matlab 的运算符 与 NumPy 的 & 和 | 运算符的显著区别是：

非逻辑{0,1}输入：NumPy的输出是输入的按位AND。 Matlab将任何非零值视为1并返回逻辑AND。 例如，NumPy中的（3&4）是0，而在Matlab中，3和4都被认为是逻辑真，而（3&4）返回1。
优先级：NumPy的＆运算符优先于<和>之类的逻辑运算符; Matlab是相反的。
如果你知道你有布尔参数，你可以使用NumPy的按位运算符，但要注意括号，如：z =（x> 1）＆（x <2）。 缺少NumPy运算符形式的logical_and和logical_or是Python设计的一个不幸结果。

**重塑与线性索引**: Matlab总是允许使用标量或线性索引访问多维数组，而NumPy则不然。 线性索引在Matlab程序中很常见，例如：矩阵上的find()函数就会返回这在类型，而NumPy的查找行为则不同。 在转换Matlab代码时，可能需要首先将矩阵重塑为线性序列，执行一些索引操作然后再重塑。 由于重塑（通常）会在同一存储上生成视图，因此应该可以相当有效地执行此操作。 请注意，在NumPy中重塑使用的扫描顺序默认为'C'顺序，而Matlab使用Fortran顺序。 如果你只是简单地转换为线性序列，那么这无关紧要。 但是如果要从依赖于扫描顺序的Matlab代码转换重构，那么这个Matlab代码：z = reshape(x, 3,4); 应该在NumPy中变成z = x.reshape(3,4, order ='F').copy()。

## 自定义环境

在MATLAB®中，可用于自定义环境的主要工具是修改你喜欢的功能的位置的搜索路径（环境变量）。你可以将这些自定义放入启动脚本中，MATLAB将在启动时运行该脚本。

NumPy，或者更确切地说是Python，具有类似的功能。

- 要修改Python搜索路径以包含自己模块的位置，请定义`PYTHONPATH`的环境变量。
- 要在启动交互式Python解释器时执行特定的脚本文件，请定义“PYTHONSTARTUP”环境变量以包含启动脚本的名称。

与MATLAB®不同，可以立即调用路径上的任何内容，使用Python，你需要先执行“import”语句，以使特定文件中的函数可访问。

例如，你可能会创建一个类似于下面代码的启动脚本（注意：这只是一个示例，而不是“最佳做法”的声明）：

```python
# Make all numpy available via shorter 'num' prefix
import numpy as num
# Make all matlib functions accessible at the top level via M.func()
import numpy.matlib as M
# Make some matlib functions accessible directly at the top level via, e.g. rand(3,3)
from numpy.matlib import rand,zeros,ones,empty,eye
# Define a Hermitian function
def hermitian(A, **kwargs):
    return num.transpose(A,**kwargs).conj()
# Make some shortcuts for transpose,hermitian:
#    num.transpose(A) --> T(A)
#    hermitian(A) --> H(A)
T = num.transpose
H = hermitian
```

## 链接

有关另一个MATLAB®/ NumPy交叉引用， 请参见[http://mathesaurus.sf.net/](http://mathesaurus.sf.net/)。

可以在主题[软件页面](http://scipy.org/topical-software.html).中找到用于python科学工作的更多工具列表。

MATLAB®和SimuLink®是The MathWorks的注册商标。