#python3常用模块

###内置
####os模块
用来操作文件

[python3 中os模块的一些常用函数](http://m.oschina.net/blog/206207)

[python-os-demo](https://github.com/coderyi/python-demos/blob/master/0006_os/0006.py)
####sys
用来接受命令行参数

[python天天进步（1）--sys.argv[]用法](http://blog.csdn.net/vivilorne/article/details/3863545)

[python-sys-demo](https://github.com/coderyi/python-demos/tree/master/0011_passgen)
####re
Python 通过 re 模块为正则表达式引擎提供一个接口，同时允许你将正则表达式编译成模式对象，并用它们来进行匹配。 

re 模块是使用 C 语言编写，所以效率比你用普通的字符串方法要高得多；将正则表达式进行编译（compile）也是为了进一步提高效率；我们经常提到的“模式”，指的就是正则表达式被编译成的模式对象。 

当你将正则表达式编译之后，你就得到一个模式对象。那你拿他可以用来做什么呢？模式对象拥有很多方法和属性，我们下边列举最重要的几个来讲： 

方法	功能 

match()	判断一个正则表达式是否从开始处匹配一个字符串 

search()	遍历字符串，找到正则表达式匹配的第一个位置 

findall()	遍历字符串，找到正则表达式匹配的所有位置，并以列表的形式返回 

finditer()	遍历字符串，找到正则表达式匹配的所有位置，并以迭代器的形式返回 


[Python3 如何优雅地使用正则表达式（详解二）](http://www.douban.com/group/topic/71307340/)

[Python正则表达式指南](http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html)

[python-re-demo](https://github.com/coderyi/python-demos/tree/master/0007_re)
####json模块
Python处理JSON

http://www.cnblogs.com/coser/archive/2011/12/14/2287739.html

####xlwt模块
Python 使用 Xlrd/xlwt 操作 Excel

http://liluo.org/blog/2011/01/python-using-xlrd-xlwt-operate-excel/

####turtle模块

python海龟绘图实例教程

http://blog.csdn.net/yelyyely/article/details/40746659


####urllib包
网络请求相关

###第三方



####PIL
图像处理
[python-pillow官网地址](http://python-pillow.github.io/)
[python-PIL-demo](https://github.com/coderyi/python-demos/tree/master/0008_PIL_image)

####requests
第三方网络请求模块
