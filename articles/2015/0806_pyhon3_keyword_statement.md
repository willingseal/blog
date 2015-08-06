#python3的关键字和语句


###赋值语句


运算|解释
---|---
name = 'mathboy'|基本形式
name, age = 'mathboy', 23|元组赋值
[name, age] = ['mathboy', 23]|列表赋值
a,b,c,d = 'girl'|序列赋值
boy = girl = 'mo'|多目标赋值
age += 23|增强赋值（相当于age = age + 23）

元组赋值可以利用来交换两个变量的值
<pre>
a = 1  
b = 2  
a,b = b,a  
print (a,b) #输出2 1  
</pre>

序列赋值的右侧可以是任意可迭代的对象，但右边元素的数目要跟左侧变量的数目相同。
<pre>
name = 'math'  
  
a,b,c,d = name  
print (a,b,c,d )
  
a,b,c = name[0], name[1], name[2:]  
print (a,b,c )
  
a,b,c = list(name[:2]) + [name[2:]]  
print(a,b,c  )
  
a,b = name[:2]  
print( a,b  )
  
(a,b),c = name[:2], name[2:]  
print (a,b,c )
</pre>
结果
<pre>
m a t h
m a th
m a th
m a
m a th
</pre>

多目标赋值时，要注意共享引用的情况。例如：
<pre>
a = b = 0  
b += 1  
print (a,b)
  
c = d = []  
d.append(3.14)  
print (c,d)
</pre>
输出结果
<pre>
0 1  
[3.14] [3.14]  
</pre>

强赋值通常比完整形式的赋值执行得更快，同时会自动选择优化技术。例如对于列表，+= 赋值会自动调用较快的extend()方法，而不是使用较慢的 + 合并运算。
<pre>
L = [1,2]  
L = L + [3,4]       # slower  
L.extend([5,6])     # faster  
L += [7,8]          # mapped to L.extend([7,8])  
</pre>


在赋值中使用if语句，如果b等于c，返回a，否则返回c
<pre>
a=1
b=2
c=21
d=a if b==c else c  #d返回21
</pre>

and符号表示如果a为零则返回a，否则返回b，也就是and总是返回第一个为零的数，当然，如果都不为零，则返回最后一个数
<pre>
a=1
c=21
d=a and c
print(d) #21
</pre>

or语句总是返回第一个不为零的数，下面返回的是a
<pre>
a=1
c=21
d=a or c
print(d) #1
</pre>


[Python学习笔记（八）：Python语句简介、赋值、表达式和打印](http://blog.csdn.net/mathboylinlin/article/details/9380567)

[python教程：[79]赋值运算中的条件语句](http://jingyan.baidu.com/article/b87fe19ebbb7f85218356891.html)


######and...or 语法


Python的and/or操作与其他语言不同的是它的返回值是参与判断的两个值之一，所以我们可以通过这个特性来实现Python下的 a ? b : c !

有C语言基础的知道 “ a ? b : c ! ” 语法是判断 a，如果正确则执行b，否则执行 c！

而Python下我们可以这么用：“ a and b or c ”（此方法中必须保证b必须是True值），python自左向右执行此句，先判断a and b ：如果a是True值，a and b语句仍需要执行b，而此时b是True值！所以a and b的值是b，而此时a and b or c就变成了b or c，因b是True值，所以b or c的结果也是b；如果a是False值，a and b语句的结果就是a，此时 a and b or c就转化为a or c,因为此时a是 False值，所以不管c是True 还是Flase，a or c的结果就是c！！！捋通逻辑的话，a and b or c 是不是就是Python下的a ? b : c ! 用法？

######with 关键字   

异常处理
<pre>
file = open("/tmp/foo.txt")  
try:  
    data = file.read()  
finally:  
    file.close()  
</pre>


虽然这段代码运行良好，但是太冗余了。这时候就是with一展身手的时候了。除了有更优雅的语法，with还可以很好的处理上下文环境产生的异常。下面是with版本的代码：
<pre>
with open("/tmp/foo.txt") as file:  
    data = file.read()  
</pre>


######yield 关键字   
<pre>
def fab(max):   
   n, a, b = 0, 0, 1   
   while n < max:   
       yield b   
       a, b = b, a + b   
       n = n + 1   
</pre>

<pre>
for n in fab(5):   
        print n   

结果
1   
1   
2   
3   
5

</pre>
简单地讲，yield 的作用就是把一个函数变成一个 generator（生成器），带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator，调用 fab(5) 不会执行 fab 函数，而是返回一个 iterable（可迭代的）对象！

<pre>
>>> f = fab(5)   
>>> f.next()   
 1
</pre>

######异常语句
<pre>
try:
    x = int(input('please input an integer:'))
    if 30/x > 5:
        print('Hello World!')
except ValueError:
    print('That was no valid number. Try again...')
except ZeroDivisionError:
    print('The divisor can not be zero, Try again...')
except:
    print('Handling other exceptions...')
</pre>

<pre>
try:
　 print "Testing begins!"
    raise MyError #自己抛出一个异常，后面代码不执行.
except MyError:
    print 'This a defined error!'
except:
　　print "Other error!"
</pre>
[python 学习笔记 7 -- Python关键字总结](http://blog.csdn.net/longerzone/article/details/17607011)

[python开发_python关键字](http://www.cnblogs.com/hongten/p/hongten_python_keywords.html)

###python3命名规范
一，包名、模块名、局部变量名、函数名

全小写+下划线式驼峰

example：this_is_var

二，全局变量

全大写+下划线式驼峰

example：GLOBAL_VAR

三，类名

首字母大写式驼峰

example：ClassName()

四，关于下划线

以单下划线开头，是弱内部使用标识，from M import * 时，将不会导入该对象（python 一切皆对象）。

以双下划线开头的变量名，主要用于类内部标识类私有，不能直接访问。模块中使用见上一条。
双下划线开头且双下划线截尾的命名方法尽量不要用，这是标识


[Python基础 - 命名规范](http://www.pythontab.com/html/2013/pythonjichu_0613/441.html)