#python数据类型
空（None） ， [布尔类型（Boolean）](#Boolean)，[整型(Int)](#Int)，[浮点型(Float)](#Float)，[字符串(String)](#String)，[列表(List)](#List)，[元组(Tuple)](#Tuple)，[集合(Set)](#Set)，[字典(Dict)](#Dict)
###<a id="Boolean"></a>布尔类型（Boolean）
布尔型，在Python2中是没有布尔型的，它用数字0表示False，用1表示True。到Python3中，把True和False定义成关键字了，但它们的值还是1和0，它们可以和数字相加。

布尔值可以用and、or和not运算。

###<a id="Int"></a>整型(Int)

在Python2内部对整数的处理分为普通整数和长整数，普通整数长度为机器位长，通常都是32位，超过这个范围的整数就自动当长整数处理，而长整数的范围几乎完全没限制。

在Python 3里，只有一种整数类型int，大多数情况下，它很像Python 2里的长整型。由于已经不存在两种类型的整数，所以就没有必要使用特殊的语法去区别他们。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
i =31212233234434343434343434343434343434
i=i*i
print(i)
</pre>
结果正确，竟然不报错，牛逼！
<pre>
974203503480727755908956330986634016915608534421568003264972961942658912356
</pre>

[link:python data type](http://nxlhero.blog.51cto.com/962631/697055)

###<a id="Float"></a>浮点型(Float)

浮点数是用机器上浮点数的本机双精度(64 bit)表示的。提供大约17位的精度和范围从-308到308的指数。和C语言里面的double类型相同。Python不支持32bit的单精度浮点数。

[link:float](http://www.cnblogs.com/herbert/p/3402245.html)
<pre>
f=3.1212132232324343434342323434332324343323343423434434343434
print(f)
</pre>
结果是17位的精度
<pre>
3.1212132232324343
</pre>

对于很大或很小的浮点数，就必须用科学计数法表示，把10用e替代，1.23x109就是1.23e9。

整数和浮点数在计算机内部存储的方式是不同的，整数运算永远是精确的（除法难道也是精确的？是的！），而浮点数运算则可能会有四舍五入的误差。

###<a id="String"></a>字符串(String)

<pre>
print('I\'m \"OK\"!')
print(r'\\\t\\')    #r''表示''内部的字符串默认不转义

#Python允许用'''...'''的格式表示多行内容
print('''line1    
line2
line3''')
</pre>

字符串操作
截取字符串
<pre>
str = '0123456789'
print(str[0:3]) #截取第一位到第三位的字符
print(str[:]) #截取字符串的全部字符
print(str[6:]) #截取第七个字符到结尾
print(str[:-3]) #截取从头开始到倒数第三个字符之前
print(str[2]) #截取第三个字符
print(str[-1]) #截取倒数第一个字符
print(str[::-1]) #创造一个与原字符串顺序相反的字符串
print(str[-3:-1]) #截取倒数第三位与倒数第一位之前的字符
print(str[-3:]) #截取倒数第三位到结尾
print(str[:-5:-3]) #逆序截取，截取倒数第五个至倒数第3个并且翻转
</pre>
查找字符串
<pre>


sStr1 = 'strchr' #strchr(sStr1,sStr2) < 0 为未找到
sStr2 = 's'
nPos = sStr1.index(sStr2)
print (nPos)
</pre>

x in s 查询 x 是否在 s 中

<pre>
>>> s = "HelloWorld"  
>>> len(s)  
10  
>>> "ll" in s  
True  
</pre>

s.find( x )  在字符串 s 中找子串 x ，找到则返回最左端的索引，找不到则返回-1

<pre>
>>> s  
'HelloWorld'  
>>> s.find("l")  
2  
>>> s.find('w')  
-1  
</pre>

s.split( x ) 以 x 作为分隔符将 s 分割成一个字符串列表，如果不提供x，则程序会自动将所有空格和换行作为分隔符分割
<pre>
>>> s = "here#there"  
>>> s.split('#') # #作为分隔符  
['here', 'there'] 
</pre>

s.join( seq )  split 的逆方法，将序列 seq 用 s 连接起来，必须是字符串序列
<pre>
>>> L = ['1','33','42']  
>>> '+'.join(L)  #用+来连接  
'1+33+42'  
</pre>

强大的格式化方法 format

[Python 3语法小记（五）字符串](http://blog.csdn.net/jcjc918/article/details/9368561)
<pre>
>>> name = 'Jonh'  
>>> call = '13560300xxx'  
>>> "{0}'s telephone number is {1}".format(name,call) # (1)  
"Jonh's telephone number is 13560300xxx"  
</pre>
<pre>
>>> L = [2,3,5,7]  
>>> print("x is {0[0]}, y is {0[2]}".format(L))  
x is 2, y is 5  
</pre>
<pre>
>>> d  
{'b': 2, 'a': 1}  
>>> print("x is {0[a]}, y is {0[b]}".format(d))  
</pre>

string模块，还提供了很多方法，如  
S.find(substring, [start [,end]]) #可指范围查找子串，返回索引值，否则返回-1  
S.rfind(substring,[start [,end]]) #反向查找  
S.index(substring,[start [,end]]) #同find，只是找不到产生ValueError异常  
S.rindex(substring,[start [,end]])#同上反向查找  
S.count(substring,[start [,end]]) #返回找到子串的个数  

S.lowercase()  
S.capitalize()      #首字母大写  
S.lower()           #转小写  
S.upper()           #转大写  
S.swapcase()        #大小写互换  

string的转换  
              
float(str) #变成浮点数，float("1e-1")  结果为0.1  
int(str)        #变成整型，  int("12")  结果为12  
int(str,base)   #变成base进制整型数，int("11",2) 结果为2  
long(str)       #变成长整型，  
long(str,base)  #变成base进制长整型，  

[Python 列表(list)、字典(dict)、字符串(string)常用基本操作小结](http://blog.csdn.net/business122/article/details/7536991)

[黄聪（string替换、删除、截取、复制、连接、比较、查找、包含、大小写转换、分割等）](http://www.cnblogs.com/huangcong/archive/2011/08/29/2158268.html)

[Python 字符串操作方法大全](http://www.jb51.net/article/47956.htm)

[strip lstrip rstrip使用方法](http://blog.csdn.net/suofiya2008/article/details/5608309)

[python函数每日一讲 - cmp(x,y)](http://www.pythontab.com/html/2013/hanshu_0201/198.html)

[Python 3语法小记（五）字符串](http://blog.csdn.net/jcjc918/article/details/9368561)


#<a id="List"></a>列表(List)

List相当于Objective-C的NSMutableArray，可变数组。
[liaoxuefeng-使用list和tuple](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014316724772904521142196b74a3f8abf93d8e97c6ee6000)
<pre>
>>> classmates = ['Michael', 'Bob', 'Tracy']
>>> classmates
['Michael', 'Bob', 'Tracy']
</pre>

如果要取最后一个元素，除了计算索引位置外，还可以用-1做索引，直接获取最后一个元素：

<pre>
>>> classmates[-1]
'Tracy'
</pre>




classmates[1] = 'Sarah' #替换
 
可以不同的数据类型 
 <pre>
 L = ['Apple', 123, True]
 </pre>
 
 列表遍历  
 <pre>
 for element in sample_list:  
    print('element')
 </pre>
 
 range()函数
<pre>
>>> list(range(1,5)) #代表从1到5(不包含5)
[1, 2, 3, 4]
>>> list(range(1,5,2)) #代表从1到5，间隔2(不包含5)
[1, 3]
>>> list(range(5)) #代表从0到5(不包含5)
[0, 1, 2, 3, 4]
</pre>

list的方法 
  
L.append(var)   #追加元素  
L.insert(index,var)  
L.pop(var)      #删除指定位置的元素，为空则删除末尾元素  
L.remove(var)   #删除第一次出现的该元素  
L.count(var)    #该元素在列表中出现的个数  
L.index(var)    #该元素的位置,无则抛异常   
L.extend(list)  #追加list，即合并list到L上  
L.sort()        #排序  
L.reverse()     #倒序 

list 操作符:,+,*，关键字del  

a[1:]       #片段操作符，用于子list的提取  
[1,2]+[3,4] #为[1,2,3,4]。同extend()  
[2]*4       #为[2,2,2,2]  
del L[1]    #删除指定下标的元素  
del L[1:3]  #删除指定下标范围的元素  

list的复制  

L1 = L      #L1为L的别名，用C来说就是指针地址相同，对L1操作即对L操作。函数参数就是这样传递的  
L1 = L[:]   #L1为L的克隆，即另一个拷贝。  

[Python 列表(list)、字典(dict)、字符串(string)常用基本操作小结](http://blog.csdn.net/business122/article/details/7536991) 
[详细记录python的range()函数用法](http://www.cnblogs.com/buro79xxd/archive/2011/05/23/2054493.html) :主要讲明白了list中[::]是什么
[List 介绍](http://woodpecker.org.cn/diveintopython/native_data_types/lists.html):其实这个[:]叫做list 的分片 (slice)


#<a id="Tuple"></a>元组(Tuple)

tuple和list非常类似，但是tuple一旦初始化就不能修改，相当于Objective-C的NSArray。

<pre>
>>> classmates = ('Michael', 'Bob', 'Tracy')
</pre>

只有1个元素的tuple定义时必须加一个逗号,，来消除歧义,因为括号()既可以表示tuple，又可以表示数学公式中的小括号：

<pre>
>>> t = (1)
>>> t
1

>>> t = (1,)
>>> t
(1,)

</pre>

<pre>
tup1 = (12, 34.56);
# 以下修改元组元素操作是非法的。
# tup1[0] = 100;
</pre>

max(tuple)：返回元组中元素最大值。

min(tuple)：返回元组中元素最小值。

tuple(list1)：将列表转换为元组。

list(tuple1)：将元组转换为列表。

可以使用 in / not in 来判断是否包含某个元素。

filter() 
<pre>
def divfilter(i):
    return i%2==0

a=list(range(1,10))
print(a)
b=list(filter(divfilter, a))
print(b)

</pre>

可以简写为
<pre>
a=list(range(1,10))
print(a)
b=list(filter(lambda i: i%2==0, a))
print(b)
</pre>

可以用来过滤空值
<pre>
a=['a', '', [], [1,2]] 
b=list(filter(None, a))
print(b)
</pre>
结果：['a', [1, 2]]

map()

<pre>
>>> list(map(lambda i:i*2, range(10)))
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
</pre>
reduce()
我们还可以使用 reduce() 对元素进行统计。
<pre>

from functools import reduce
import operator
sum1=reduce(operator.add, range(10))
print(sum1)
sub1=reduce(operator.sub, [100, 5, 7])
print(sub1)

</pre>

结果：
<pre>
45
88
</pre>

zip() 方法可以对两个或多个列表/元组进行交叉合并。
<pre>
>>> list(zip(range(2,10), ('a', 'b', 'c', 'd', 'e')) )
[(2, 'a'), (3, 'b'), (4, 'c'), (5, 'd'), (6, 'e')]
</pre>
[liaoxuefeng-list&tuple](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014316724772904521142196b74a3f8abf93d8e97c6ee6000)

[Python 元组(Tuple)操作详解](http://www.jb51.net/article/47986.htm)



#<a id="Dict"></a>字典(Dict)

每一个元素是pair，包含key、value两部分。key是Integer或string类型，value 是任意类型。  
键是唯一的，字典只认最后一个赋的键值。

<pre>
>>> d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
>>> d['Michael']
95
</pre>

<pre>
>>> 'Thomas' in d
False
</pre>

通过dict提供的get方法，如果key不存在，可以返回None，或者自己指定的value：

<pre>
>>> d.get('Thomas')
>>> d.get('Thomas', -1)
-1
</pre>



dictionary的方法  
D.get(key, 0)       #同dict[key]，多了个没有则返回缺省值，0。[]没有则抛异常  
D.has_key(key)      #有该键返回TRUE，否则FALSE  
D.keys()            #返回字典键的列表  
D.values()          #以列表的形式返回字典中的值，返回值的列表中可包含重复元素  
D.items()           #将所有的字典项以列表方式返回，这些列表中的每一项都来自于(键,值),但是项在返回时并没有特殊的顺序           
  
D.update(dict2)     #增加合并字典  
D.popitem()         #得到一个pair，并从字典中删除它。已空则抛异常  
D.clear()           #清空字典，同del dict  
D.copy()            #拷贝字典 

pop():删除一个key

[liaoxuefeng-dict&set](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143167793538255adf33371774853a0ef943280573f4d000)

[ython 列表(list)、字典(dict)、字符串(string)常用基本操作小结](http://blog.csdn.net/business122/article/details/7536991)


#<a id="Set"></a>集合(Set)

set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。

<pre>
>>> s = set([1, 2, 3])
>>> s
{1, 2, 3}

>>> s = set([1, 1, 2, 2, 3, 3])
>>> s
{1, 2, 3}

>>> s.add(4)
>>> s
{1, 2, 3, 4}


>>> s.remove(4)
>>> s
{1, 2, 3}
</pre>

<pre>
>>> s1 = set([1, 2, 3])
>>> s2 = set([2, 3, 4])
>>> s1 & s2
{2, 3}
>>> s1 | s2
{1, 2, 3, 4}
</pre>



[Python 3 学习笔记](https://www.zybuluo.com/king/note/76005) :介绍了一些内置数据类型，比较全面





