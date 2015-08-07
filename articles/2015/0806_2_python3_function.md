#python3的函数

###print()函数


<pre>
fid = open("log.txt", "a")
print("i love you", file=fid) #把“i love you”写进log.txt文件
</pre>

<pre>
print("Foo", "Bar", sep="%",end='-iloveu')
#打印 Foo%Bar-iloveu
</pre>

总地来说，新的语法为：
<pre>
print([object, ...][, sep=' '][, end='endline_character_here'][, file=redirect_to_here])

</pre>
其中，方括号（[]）内的代码是可选的。默认地，若只调用 print() 自身，结果会追加一个换行符（ \n）。

参数格式化输出

<pre>
#%x --- hex 十六进制
#%d --- dec 十进制
#%o --- oct 八进制
#%s --- 字符串
#%f --- float 浮点数
</pre>

<pre>
str1 = "The value is:"
number1 = 11.12232323
print("%s %10.3f" %(str1,number1)) #输出"The value is: 11.122"
</pre>
<pre>
print("%.3s" %("abcde")) #输出abc
print("%.*s" %(4,"abcde")) #输出abcd
print("%10.3s" %("abcde")) #输出        abc(总长度为10，字符长度不够前面填空格)
</pre>

<pre>
print('abc\\a')print('abc\\a') #直接打印字符串abc\a
print(r'abc\\a') #直接打印字符串abc\\a    ;加上r表示忽略\转义字符
</pre>
<pre>


print('''sd
sd
sd

sd''')

</pre>
'''表示输出多行文本，输出结果
<pre>
sd
sd
sd

sd
</pre>

[python 3 初探，第 1 部分: Python 3 的新特性](https://www.ibm.com/developerworks/cn/linux/l-python3-1/)


###常用内置函数

<pre>
>>> abs(-20)
20

>>> max(2, 3, 1, -5)
3

>>> str(1.23)
'1.23'

</pre>



help(obj) 在线帮助, obj可是任何类型

type(obj) 查看一个obj的类型


str(obj) 得到obj的字符串描述

list(seq) 把一个sequence转换成一个list
   
判断一个变量是否是某个类型可以用isinstance()判断：
<pre>
>>> isinstance(a, list)
True
</pre>
判断一个类是否有某个属性
<pre>
print('hasattr(obj, \'x\') =', hasattr(obj, 'x')) # 有属性'x'吗？
</pre>
仅仅把属性和方法列出来是不够的，配合getattr()、setattr()以及hasattr()，我们可以直接操作一个对象的状态

[一个类的常用内置方法](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431866385235335917b66049448ab14a499afd5b24db000)

[liaoxuefeng-function](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014316784721058975e02b46cc45cb836bb0827607738d000)

[python常用函数年初大总结](http://my.oschina.net/fsmwhx/blog/112338)

[python学习笔记17-常用函数总结整理](http://blog.sciencenet.cn/blog-252888-787957.html)


###定义函数
<pre>
def my_abs(x):
    if x >= 0:
        return x
    else:
        return -x
</pre>

python中的函数是可以返回多个值

<pre>

import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny
</pre>

<pre>
>>> x, y = move(100, 100, 60, math.pi / 6)
>>> print(x, y)
151.96152422706632 70.0
</pre>
但其实这只是一种假象，Python函数返回的仍然是单一值,是一个tuple：
<pre>
>>> r = move(100, 100, 60, math.pi / 6)
>>> print(r)
(151.96152422706632, 70.0)
</pre>

默认参数的用法
<pre>
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
</pre>
<pre>
>>> power(5)
25
>>> power(5, 2)
25
</pre>

可变参数

由于参数个数不确定，我们首先想到可以把a，b，c……作为一个list或tuple传进来，这样，函数可以定义如下：
<pre>
def calc(numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
</pre>
<pre>
>>> calc([1, 2, 3])
14
>>> calc((1, 3, 5, 7))
84
</pre>
上面这种方法也是实现可变参数的一种牛逼方法啊！

其实可变参数就是在参数前面加了一个*号。在函数内部，参数numbers接收到的是一个tuple

<pre>
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
</pre>
<pre>
>>> calc(1, 2)
5
>>> calc()
0
</pre>

关键字参数
<pre>
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
</pre>
<pre>
>>> person('Michael', 30)
name: Michael age: 30 other: {}
</pre>
<pre>
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, city=extra['city'], job=extra['job'])
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
</pre>
当然，上面复杂的调用可以用简化的写法：
<pre>
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, **extra)
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
</pre>

命名关键字参数
如果要限制关键字参数的名字，就可以用命名关键字参数，例如，只接收city和job作为关键字参数。这种方式定义的函数如下：
<pre>
def person(name, age, *, city, job):
    print(name, age, city, job)
</pre>
和关键字参数**kw不同，命名关键字参数需要一个特殊分隔符*，*后面的参数被视为命名关键字参数。

参数组合

在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用，除了可变参数无法和命名关键字参数混合。但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数/命名关键字参数和关键字参数。
<pre>
def f1(a, b, c=0, *args, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

def f2(a, b, c=0, *, d, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
</pre>


######匿名函数

当一些函数很简单，仅仅只是计算一个表达式的值的时候，就可以使用lambda表达式来代替了。比如：
<pre>
>>> add = lambda x, y: x + y
>>> add(2,3)
5
>>> add('hello', 'world')
'helloworld'
>>>
</pre>
这里使用的lambda表达式跟下面的效果是一样的：
<pre>
>>> def add(x, y):
...     return x + y
...
>>> add(2,3)
5
>>>
</pre>

lambda表达式典型的使用场景是排序或数据reduce等：
<pre>
>>> names = ['David Beazley', 'Brian Jones',
...         'Raymond Hettinger', 'Ned Batchelder']
>>> sorted(names, key=lambda name: name.split()[-1].lower())
['Ned Batchelder', 'David Beazley', 'Raymond Hettinger', 'Brian Jones']
>>>
</pre>

匿名函数捕获变量值
<pre>
>>> x = 10
>>> a = lambda y: x + y
>>> x = 20
>>> b = lambda y: x + y
>>>
</pre>
<pre>
>>> a(10)
30
>>> b(10)
30
>>>
</pre>
详细用法看下面的链接，很高级的用法

[匿名函数捕获变量值](http://python3-cookbook.readthedocs.org/zh_CN/latest/c07/p07_capturing_variables_in_anonymous_functions.html)
######给函数参数增加元信息

使用函数参数注解是一个很好的办法，它能提示程序员应该怎样正确使用这个函数。 例如，下面有一个被注解了的函数：
<pre>
def add(x:int, y:int) -> int:
    return x + y
</pre>

<pre>
>>> help(add)
Help on function add in module __main__:
add(x: int, y: int) -> int
>>>
</pre>

[《Python Cookbook》给函数参数增加元信息](http://python3-cookbook.readthedocs.org/zh_CN/latest/c07/p03_attach_informatinal_matadata_to_function_arguments.html)
[liaoxuefeng-function](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431752945034eb82ac80a3e64b9bb4929b16eeed1eb9000)