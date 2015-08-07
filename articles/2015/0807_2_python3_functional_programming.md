#python的函数式编程

函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

Python对函数式编程提供部分支持。由于Python允许使用变量，因此，Python不是纯函数式编程语言。

####高阶函数
python中变量可以指向函数
<pre>
>>> f = abs
>>> f(-10)
10
</pre>
既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。
<pre>
def add(x, y, f):
    return f(x) + f(y)
    
  

>>> add(-5, 6, abs)
11
</pre>


######迭代器(Iterator)、iterable、生成器(Generator)
python 标准库中提供了 itertools, functools, operator 三个库支持函数式编程，对高阶函数的支持，python 提供 decorator 语法糖。 迭代器 (iterator)和生成器(generator)概念是 python 函数式编程的基础，利用迭代器和生成器可以实现函数式编程中经常用到的 map(), filter(), reduce() 等过程以及 itertools, functools 中提供的绝大部分功能。

迭代器(iterator)必须至少要定义 __iter__() 和 __next__() 两个方法，通过 iter() 和 next() 函数调用。 iter() 生成一个迭代器， next() 每调用一次都会返回下一个值

生成器(generator)是用来生成迭代器的函数。与普通函数相同，只是返回值时用 yield 而不是 return。

yield构造的生成器
<pre>
def squares(start, stop):
    for i in xrange(start, stop):
        yield i*i
</pre>

等同于生成器表达式：
<pre>
（i*i for i in xrange(start, stop))
</pre>
列表推倒式是：
<pre>
[i*i for i in xrange(start, stop)]

</pre>
如果是构建一个自定义的迭代器：
<pre>
class Squares(object):
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    def __iter__(self):
        return self
    def next(self):
        if self.start >= self.stop:
            raise StopIteration
        current = self.start * self.start
        self.start += 1
        return current
</pre>
我们已经知道，可以直接作用于for循环的数据类型有以下几种：

一类是集合数据类型，如list、tuple、dict、set、str等；

一类是generator，包括生成器和带yield的generator function。

这些可以直接作用于for循环的对象统称为可迭代对象：Iterable。

可以使用isinstance()判断一个对象是否是Iterable对象
<pre>
>>> isinstance('abc', Iterable)
True
>>> isinstance((x for x in range(10)), Iterable)
True
</pre>
而生成器不但可以作用于for循环，还可以被next()函数不断调用并返回下一个值，直到最后抛出StopIteration错误表示无法继续返回下一个值了。

可以被next()函数调用并不断返回下一个值的对象称为迭代器：Iterator。

可以使用isinstance()判断一个对象是否是Iterator对象
<pre>
>>> from collections import Iterator
>>> isinstance((x for x in range(10)), Iterator)
True
>>> isinstance([], Iterator)
False
</pre>
生成器都是Iterator对象，但list、dict、str虽然是Iterable，却不是Iterator。

把list、dict、str等Iterable变成Iterator可以使用iter()函数：
<pre>
>>> isinstance(iter([]), Iterator)
True
>>> isinstance(iter('abc'), Iterator)
True
</pre>


最后可以看一下他们的源码 
内置模块builtins.py里面
<pre>
class __generator(object):
    '''A mock class representing the generator function type.'''
    def __init__(self):
        self.gi_code = None
        self.gi_frame = None
        self.gi_running = 0

    def __iter__(self):
        '''Defined to support iteration over container.'''
        pass

    def __next__(self):
        '''Return the next item from the container.'''
        pass

    def close(self):
        '''Raises new GeneratorExit exception inside the generator to terminate the iteration.'''
        pass

    def send(self, value):
        '''Resumes the generator and "sends" a value that becomes the result of the current yield-expression.'''
        pass

    def throw(self, type, value=None, traceback=None):
        '''Used to raise an exception inside the generator.'''
        pass
</pre>

from collections import Iterator
<pre>
class Iterator(Iterable):

    __slots__ = ()

    @abstractmethod
    def __next__(self):
        'Return the next item from the iterator. When exhausted, raise StopIteration'
        raise StopIteration

    def __iter__(self):
        return self

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            if (any("__next__" in B.__dict__ for B in C.__mro__) and
                any("__iter__" in B.__dict__ for B in C.__mro__)):
                return True
        return NotImplemented
</pre>




[迭代器(Iterator)与生成器(Generator)的区别](https://github.com/lzjun567/note/blob/master/note/python/iterator_generator.md)讲的很好
[Python 函数式编程](http://dengshuan.me/techs/python-functional.html)

[liaoxuefeng-Iterator](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143178254193589df9c612d2449618ea460e7a672a366000)


######map/reduce

map()函数接收两个参数，一个是函数，一个是Iterable，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回。

<pre>
>>> def f(x):
...     return x * x
...
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[1, 4, 9, 16, 25, 36, 49, 64, 81]
</pre>


reduce把一个函数作用在一个序列[x1, x2, x3, ...]上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算

<pre>
>>> from functools import reduce
>>> def fn(x, y):
...     return x * 10 + y
...
>>> reduce(fn, [1, 3, 5, 7, 9])
13579
</pre>

######filter
和map()类似，filter()也接收一个函数和一个序列。和map()不同的时，filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。

<pre>
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]
</pre>


######sorted
<pre>
>>> sorted([36, 5, -12, 9, -21])
[-21, -12, 5, 9, 36]
</pre>

sorted()函数是一个高阶函数，它还可以接收一个key函数来实现自定义的排序，例如按绝对值大小排序：

<pre>
>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, -21, 36]
</pre>


####装饰器decorator
使用语法糖@来装饰函数，相当于“myfunc = deco(myfunc)”  
但发现新函数只在第一次被调用，且原函数多调用了一次
<pre>
 
   
def deco(func):  
    print("before myfunc() called.")  
    func()  
    print("  after myfunc() called.")  
    return func  
   
@deco  
def myfunc():  
    print(" myfunc() called.")  
   
myfunc()  
</pre>

其实任何时候你定义装饰器的时候，都应该使用 functools 库中的 @wraps 装饰器来注解底层包装函数。
<pre>
import time
from functools import wraps
def timethis(func):
    '''
    Decorator that reports the execution time.
    '''
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end-start)
        return result
    return wrapper

@timethis
def countdown(n:int):

     while n > 0:
         n -= 1

countdown(100000)  #结果输出：countdown 0.016767024993896484
</pre>
<p>
假设装饰器是通过 @wraps 来实现的，那么你可以通过访问__wrapped__属性来访问原始函数：
</p>
<pre>
>>> @somedecorator
>>> def add(x, y):
...     return x + y
...
>>> orig_add = add.__wrapped__
>>> orig_add(3, 4)
7
>>>
</pre>


[Python装饰器学习（九步入门）](http://itbokeyun.com/blog/2015/07/12/python%E8%A3%85%E9%A5%B0%E5%99%A8%E5%AD%A6%E4%B9%A0%EF%BC%88%E4%B9%9D%E6%AD%A5%E5%85%A5%E9%97%A8%EF%BC%89.html)：详细

[创建装饰器时保留函数元信息](http://python3-cookbook.readthedocs.org/zh_CN/latest/c09/p02_preserve_function_metadata_when_write_decorators.html):标准

