## 1.定义函数
在 Python 中，定义一个函数要使用`def`语句，依次写出函数名、括号、括号中的参数和冒号`:`，然后，在缩进块中编写函数体，函数的返回值用`return`语句返回。
比如定义一个空函数:
```python
def nop():
	pass	#pass可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个pass，让代码能运行起来。
```
## 2.返回多个值
python中函数可以返回多个值,比如:
```python
def lxj(x,y,t):
	a = x + y*t
	b = y + x*t
	return a,b
```
那么我们可以这样调用:
```python
>>>p,q = lxj(5,8,2) 
>>> print(p, q)
```
但其实这只是一种假象，Python函数返回的仍然是单一值：
```python
>>>r = lxj(4,7,3)
>>>print(r)	#与上面的代码输出一样
```
python函数返回值是一个tuple！但是，在语法上，返回一个tuple可以省略括号，而多个变量可以同时接收一个tuple，按位置赋给对应的值，所以，Python的函数返回多值其实就是返回一个tuple，但写起来更方便。
总之:
- 函数执行完毕没有return语句时，自动return None
- 函数可以同时返回多个值，但其实就是一个tuple

## 3.函数的参数
定义函数时,把函数参数的名字和位置确定下来,那么函数的接口定义就完成了.
### 3.1 位置参数
```python
    def power(x, n):	#x和n，这两个参数都是位置参数
    s = 1				#调用函数时，传入的两个值按照位置顺序依次赋给参数x和n。
    while n > 0:
        n = n - 1
        s = s * x
    return s
```
### 3.2 默认参数
```python
	def power(x, n=2):	#把第二个参数n的默认值设定为2
    s = 1				#当我们调用power(5)时，相当于调用power(5, 2)
    while n > 0:
        n = n - 1
        s = s * x
    return s
```
*注意:*
- 必选参数在前，默认参数在后，否则Python的解释器会报错
- 当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面。变化小的参数就可以作为默认参数
- 有多个默认参数时，调用的时候，既可以按顺序提供默认参数，也可以不按顺序提供部分默认参数(当不按顺序提供部分默认参数时，需要把参数名写上)
- **定义默认参数要牢记一点：默认参数必须指向不变对象！**因为不变对象一旦创建，对象内部的数据就不能修改，这样就减少了由于修改数据导致的错误。

### 3.3 可变参数
顾名思义，可变参数就是传入的参数个数是可变的，可以是1个、2个到任意个，还可以是0个。
比如计算a^2 + b^2 + c^2 + ……
```python
    def calc(numbers):	#可以把a，b，c……作为一个list或tuple传进来
    sum = 0				
    for n in numbers:	
        sum = sum + n * n
    return sum
```
```python
    >>> calc([1, 2, 3])	##但是调用的时候，需要先组装出一个list或tuple
    14
    >>> calc((1, 3, 5, 7))
    84
```

引入可变参数后:
```python
    def calc(*numbers):	#仅仅在参数前面加了一个*号
    sum = 0				#在函数内部，参数numbers接收到的是一个tuple
    for n in numbers:
        sum = sum + n * n
    return sum
```

```python
    >>> calc(1, 2)	#调用函数的方式可以简化成这样,无需先组装一个list或tuple
    5
    >>> calc()
    0
```
```python
    >>> nums = [1, 2, 3]	
    >>> calc(*nums)	#*nums表示把nums这个list的所有元素作为可变参数传进去。
    14
```
总之:

- 在函数定义中,在函数参数前了一个*号,表示在函数内部,该参数接收到的是一个list或tuple
- 在函数调用中,在函数参数前了一个*号,表示把这个list或tuple变成可变参数传进去

### 3.4 关键字参数
- 可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。
- 而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。
```python
    def person(name, age, **kw):	
        print('name:', name, 'age:', age, 'other:', kw)
```
函数person除了必选参数name和age外，还接受关键字参数kw。在调用该函数时，可以只传入必选参数：
```python
>>> person('Michael', 30)
name: Michael age: 30 other: {}
#也可以传入任意个数的关键字参数:
>>> person('Bob', 35, city='Beijing')	
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
#和可变参数类似，也可以先组装出一个dict，然后，把该dict转换为关键字参数传进去：
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}	
>>> person('Jack', 24, city=extra['city'], job=extra['job'])
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```
当然，上面复杂的调用可以用简化的写法：
```python
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, **extra)
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```
`**extra`表示把extra这个dict的所有key-value用关键字参数传入到函数的`**kw`参数，kw将获得一个dict，注意kw获得的dict是extra的一份拷贝，对kw的改动不会影响到函数外的extra。

