高级特性
-----
## 1.切片
silce操作符取一个list或tuple的部分元素:
```python
>>> L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']#定义一个list
>>> L[0:3]	#L[0:3]表示，从索引0开始取，直到索引3为止，但不包括索引3。
['Michael', 'Sarah', 'Tracy']
>>> L[:3]	#如果第一个索引是0，还可以省略
['Michael', 'Sarah', 'Tracy']
>>> L[1:3]	#从索引 1 开始，取出 2 个元素出来
['Sarah', 'Tracy']
>>> L[-2:]	#倒数切片,倒数第一个元素索引是-1,取倒数第二个到最后一个
['Bob', 'Jack']
>>> L[-2:-1]#取倒数第二个
['Bob']
>>> L[:5:2]#前五个元素,每两个取一个
['Michael', 'Tracy', 'Jack']
>>> L[::3]##所有元素,每三个取一个
['Michael', 'Bob']
>>> L[:]#什么都不写，只写[:]就可以原样复制一个list
['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
```
字符串'xxx'也可以看成是一种 list，每个元素就是一个字符。因此，字符串也可以用切片操作，只是操作结果仍是字符串：
```python
>>> 'ABCDEFG'[:3]
'ABC'
>>> 'ABCDEFG'[::2]
'ACEG'
```

## 2.迭代-Iteration
- 如果给定一个 list 或 tuple，我们可以通过for循环来遍历这个 list 或 tuple，这种遍历我们称为迭代（Iteration）.
- 在 Python 中，迭代是通过`for ... in`来完成的.
- 几个例子:
```python
>>> d = {'a': 1, 'b': 2, 'c': 3}	#dict的迭代
>>> for key in d:	#因为 dict 的存储不是按照 list 的方式顺序排列，
...     print(key)	#所以，迭代出的结果顺序很可能不一样。
...	#默认情况下，dict 迭代的是 key。
a	#如果要迭代 value，可以用for value in d.values()
c	#如果要同时迭代 key 和 value，可以用for k, v in d.items()
b
```
```python
>>> for ch in 'ABC':	#字符串也是可迭代对象
...     print(ch)
...
A
B
C
```
- 判断对象是否可以进行迭代:通过`collections`模块的`Iterable`类型进行判断:
```python
>>> from collections import Iterable
>>> isinstance('abc', Iterable) 
True
>>> isinstance([1,2,3], Iterable) 
True
>>> isinstance(123, Iterable) 
False
```

## 3.列表生成式
列表生成式即**List Comprehensions**，是 Python 内置的非常简单却强大的可以用来创建 list 的生成式。
例如:使用列表生产式生成list[1x1, 2x2, 3x3, ..., 10x10]
```python	
>>> [x * x for x in range(1, 11)]#把要生成的元素x * x放到前面，后面跟for循环
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```
for 循环后面还可以加上 if 判断，这样我们就可以筛选出仅偶数的平方：
```python
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]
```
还可以使用两层循环，可以生成全排列：
```python
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```
列表生成式也可以使用两个变量来生成 list：
```python
>>> d = {'x': 'A', 'y': 'B', 'z': 'C' }
>>> [k + '=' + v for k, v in d.items()]
['y=B', 'x=A', 'z=C']
```
把一个 list 中所有的字符串变成小写：
```python
>>> L = ['Hello', 'World', 'IBM', 'Apple']
>>> [s.lower() for s in L]
['hello', 'world', 'ibm', 'apple']
```
## 4.生成器
如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的 list，从而节省大量的空间。在 Python 中，这种一边循环一边计算的机制，称为生成器：**generator**。
- **`generator `的工作原理:**在for循环的过程中不断计算出下一个元素，并在适当的条件结束for循环。对于函数改成的 generator 来说，遇到return语句或者执行到函数体最后一行语句，就是结束 generator 的指令，for循环随之结束。
- 注意区分普通函数和 generator 函数，普通函数调用直接返回结果,generator 函数的 “调用” 实际返回一个 generator 对象.
- 产生generator法一:把一个列表生成式的[]改成()，就创建了一个 generator
```python
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>
>>> for n in g:	#通过for循环来迭代,遍历生成器.
>>> ...     print(n)
```
创建L和g的区别仅在于最外层的[]和()，L是一个 list，而g是一个 generator。
- 产生generator法二:用函数来实现,
```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b	#斐波拉契数列生成器
        a, b = b, a + b
        n = n + 1
    return 'done'
```
如果一个函数定义中包含yield关键字，那么这个函数就不再是一个普通函数，而是一个 generator:
```python
>>> f = fib(6)
>>> f
<generator object fib at 0x104feaaa0>
```
函数是顺序执行，遇到return语句或者最后一行函数语句就返回。而变成 generator 的函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回的yield语句处继续执行。

把函数改成 generator 后，我们基本上从来不会用next()来获取下一个返回值，而是直接使用for循环来迭代：

```python
>>> for n in fib(6):
...     print(n)
...
```
## 5.迭代器
可以直接作用于for循环(就是迭代iteration)的数据类型有以下几种:
- 一类是集合数据类型,如list/tuple/dict/set/str等;
- 一类是`generator`,包括生成器和带`yield`的`generator function`.
以上可以直接作用于for循环的对象统称为可迭代对象,可以使用**`isinstance()`**判断一个对象是否是**`Iteratable`对象:**
```python
>>> from collections import Iterable
>>> isinstance([], Iterable)
True
>>> isinstance({}, Iterable)
True
>>> isinstance('abc', Iterable)
True
>>> isinstance((x for x in range(10)), Iterable)
True
>>> isinstance(100, Iterable)
False
```
而生成器不但可以作用于for循环，还可以被next()函数不断调用并返回下一个值，直到最后抛出StopIteration错误表示无法继续返回下一个值了。

**迭代器(iterator):**可以被next()函数调用并不断返回下一个值的对象.

同样地,可以使用**`isinstance()`**判断一个对象是否是**`Iterator`对象:**
```python
>>> from collections import Iterator
>>> isinstance((x for x in range(10)), Iterator)
True
>>> isinstance([], Iterator)
False
>>> isinstance({}, Iterator)
False
>>> isinstance('abc', Iterator)
False
```
生成器都是Iterator对象，但list、dict、str虽然是Iterable，却不是Iterator。

把list、dict、str等Iterable变成Iterator可以使用iter()函数：
```python
>>> isinstance(iter([]), Iterator)
True
>>> isinstance(iter('abc'), Iterator)
True
```
Python 的Iterator对象表示的是一个数据流，Iterator 对象可以被next()函数调用并不断返回下一个数据，直到没有数据时抛出StopIteration错误。可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度，只能不断通过next()函数实现按需计算下一个数据，所以Iterator的计算是惰性的，只有在需要返回下一个数据时它才会计算。Iterator甚至可以表示一个无限大的数据流，例如全体自然数。而使用 list 是永远不可能存储全体自然数的。

**以上关于迭代器的小结:**
- 凡是可作用于for循环的对象都是Iterable类型；
- 凡是可作用于next()函数的对象都是Iterator类型，它们表示一个惰性计算的序列；
- 集合数据类型如list、dict、str等是Iterable但不是Iterator，不过可以通过iter()函数获得一个Iterator对象。
- Python 的for循环本质上就是通过不断调用next()函数实现的，例如：
```python
for x in [1, 2, 3, 4, 5]:
    pass
```
实际上完全等价于：
```python
it = iter([1, 2, 3, 4, 5])

while True:
    try:
        
        x = next(it)
    except StopIteration:
        
        break
```
