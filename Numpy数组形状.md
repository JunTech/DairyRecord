---
title: Numpy数组形状
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
summary: numpy數組形狀
categories: python
tags: Numpy
keywords: numpy
abbrlink: 377f5f44
date: 2019-09-27 11:45:17
---

# 数组形状

```python
%pylab
```

```
Using matplotlib backend: Qt4Agg
Populating the interactive namespace from numpy and matplotlib
```

## 修改数组的形状

## 修改数组的形状

```python
a = arange(6)
a
```



```
array([0, 1, 2, 3, 4, 5])

```

将形状修改为2乘3：
将形状修改为2乘3：

```python
a.shape = 2,3
a
```



```
array([[0, 1, 2],
       [3, 4, 5]])

```

与之对应的方法是 `reshape` ，但它不会修改原来数组的值，而是返回一个新的数组：
与之对应的方法是 `reshape` ，但它不会修改原来数组的值，而是返回一个新的数组：

```python
a.reshape(3,2)
```



```
array([[0, 1],
       [2, 3],
       [4, 5]])


```

```python
​```python
a
```



```
array([[0, 1, 2],
       [3, 4, 5]])

```

`shape` 和 `reshape` 方法不能改变数组中元素的总数，否则会报错：
`shape` 和 `reshape` 方法不能改变数组中元素的总数，否则会报错：

```python
a.reshape(4,2)
```

```
---------------------------------------------------------------------------

ValueError                                Traceback (most recent call last)

<ipython-input-6-1a35a76a1693> in <module>()
----> 1 a.reshape(4,2)


ValueError: total size of new array must be unchanged
```

## 使用 newaxis 增加数组维数

## 使用 newaxis 增加数组维数

```python
a = arange(3)
shape(a)
```



```
(3L,)


```

```python
​```python
y = a[newaxis, :]
shape(y)
```



```
(1L, 3L)

```

根据插入位置的不同，可以返回不同形状的数组：
根据插入位置的不同，可以返回不同形状的数组：

```python
y = a[:, newaxis]
shape(y)
```



```
(3L, 1L)

```

插入多个新维度：
插入多个新维度：

```python
y = a[newaxis, newaxis, :]
shape(y)
```



```
(1L, 1L, 3L)

```

## squeeze 方法去除多余的轴

## squeeze 方法去除多余的轴

```python
a = arange(6)
a.shape = (2,1,3)
```

```python
b = a.squeeze()
b.shape
```



```
(2L, 3L)

```

squeeze 返回一个将所有长度为1的维度去除的新数组。
squeeze 返回一个将所有长度为1的维度去除的新数组。

## 数组转置

使用 `transpose` 返回数组的转置，本质上是将所有维度反过来：

```python
a
```



```
array([[[0, 1, 2]],

       [[3, 4, 5]]])

```

对于二维数组，这相当于交换行和列：
对于二维数组，这相当于交换行和列：

```python
a.transpose()
```



```
array([[[0, 3]],

       [[1, 4]],

       [[2, 5]]])

```

或者使用缩写属性：
或者使用缩写属性：

```python
a.T
```



```
array([[[0, 3]],

       [[1, 4]],

       [[2, 5]]])

```

注意：
注意：

- 对于复数数组，转置并不返回复共轭，只是单纯的交换轴的位置
- 转置可以作用于多维数组

```python
a = arange(60)
a
```



```
array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16,
       17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33,
       34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50,
       51, 52, 53, 54, 55, 56, 57, 58, 59])



```

```python
​```python
a.shape = 3,4,5
a
```



```
array([[[ 0,  1,  2,  3,  4],
        [ 5,  6,  7,  8,  9],
        [10, 11, 12, 13, 14],
        [15, 16, 17, 18, 19]],

       [[20, 21, 22, 23, 24],
        [25, 26, 27, 28, 29],
        [30, 31, 32, 33, 34],
        [35, 36, 37, 38, 39]],

       [[40, 41, 42, 43, 44],
        [45, 46, 47, 48, 49],
        [50, 51, 52, 53, 54],
        [55, 56, 57, 58, 59]]])



```

```python
​```python
b = a.T
b.shape
```



```
(5L, 4L, 3L)


```

转置只是交换了轴的位置。
转置只是交换了轴的位置。

另一方面，转置返回的是对原数组的另一种view，所以改变转置会改变原来数组的值。

```python
a = arange(6)
a.shape = (2,3)
a
```



```
array([[0, 1, 2],
       [3, 4, 5]])


```

修改转置：
修改转置：

```python
b = a.T
b[0,1] = 30
```

原数组的值也改变：

```python
a
```



```
array([[ 0,  1,  2],
       [30,  4,  5]])


```

## 数组连接

## 数组连接

有时我们需要将不同的数组按照一定的顺序连接起来：

```
concatenate((a0,a1,...,aN), axis=0)

```

注意，这些数组要用 `()` 包括到一个元组中去。

除了给定的轴外，这些数组其他轴的长度必须是一样的。

