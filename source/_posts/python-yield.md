---
title: python中的yield关键字
date: 2019-07-02 16:47:06
categories: python
tags: [python]
---
今天在看别人的代码时遇到了yield关键字，当时学python的时候学的并不扎实，现在做个总结

### 1.yield example
一个网上经常列举的yield的例子：

```python
def node._get_child_candidates(self, distance, min_dist, max_dist):
    if self._leftchild and distance - max_dist < self._median:
        yield self._leftchild
    if self._rightchild and distance + max_dist >= self._median:
        yield self._rightchild
```

下面是具体调用时的执行

```python
result, candidates = list(), [self]
while candidates:
    node = candidates.pop()
    distance = node._get_dist(obj)
    if distance <= max_dist and distance >= min_dist:
        result.extend(node._values)
    candidates.extend(node._get_child_candidates(distance, min_dist, max_dist))
return result
```

调用_get_child_candidates时，返回了一个child的合集（list），yield是如何生成这个对象的呢？
### 2.先来看一下可迭代对象

当你建立了一个列表，你可以逐项地读取这个列表，这叫做一个可迭代对象:

```python
>>> mylist = [1, 2, 3]
>>> for i in mylist :
...    print(i)
1
2
3
```

`mylist`是一个可迭代的对象。当你使用一个列表生成式来建立一个列表的时候，同样生成了一个可迭代的对象:

```python
>>> mylist = [x for x in range(3)]
>>> for i in mylist :
...    print(i)
0
1
2
```
可以通过`for`循环读取的对象就是一个迭代器，其元素在遍历访问时均存储在了内存中，如果要大量访问数据的话，迭代器的方式是很占用资源的

### 3.生成器

生成器是可以迭代的对象，但是你**只需读取一次**，它可以在调用时实时生成数据，而不是将数据都存放在内存中。

```python
>>> mygenerator = (x for x in range(3))
>>> for i in mygenerator :
...    print(i)
0
1
2
```

虽然只是把(换成了[，但是对于生成器而言，只能迭代一次，而且也从迭代器变成了生成器。

### 4.回到yield关键字

`yield`关键字在调用后会返回一个类似于`mygenerator`的生成器，类似于返回生成器的`return`关键字。

```python
>>> def createGenerator() :
...    mylist = range(3)
...    for i in mylist :
...        yield i*i
...
>>> mygenerator = createGenerator() # create a generator
>>> print(mygenerator) # mygenerator is an object!
<generator object createGenerator at 0xb7555c34>
>>> for i in mygenerator:
...     print(i)
0
1
4
```

在执行`for i in mygenerator`时，到达`yield`关键字时，返回`yield`后的值作为第一次迭代的返回值. 然后，每次执行这个函数都会继续返回定义的迭代值，直到没有可以返回的。带有yield的函数不仅仅只用于for循环中，而且可用于某个函数的参数，只要这个函数的参数允许迭代参数。比如`array.extend`函数，它的原型是`array.extend(iterable)`。
此处应注意的是，生成器的方法虽然可以调用多次，获取多个迭代结果，但生成器只会实例化一次，既实例化后的生成器可以通过变量等来控制生成器的生成与穷尽。可参考如下代码：

```python
>>> class Bank(): # let's create a bank, building ATMs
...    crisis = False
...    def create_atm(self) :
...        while not self.crisis :
...            yield "$100"
>>> hsbc = Bank() # when everything's ok the ATM gives you as much as you want
>>> corner_street_atm = hsbc.create_atm()
>>> print(corner_street_atm.__next__()())
$100
>>> print(corner_street_atm.__next__()())
$100
>>> print([corner_street_atm.__next__()() for cash in range(5)])
['$100', '$100', '$100', '$100', '$100']
>>> hsbc.crisis = True # crisis is coming, no more money!
>>> print(corner_street_atm.__next__()())
<type 'exceptions.StopIteration'>
>>> wall_street_atm = hsbc.create_atm() # it's even true for new ATMs
>>> print(wall_street_atm.__next__()())
<type 'exceptions.StopIteration'>
>>> hsbc.crisis = False # trouble is, even post-crisis the ATM remains empty
>>> print(corner_street_atm.__next__()())
<type 'exceptions.StopIteration'>
>>> brand_new_atm = hsbc.create_atm() # build a new one to get back in business
>>> for cash in brand_new_atm :
...    print cash
$100
$100
$100
$100
$100
$100
$100
$100
$100
...
```

### 5.迭代器的操作
生成器只能实例化一次，这就为我们的复用造成一些麻烦，如果需要两个一模一样但是相互独立的生成器怎么办呢？
itertools提供了很多特殊的迭代方法，包括复制一个迭代器，串联迭代器，把嵌套的列表分组等等等，这个类从一定程度上解决了生成器的复用问题。
如果想要动态变化生成器的内容呢？生成器本身除去next方法外，还有一个send(msg)的方法，send(msg)与next()的区别在于send可以传递参数给yield表达式，这时传递的参数会作为yield表达式的值，而yield的参数是返回给调用者的值。比如函数中有一个yield赋值，`a = yield 5`，第一次迭代到这里会返回5，a还没有赋值。第二次迭代时，使用`.send(10)`，那么，就是强行修改yield 5表达式的值为10，本来是5的，而现在`a=10`。可以认为，next()等同于send(None)。

> 此处应注意的是，第一次调用时必须先next()或send(None)，否则会报错，因为这时候没有上一个yield值。

### 6.关于迭代器的内部原理
迭代是一个实现可迭代对象(实现的是`__iter__()`方法)和迭代器(实现的是`__next__()`方法)的过程。可迭代对象是你可以从其获取到一个迭代器的任一对象。迭代器是那些允许你迭代可迭代对象的对象。


参考文献：https://pyzh.readthedocs.io/en/latest/the-python-yield-keyword-explained.html#yield
