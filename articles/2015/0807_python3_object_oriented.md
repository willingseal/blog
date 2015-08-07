#python3的面向对象编程

<pre>
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_score(self):
        print('%s: %s' % (self.name, self.score))
</pre>

__init__方法是一个类的初始化方法，__init__方法的第一个参数永远是self，表示创建的实例本身，因此，在__init__方法内部，就可以把各种属性绑定到self，因为self就指向创建的实例本身。

<pre>
>>> bart = Student('Bart Simpson', 59)
>>> bart.name
'Bart Simpson'
>>> bart.score
59

>>> bart.print_score()
Bart Simpson: 59

</pre>

######访问限制

以一个下划线开头的实例变量名，比如_name，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是，“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”。

如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线__，在Python中，实例的变量名如果以__开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问。


<pre>
class Student(object):

    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))
        
    def get_name(self):
        return self.__name

    def get_score(self):
        return self.__score
    
    def set_score(self, score):
        if 0 <= score <= 100:
            self.__score = score
        else:
            raise ValueError('bad score')
            
</pre>




######实例属性和类属性

<pre>
class Student(object):
    name = 'Student'
    def __init__(self, name):
        self.name = name
    
s = Student('23') # 创建实例s
print(s.name)# 打印name属性，如果实例并没有name属性，会继续查找class的name属性
print(Student.name)# 打印类的name属性
s.name = 'Michael'# 给实例绑定name属性
print(s.name) # 由于实例属性优先级比类属性高，因此，它会屏蔽掉类的name属性
print(Student.name) # 但是类属性并未消失，用Student.name仍然可以访问
del s.name # 如果删除实例的name属性
print(s.name) # 再次调用s.name，由于实例的name属性没有找到，类的name属性就显示出来了
</pre>
结果
<pre>
23
Student
Michael
Student
Student
</pre>
[liaoxuefeng-实例属性和类属性](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014319117128404c7dd0cf0e3c4d88acc8fe4d2c163625000)

######使用@property
<pre>
class Student(object):

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value

s = Student()
s.score = 60
print('s.score =', s.score)
# ValueError: score must between 0 ~ 100!
s.score = 9999
</pre>


######python的运算符多态
<pre>
def add(x,y):
    return x+y

print add(1,2) #输出3

print add("hello,","world") #输出hello,world

print add(1,"abc") #抛出异常 TypeError: unsupported operand type(s) for +: 'int' and 'str'
</pre>


<pre>
#!/usr/bin/python

class Vector:
   def __init__(self, a, b):
      self.a = a
      self.b = b

   def __str__(self):
      return 'Vector (%d, %d)' % (self.a, self.b)
   
   def __add__(self,other):
      return Vector(self.a + other.a, self.b + other.b)

v1 = Vector(2,10)
v2 = Vector(5,-2)
print v1 + v2   #结果Vector(7,8)
</pre>


######类中内置的方法
在Python中有一些内置的方法，这些方法命名都有比较特殊的地方（其方法名以2个下划线开始然后以2个下划线结束）。类中最常用的就是构造方法和析构方法。

构造方法__init__(self,....)：在生成对象时调用，可以用来进行一些初始化操作，不需要显示去调用，系统会默认去执行。构造方法支持重载，如果用户自己没有重新定义构造方法，系统就自动执行默认的构造方法。
　　
析构方法__del__(self)：在释放对象时调用，支持重载，可以在里面进行一些释放资源的操作，不需要显示调用。

　　　　　　
<table >
<tr>
<td>&nbsp;内置方法 </td>
<td>&nbsp;说明</td>
</tr>
<tr>
<td>&nbsp;__init__(self,...)</td>
<td>&nbsp;初始化对象，在创建新对象时调用</td>
</tr>
<tr>
<td>&nbsp;__del__(self)</td>
<td>&nbsp;释放对象，在对象被删除之前调用</td>
</tr>
<tr>
<td>&nbsp;__new__(cls,*args,**kwd)</td>
<td>&nbsp;实例的生成操作</td>
</tr>
<tr>
<td>&nbsp;__str__(self)</td>
<td>&nbsp;在使用print语句时被调用</td>
</tr>
<tr>
<td>&nbsp;__getitem__(self,key)</td>
<td>&nbsp;获取序列的索引key对应的&#20540;，等价于seq[key]</td>
</tr>
<tr>
<td>&nbsp;__len__(self)</td>
<td>&nbsp;在调用内联函数len()时被调用</td>
</tr>
<tr>
<td>&nbsp;__getattr__(s,name)</td>
<td>&nbsp;获取属性的&#20540;</td>
</tr>
<tr>
<td>&nbsp;__setattr__(s,name,value)</td>
<td>&nbsp;设置属性的&#20540;</td>
</tr>
<tr>
<td>&nbsp;__delattr__(s,name)</td>
<td>&nbsp;删除name属性</td>
</tr>
<tr>
<td>&nbsp;__getattribute__()</td>
<td>&nbsp;__getattribute__()功能与__getattr__()类&#20284;</td>
</tr>
<tr>
<td>&nbsp;__gt__(self,other)</td>
<td>&nbsp;判断self对象是否大于other对象</td>
</tr>
<tr>
<td>&nbsp;__lt__(slef,other)</td>
<td>&nbsp;判断self对象是否小于other对象</td>
</tr>
<tr>
<td>&nbsp;__ge__(slef,other)</td>
<td>&nbsp;判断self对象是否大于或者等于other对象</td>
</tr>
<tr>
<td>&nbsp;__le__(slef,other)</td>
<td>&nbsp;判断self对象是否小于或者等于other对象</td>
</tr>
<tr>
<td>&nbsp;__eq__(slef,other)</td>
<td>&nbsp;判断self对象是否等于other对象</td>
</tr>
<tr>
<td>&nbsp;__call__(self,*args)</td>
<td>&nbsp;把实例对象作为函数调用</td>
</tr>
</table>
[类中内置的方法](http://blog.csdn.net/zhoudaxia/article/details/23341261)