### 3.5命名关键字参数
对于关键字参数，函数的调用者可以传入任意不受限制的关键字参数。至于到底传入了哪些，就需要在函数内部通过kw检查。
我们希望检查是否有city和job参数：
```python
def person(name, age, **kw):
    if 'city' in kw:
        # 有city参数
        pass
    if 'job' in kw:
        # 有job参数
        pass
    print('name:', name, 'age:', age, 'other:', kw)
```
但是调用者仍可以传入不受限制的关键字参数：
```python
>>> person('Jack', 24, city='Beijing', addr='Chaoyang', zipcode=123456)
```
如果要限制关键字参数的名字，就可以用命名关键字参数，例如，只接收city和job作为关键字参数。这种方式定义的函数如下：
```python
def person(name, age, *, city, job):
    print(name, age, city, job)
```
和关键字参数`**kw`不同，命名关键字参数需要一个特殊分隔符`*`，`*`后面的参数被视为命名关键字参数。相应的调用方式如下：
```python
>>> person('Jack', 24, city='Beijing', job='Engineer')
Jack 24 Beijing Engineer
```
如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符`*`了：
```python
def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```
- 命名关键字参数必须传入参数名，这和位置参数不同。如果没有传入参数名，调用将报错.
- 命名关键字参数可以有缺省值，从而简化调用：
```python
def person(name, age, *, city='Beijing', job):
    print(name, age, city, job)
```
    由于命名关键字参数city具有默认值，调用时，可不传入city参数：
    ```python
    >>> person('Jack', 24, job='Engineer')
    Jack 24 Beijing Engineer
    ```
- 使用命名关键字参数时，要特别注意，如果没有可变参数，就必须加一个`*`作为特殊分隔符。如果缺少`*`，Python解释器将无法识别位置参数和命名关键字参数：
```python
def person(name, age, city, job):
    # 缺少 *，city和job被视为位置参数
    pass
```
### 3.6 参数组合
在Python中定义函数:
- 可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用
- 参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数
比如:
```python
def f1(a, b, c=0, *args, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

def f2(a, b, c=0, *, d, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
```
    在函数调用的时候，Python解释器自动按照参数位置和参数名把对应的参数传进去:
```python
    >>> f1(1, 2)
    a = 1 b = 2 c = 0 args = () kw = {}
    >>> f1(1, 2, c=3)
    a = 1 b = 2 c = 3 args = () kw = {}
    >>> f1(1, 2, 3, 'a', 'b')
    a = 1 b = 2 c = 3 args = ('a', 'b') kw = {}
    >>> f1(1, 2, 3, 'a', 'b', x=99)
    a = 1 b = 2 c = 3 args = ('a', 'b') kw = {'x': 99}
    >>> f2(1, 2, d=99, ext=None)
    a = 1 b = 2 c = 0 d = 99 kw = {'ext': None}
```
最神奇的是通过一个tuple和dict，你也可以调用上述函数：
```python
	>>> args = (1, 2, 3, 4)
    >>> kw = {'d': 99, 'x': '#'}
    >>> f1(*args, **kw)
    a = 1 b = 2 c = 3 args = (4,) kw = {'d': 99, 'x': '#'}
    >>> args = (1, 2, 3)
    >>> kw = {'d': 88, 'x': '#'}
    >>> f2(*args, **kw)
    a = 1 b = 2 c = 3 d = 88 kw = {'x': '#'}
```
- 所以，对于任意函数，都可以通过类似`func(*args, **kw)`的形式调用它，无论它的参数是如何定义的。

**函数参数小结:**
>- 默认参数一定要用不可变对象，如果是可变对象，程序运行时会有逻辑错误！
>- 可变参数:`*args`是可变参数，args接收的是一个tuple；
>- 关键字参数:`**kw`是关键字参数，kw接收的是一个dict.
>- 可变参数既可以直接传入：`func(1, 2, 3)`，又可以先组装list或tuple，再通过`*args`传入：`func(*(1, 2, 3))`；
>- 关键字参数既可以直接传入：`func(a=1, b=2)`，又可以先组装dict，再通过`**kw`传入：`func(**{'a': 1, 'b': 2})`。
>- 使用`*args`和`**kw`是Python的习惯写法，当然也可以用其他参数名，但最好使用习惯用法。
>- 命名的关键字参数是为了限制调用者可以传入的参数名，同时可以提供默认值。
>- 定义命名的关键字参数在没有可变参数的情况下不要忘了写分隔符*，否则定义的将是位置参数。


## 4.递归函数
在函数内部，可以调用其他函数。如果一个函数在内部调用自身本身，这个函数就是递归函数。
比如:计算阶乘`n! = 1 x 2 x 3 x ... x n`
```python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)	#每次返回的都是包含函数本身的一个乘法表达式
```
- 理论上，所有的递归函数都可以写成循环的方式，但循环的逻辑不如递归清晰。
- 使用递归函数需要注意防止栈溢出。在计算机中，函数调用是通过栈（stack）这种数据结构实现的，当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。
- 解决递归调用栈溢出的方法是通过尾递归优化，事实上尾递归和循环的效果是一样的，所以，把循环看成是一种特殊的尾递归函数也是可以的。
- **尾递归**是指，**在函数返回的时候，调用自身本身，并且，return 语句不能包含表达式**。这样，编译器或者解释器就可以把尾递归做优化，使递归本身无论调用多少次，都只占用一个栈帧，不会出现栈溢出的情况。
使用尾递归求阶乘:
```python
def fact(n):
    return fact_iter(n, 1)

def fact_iter(num, product):
    if num == 1:
        return product
    return fact_iter(num - 1, num * product)	#仅仅返回函数本身,不是表达式
```
- 尾递归调用时，如果做了优化，栈不会增长，因此，无论多少次调用也不会导致栈溢出。
- 遗憾的是，大多数编程语言没有针对尾递归做优化，Python 解释器也没有做优化，所以，即使把上面的fact(n)函数改成尾递归方式，也会导致栈溢出。