```python
x = array([
        [0,1,2],
        [10,11,12]
    ])
y = array([
        [50,51,52],
        [60,61,62]
    ])
print x.shape
print y.shape
```

```
(2L, 3L)
(2L, 3L)

```

默认沿着第一维进行连接：
默认沿着第一维进行连接：

```python
z = concatenate((x,y))
z
```



```
array([[ 0,  1,  2],
       [10, 11, 12],
       [50, 51, 52],
       [60, 61, 62]])



```

```python
​```python
z.shape
```



```
(4L, 3L)


```

沿着第二维进行连接：
沿着第二维进行连接：

```python
z = concatenate((x,y), axis=1)
z
```



```
array([[ 0,  1,  2, 50, 51, 52],
       [10, 11, 12, 60, 61, 62]])



```

```python
​```python
z.shape
```



```
(2L, 6L)


```

注意到这里 `x` 和 `y` 的形状是一样的，还可以将它们连接成三维的数组，但是 `concatenate` 不能提供这样的功能，不过可以这样：
注意到这里 `x` 和 `y` 的形状是一样的，还可以将它们连接成三维的数组，但是 `concatenate` 不能提供这样的功能，不过可以这样：

```python
z = array((x,y))
```

```python
z.shape
```



```
(2L, 2L, 3L)


```

事实上，**Numpy**提供了分别对应这三种情况的函数：
事实上，**Numpy**提供了分别对应这三种情况的函数：

- vstack
- hstack
- dstack

```python
vstack((x, y)).shape
```



```
(4L, 3L)



```

```python
​```python
hstack((x, y)).shape
```



```
(2L, 6L)



```

```python
​```python
dstack((x, y)).shape
```



```
(2L, 3L, 2L)


```

## Flatten 数组

## Flatten 数组

`flatten` 方法的作用是将多维数组转化为1维数组：

```python
a = array([[0,1],
           [2,3]])
b = a.flatten()
b
```



```
array([0, 1, 2, 3])


```

返回的是数组的复制，因此，改变 `b` 并不会影响 `a` 的值：
返回的是数组的复制，因此，改变 `b` 并不会影响 `a` 的值：

```python
b[0] = 10
print b
print a
```

```
[10  1  2  3]
[[0 1]
 [2 3]]

```

## flat 属性

## flat 属性

还可以使用数组自带的 `flat` 属性：

```python
a.flat
```



```
<numpy.flatiter at 0x3d546a0>


```

`a.flat` 相当于返回了所有元组组成的一个迭代器：
`a.flat` 相当于返回了所有元组组成的一个迭代器：

```python
b = a.flat
```

```python
b[0]
```



```
0


```

但此时修改 `b` 的值会影响 `a` ：
但此时修改 `b` 的值会影响 `a` ：

```python
b[0] = 10
print a
```

```
[[10  1]
 [ 2  3]]


```

```python
​```python
a.flat[:]
```



```
array([10,  1,  2,  3])


```

## ravel 方法

## ravel 方法

除此之外，还可以使用 `ravel` 方法，`ravel` 使用高效的表示方式：

```python
a = array([[0,1],
           [2,3]])
b = a.ravel()
b
```



```
array([0, 1, 2, 3])


```

修改 `b` 会改变 `a` ：
修改 `b` 会改变 `a` ：

```python
b[0] = 10
a
```



```
array([[10,  1],
       [ 2,  3]])


```

但另一种情况下：
但另一种情况下：

```python
a = array([[0,1],
           [2,3]])
aa = a.transpose()
b = aa.ravel()
b
```



```
array([0, 2, 1, 3])



```

```python
​```python
b[0] = 10
```

```python
aa
```



```
array([[0, 2],
       [1, 3]])



```

```python
​```python
a
```



```
array([[0, 1],
       [2, 3]])


```

可以看到，在这种情况下，修改 `b` 并不会改变 `aa` 的值，原因是我们用来 `ravel` 的对象 `aa` 本身是 `a` 的一个view。
可以看到，在这种情况下，修改 `b` 并不会改变 `aa` 的值，原因是我们用来 `ravel` 的对象 `aa` 本身是 `a` 的一个view。

## atleast_xd 函数

保证数组至少有 `x` 维：

```python
x = 1
atleast_1d(x)
```



```
array([1])



```

```python
​```python
a = array([1,2,3])
b = atleast_2d(a)
b.shape
```



```
(1L, 3L)



```

```python
​```python
b
```



```
array([[1, 2, 3]])



```

```python
​```python
c = atleast_3d(b)
```

```python
c.shape
```



```
(1L, 3L, 1L)


```

`x` 可以取值 1，2，3。
`x` 可以取值 1，2，3。

在**Scipy**库中，这些函数被用来保证输入满足一定的条件：“ 

|用法|**Scipy**中出现次数|
|-|-|
|value.flaten() <br> value.flat <br>  value.ravel() |     ~2000次
| atleast_1d(value) <br> atleast_2d(value)  |~700次
| asarray(value)    |~4000次