# Python3基础面试题

> 本文很多题目总结自廖雪峰的Python3免费教程，感谢廖老师的无私奉献：
>
> 教程链接：https://www.liaoxuefeng.com/wiki/1016959663602400/

### 0x0001

Q：以下代码执行后，最终会输出`ABC`还是`XYZ`，为什么？

```python
a = 'ABC'
b = a
a = 'XYZ'
print(b)
```

A：最终输出的是`ABC`,我们看一下上面代码执行过程中，Python解析器都做了什么。

首先执行`a = 'ABC'`，解释器创建了字符串`'ABC'`和变量`a`，并把`a`指向`'ABC'`：

![py-var-code-1](resource/000101.png)

然后执行`b = a`，解释器创建了变量`b`，并把`b`指向`a`指向的字符串`'ABC'`：

![py-var-code-2](resource/000102.png)

执行`a = 'XYZ'`，解释器创建了字符串'XYZ'，并把`a`的指向改为`'XYZ'`，但`b`并没有更改：

![py-var-code-3](resource/000103.png)

所以，最后打印变量`b`的结果自然是`'ABC'`了。



### 0x0002

Q：下面代码执行后会输出什么结果？

```python
>>> r = 2.5
>>> s = 3.14 * r ** 2
>>> print(f'The area of a circle with radius {r} is {s:.2f}')
```

A：因为代码中使用的是`f`开头的字符串，称之为`f-string`，它和普通字符串不同之处在于，字符串如果包含`{xxx}`，就会以对应的变量替换。

上述代码中，`{r}`被变量`r`的值替换，`{s:.2f}`被变量`s`的值替换，并且`:`后面的`.2f`指定了格式化参数（即保留两位小数），因此，`{s:.2f}`的替换结果是`19.62`，所以最后输出的结果是：

```shell
The area of a circle with radius 2.5 is 19.62
```



### 0x0003

Q：Python中的元组（tuple）一旦创建就不可变了，那么我执行下面代码会报错吗？

```python
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
```

A：不会报错。这个元组定义的时候有3个元素，分别是`'a'`，`'b'`和一个list。

我们先看看定义元组t后内存中是这样的：

![tuple-0](https://www.liaoxuefeng.com/files/attachments/923973516787680/0)

当我们把list的元素`'A'`和`'B'`修改为`'X'`和`'Y'`后，内存中的是这个样子的：

![tuple-1](https://www.liaoxuefeng.com/files/attachments/923973647515872/0)

表面上看，tuple的元素确实变了，但其实变的不是tuple的元素，而是list的元素。tuple一开始指向的list并没有改成别的list，所以，tuple所谓的“不变”是说，tuple的每个元素，指向永远不变。即指向`'a'`，就不能改成指向`'b'`，指向一个list，就不能改成指向其他对象，但指向的这个list本身是可变的。



### 0x0004

Q：Python中可以使用哪些数据类型作为dict字典的key？

A：Python中的dict底层是hash算法，所以必须使用不可变的值作为key，那么就只能使用字符串、整数、浮点数、布尔值、tuple元组作为key。我们分别测试一下使用不同的类型作为key：

```python
>>> a[1] = 1
>>> a[2] = "2"
>>> a[True] = True
>>> a[False] = False
>>> a[0] = 0
>>> t = (1,2)
>>> a[t] = 't'
>>> d = {'a': 'a'}
>>> a[d] = 'd'
>>> a[3.14] = 3.14
>>> a[3] = 3
>>> a[3.0] = 3.0
>>> print(a)
```

最后控制台显示

```python
{1: True, 2: '2', False: 0, (1, 2): 't', 3.14: 3.14, 3: 3.0}
```

由此可见当布尔值True作为key是，Python解析器会认为是整数1，False会被认为是0。而且整数3和浮点数3.0被认为是同样的key。

需要注意的是，tuple虽然可以作为key，但是tuple中一旦包含了可变的对象，那么同样不可以作为key，比如说我们创建一个tuple中包含一个list，那么如果将这个tuple作为key是会报错的：

```python
>>> t2 = (1,2,[3])
>>> a[t2] = 't2'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'list'
```



### 0x0005

Q：下面代码执行后最终控制台输出什么？讲讲为什么会输出这样的结果。

```python
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs

f1, f2, f3 = count()
print('f1: ', f1())
print('f2: ', f2())
print('f3: ', f3())
```

A：上面代码每次循环都会创建一个闭包函数，每个函数中都引用了变量`i`，然而3个函数并没后立即执行，而是全部返回以后再执行的。在执行的 时候他们所引用的变量`i`已经变成了`3`，所以最终实际三个函数执行时的结果都是输出的`3 * 3`，即会输出以下结果：

```python
f1:  9
f2:  9
f3:  9
```



### 0x0006

Q：请利用闭包返回一个计数器函数`createCounter`，每次调用它返回递增后的整数。最终执行以下测试的时候，控制台要输出`1 2 3 4 5`。请实现该函数后讲讲其中的执行过程和需要注意的地方：

```python
counterA = createCounter()
print(counterA(), counterA(), counterA(), counterA(), counterA()) # 1 2 3 4 5
```

A：实现的代码如下：

```python
def createCounter():
    result = 0
    def counter():
        nonlocal result
        result = result + 1
        return result
    return counter
```

首先需要在`createCounter`中创建一个变量`result`，然后在返回的闭包函数`counter`中使用`nonlocal`关键字来引用`result`这个变量，这样就能做到闭包函数中每次使用的是外层中的`result`中的值，才能做到不断自增`+1`的效果。`nonlocal`关键字是Python3中新增加的关键字，因此注意Python2中无法使用。



