---
title: "语法"
linkTitle: "语法"
weight: 1
description: >
  python 语法
---

{{% pageinfo %}}
语法是学习一种语言的第一步，按照惯例，还是先总结备忘录。
{{% /pageinfo %}}


## 简介

- 解释型，没有编译
- 交互式，在一个提示符 >>> 之后可以执行代码
- 面向对象，可以定义类和对象，并使用继承，多态等特性。


## 基础语法
默认情况下，python3的源码都是以 **UTF-8** 编码，所有字符串都是 unicode 字符串。

单行注释以 # 开头，多行注释可以用多个 # 号，还有 ''' 和 """。

行间缩进：使用缩进代表代码块，不需要使用大括号{}。缩进的空格是可变的，但是同一个代码块不许包含相同的缩进空格数。

```py
if True:
  print(1)
else:
  print(2)
```

多行语句：太长的行最后可以使用反斜杠 **\** 来实现多行语句。

数字（Number）类型：
- int
- bool
- float
- complex

字符串String：
- 单引号和双引号使用完全相同
- 使用三引号（''' 或者 """）可以指定一个多行字符串
- 转义符 \
- 用 + 运算符连接在一起，用 * 运算符重复
- 索引，从左往右以 0 开始，从右向左以 -1 开始
- 字符串不可变
- 切边 **str[start:end]** 或者加上步长 **str[start:end:step]**
- 反斜杠可以用来转义，使用 r 可以让反斜杠不发生转义。 如 r"this is a line with \n" 则 \n 会显示，并不是换行


等待用户输入： **\n\n** 在结果输出前会输出两个新的空行，一旦用户按下 enter 键后，程序将退出。


导入：
- import xxx
- from xxx import ...,...

```py
import sys
from sys import argv,path  #  导入特定的成员
```

## 数据类型
Python3 中常见的数据类型有：
- Number（数字）
- String（字符串）
- bool（布尔类型）
- List（列表）
- Tuple（元组）
- Set（集合）
- Dictionary（字典）


### List
写在**方括号 []** 之间，用逗号分隔开的元素列表。

列表中的元素类型可以不相同，和字符串一样可以被索引和截断，列表被截断之后返回一个包含所需元素的新列表。
列表截取的语法格式如下：
> 变量[头下标：尾下标]

索引值是以0开始，-1为末尾的位置。
加号 + 是列表连接运算符，星号 * 是重复操作。

> list = [1,2,3,4]

```py
list.append(1) # 末尾添加
del list[2] # 删除第三个元素
len(list) # 长度
list1 + list2 # 组合
3 in [1,2,3] # 元素是否在列表中
for x in [1,2,3]: print(x,end="") # 迭代
list(seq) # 将元组转化哪位列表
min(list) # 列表元素最小值
max(list) # 列表元素最大值
list.reverse() #翻转
list.clear() #清空
list.copy()  #复制列表
list.remove(obj) #移除列表中某个值的第一个匹配项
```


### Tuple 元组
元组（tuple）与列表类似，不同之处在于元组的个数不能修改，值也不能修改。元组写在小括号 **()** 里，元素之间用逗号隔开。
元组中的元素类型也可以不相同。

> tuple = ( 'abcd', 786 , 2.23, 'runoob', 70.2  )


虽然tuple的元素不可改变，但可以包含可变对象，如list。
构造包含0个或者1个 元素的元组比较特殊，有一些额外的语法规则：

```py
tup1 = ()  #空元组
tup2 = (20,) #一个元素，需要在元素后面加个逗号
```

```py
tup = (1,2,3,4,5)
tup[0] #访问
tup3 = tup1 + tup2 #创建一个新元组
del tup3 #删除，删除后再使用输出变量就会显示异常信息
3 in（1,2,3） #元素是否存在
for x in (1,2,3): print(x,end="")# 迭代

len(tuple) # 计算个数
max(tuple) #返回元组中最大值
min(tuple) #返回元组中最小值
tuple(iterable) #将可迭代系列转换为元组 

```

### Set 集合
无序，可变的数据类型，用于存储唯一的元素。
集合中的元素不会重复，并且可以进行交集、并集、差集等常见的集合操作。
集合使用**大括号{}** 表示，元素之间使用逗号分隔。也可以使用 set()函数创建集合。

创建一个空集合必须使用 set() 而不是 {}，因为 {}是用来创建一个空字典。
```py
parame = {value01,value02,...}
#或者
set(value)
```

基本操作：
```py
s = {'apple','pear'}
'pear' in s # 判断是否在集合中存在
s.remove('pear') # 移除
s.remove('abc')  # 不存在发生报错

s.discard('abc') # 移除集合中的元素，如果不存在不会发生错误

s.pop() #随机删除结合中的元素
len(s) #集合s中的元素个数
s.clear() #清空集合


symmetric_difference() #返回两个集合中不重复的元素集合
symmetric_difference_update() #移除当前集合中在另外一个指定集合相同的元素，并将另一个集合中不同的元素插入到当前集合中
union() #返回两个集合的并集
update() #给集合添加元素

```




### Dictionary 字典
字典是一种映射类型，用 **{}** 标识，是一个无序的键值对集合。
key必须使用不可变类型，key必须是唯一的。

创建字典：
> dict1 = {} 或者使用内建函数 dict2 = dict()

如果用字典里没有的键来访问数据，会输出错误。


```py 

dict = {}
dict['one'] = "1 - 菜鸟教程"  #添加
dict[2]     = "2 - 菜鸟工具"  # 添加，key可以是不同类型

del dict[2]   # 删除键
dict.clear()  # 清空字典
del dict      # 删除字典

len(dict1) # 字典元素的个数
str(dict1) # 输出字典


dict.copy() # 浅复制
dict.fromkeys() #创建一个新字典，以序号seq为key，val为字典所有键对应的初始值

dict.get(key,default=None) #返回指定的值，键不在字典中返回 default 设置的默认值

key in dict #是否存在
dict.items() #以列表返回一个视图对象
dict.keys() #返回一个视图对象
dict.values() #返回一个视图对象
dict.setdefault(key,default=None) # 和get()类似，如果键不在字典中，将会添加键并将值设置为 default


```



### 数据类型转换

- int(x) 将x转换为一个整数
- str(x) 将对象x转换为字符串
- repr(x)
- eval(str)
- tuple(s) 将序列s转为一个元组
- list(s) 将序列s转换为一个列表
- set(s) 将序列s转为一个可变集合
- dict(d) 创建一个字典，d必须是一个(key,value)元组序列
- ord(x) 将字符转为它的整数值


## 流程控制

### if 语句

```py
if condition1:
  statement_block_1
elif condition2:
  statement_block_2
else:
  statement_block_3

```

### 循环语句
使用 while 和 for 都可以，break是跳出当前循环体，不会执行else子句。continue 是跳出本次循环。

如果需要遍历数字，可以使用内置 range() 函数，它会生成数列， **for i in range(5)** 。

```py
while 判断条件:
  执行语句
else:
  执行语句2


for <variable> in <sequence>:
  <statements>
else:
  <statements>
```

## Python 推导式
推导式是一个独特的数据处理方式，可以从一个数据序列构建另一个新的数据序列的结构体。

python支持各种数据结构的推导式。

### 1. List 列表

> [表达式 for 变量 in 列表]  或者  [表达式 for 变量 in 列表 if 条件]

```py
# 过滤长度小于等于3的字符串列表，并且转换成大写的字母
names = ['Bob','Tom','alice']
new_names = [name.upper() for name in names if len(name) > 3]
print(new_names)

# 30以内可以被3整除的整数
a = [i for i in range(30) if i % 3 == 0]
print(a)
```

### 2. Dict 字典
> {key_expr: value_expr for value in collection}   或者   {key_expr: value_expr for value in collection if condition}

```py
demo = ['a','b','c']
my_dict = {key:len(key) for key in demo}

my_dict2 = {x:x**2 for x in (2,4,6)}
```

### 3. Set 集合
>  { expression for item in Sequence } 或者 { expression for item in Sequence if conditional }

```py
setnew = { i**2 for i in (1,2,3)}
print(setnew)
# 1,4,9


# 判断不是abc的字母输出
a = { x for x in 'abracadabra' if x not in 'abc'}
print(a)
# {'d','r'}
type(a)
# <class 'set'>
```

### 元组
元组推导式可以利用 range 区间、元组、列表、字典和集合等数据结构，快速生成一个满足指定需求的元组。
> ( expression for item in Sequence ) 或者 ( expression for item in Sequence if conditional )

元组和list用法完全相同，元组推导式返回的结果是一个生成器对象。

```py
a= (x for x in range(1,10))
print(a)
# <generator object <genexpr> at xxxx> 返回生成器对象

tuple(a) #使用 tuple()函数，可以直接将生成器对象转换成元组
# (1,2,3,4,5,6,7,8,9)
```


## 迭代器和生成器

### 迭代器
- iter()
- next()

```py
list = [1,2,3,4]
it = iter(list)
print(next(it)) # 1
print(next(it)) # 2

for x in it:
  print(x,end=" ")
```

把一个类作为迭代器使用需要在类中实现两个方法 **__iter__()** 和 **__next__()**。

python的构造函数 **__init__()** ，会在对象初始化的时候执行

```py
class MyNumbers:
  def __iter__(self):
      self.a = 1
      return self

  def __next__(self):
    x = self.a
    self.a += 1
    return x


myclass = MyNumbers()
myIter = iter(myclass)

print(next(myIter))
```


**StopIteration** 异常用于标识迭代的完成，防止出现无限循环的情况，在 __next__() 方法中设置在完成指定循环后触发异常来结束迭代。

```py

def __next__(self):
  if self.a <= 20:
    x = self.a
    self.a += 1
    return x
  else:
    raise StopIteration

```

### 生成器
使用了 **yield** 的函数称为生成器。
yield 是一个关键字，用于定义生成器函数，在迭代的过程中逐步产生值，而不是一次性返回所有的结果。



## lambda
使用 lambda 创建匿名函数

通常是用于编写简单的、单行的函数，通常在需要函数作为参数传递的情况下使用，例如：

- map()
- filter()
- reduce()

```py

# 计算乘积，输出120
numbers = [1,2,3,4,5]
product = reduce(lambda x,y :x * y ,numbers )
print(product)


numbers = [1, 2, 3, 4, 5, 6, 7, 8]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # 输出：[2, 4, 6, 8]

```

## 装饰器 Decorators
装饰器是 Python 的一种高级功能，允许动态地修改函数或者类的行为。

装饰器是一种函数，接受一个函数作为参数，并返回一个新的函数或者修改原来的函数。
装饰器的语法使用 **@decorator_name** 来应用在函数或者方法上。


还提供一些内置的装饰器，用于定义静态方法和类方法以及属性。如：
- @staticmethod
- @classmethod
- @property 

```py
@decorator_function
def function_name():
  pass

# 等价于
def function_name():
  pass
function_name = decorator_function(function_name)


def decorator_function(func):
  def wrapper_function(*args, **kwargs):
    # 装饰器逻辑
    result = func(*args, **kwargs)
    # 装饰器逻辑
    return result
  return wrapper_function

# 类装饰器
class decorator_class(object):
  def __init__(self, func):
    self.func = func

  def __call__(self, *args, **kwargs):
    # 装饰器逻辑
    result = self.func(*args, **kwargs)
    # 装饰器逻辑
    return result

@decorator_class
def function_name():
  pass

# 等价于
def function_name():
  pass
function_name = decorator_class(function_name) # 实例化装饰器类 
```



## 模块
模块包含了一些常用的函数和类，可以直接导入使用。

模块的导入方式有两种：
- import 模块名
- from 模块名 import 成员名


__name__ 是模块的名称，当模块被直接运行时，__name__ 的值为 '__main__'，当模块被导入时，__name__ 的值为模块名。


dir() 函数可以查看模块内定义的所有名称。如果没有指定参数，则返回当前模块的所有名称。

```py
import math

print(dir(math))
```


包时一种管理模块的结构，可以将相关的模块放在一个包中，通过包名来导入模块。

包的导入方式：
- import 包名
- from 包名 import 成员名

包的目录结构：
- 包名
  - __init__.py
  - 模块1.py
  - 模块2.py
  - ...

目录中有一个 __init__.py 文件，它是包的初始化文件，当包被导入时，该文件会被自动执行。

包的作用：
- 避免命名冲突
- 组织代码
- 隐藏实现细节
- 共享代码


从一个包中导入 * ，可以导入包中的所有模块。

存在一个 __all__ 列表，它包含了包中所有模块的名称，如果没有 __all__ 列表，则默认导入包中的所有模块。

```py
from mypackage import *
``` 

## 异常处理

Python 使用 try-except 语句来处理异常。

```py
try:
  # 可能发生异常的代码
except ExceptionName:
  # 异常处理代码
```

如果没有指定异常类型，则会捕获所有的异常。

```py
try:
  # 可能发生异常的代码
except:
  # 异常处理代码
```

如果有多个异常类型需要处理，可以用元组来指定多个异常类型。

```py
try:
  # 可能发生异常的代码
except (Exception1, Exception2):
  # 异常处理代码
```

如果想在异常发生时，执行一些代码，可以用 else 语句。

```py
try:
  # 可能发生异常的代码
except:
  # 异常处理代码
else:
  # 正常处理代码
```

如果想在异常发生时，执行一些代码，并且不再向上层抛出异常，可以用 finally 语句。

```py
try:
  # 可能发生异常的代码
except:
  # 异常处理代码
else:
  # 正常处理代码
finally:
  # 最终处理代码
```

## 调试

Python 提供了 pdb 调试器，可以设置断点、单步执行代码、查看变量、查看调用栈等。

```py
import pdb

def my_func(x):
  y = x + 1
  pdb.set_trace()
  z = y + 2
  return z

my_func(1)
```

## 输入和输出


### 输入

Python 使用 input() 函数来获取用户输入。

```py
name = input("请输入你的名字：")
print("你好，" + name)
```


### 输出

Python 使用 print() 函数来输出内容到控制台。

```py
print("Hello, world!")
```

print() 函数可以接受多个参数，参数之间用逗号隔开。

```py
print("Hello,", "world!")
```

print() 函数还可以指定输出内容的格式，如：

```py
print("Hello, world!", end="")
print("Hello, world!")
```

end="" 参数用于指定 print() 函数的换行方式，默认是换行。

### 文件输入输出

Python 使用 open() 函数来打开文件，并返回一个文件对象。

```py
f = open("test.txt", "r")
```

open() 函数的第一个参数是文件名，第二个参数是打开模式，"r" 表示以读方式打开文件。

打开模式：
- "r"：读模式，文件必须存在，否则抛出 FileNotFoundError 异常。
- "w"：写模式，文件不存在时创建，存在时覆盖。
- "a"：追加模式，文件不存在时创建，存在时在末尾追加。
- "r+"：读写模式，文件必须存在，否则抛出 FileNotFoundError 异常。
- "w+"：读写模式，文件不存在时创建，存在时覆盖。
- "a+"：读写模式，文件不存在时创建，存在时在末尾追加。


文件对象有以下方法：
- read()：读取文件内容，返回字符串。
- readline()：读取文件的一行内容，返回字符串。
- readlines()：读取文件的所有内容，返回列表。
- write()：写入文件内容，返回写入的字符数。
- close()：关闭文件。


```py
# 读模式
f = open("test.txt", "r")
content = f.read()
print(content)
f.close()


# 写模式
f = open("test.txt", "w")
f.write("Hello, world!")
f.close()


# 追加模式
f = open("test.txt", "a")
f.write("Hello, world!")
f.close()
``` 

## 面向对象

类的专有方法：
- __init__()：构造函数，在对象创建时调用。
- __str__()：打印对象时调用，返回字符串。
- __del__()：对象被删除时调用。
- __len__()：返回对象的长度。


## 一些标准库

- os 模块
- sys 模块
- math 模块
- random 模块
- datetime 模块
- json 模块
- re 模块   正则模块
- time 模块 处理时间函数
- urllib 模块 ,访问网页处理URL功能



## other


```py
pip freeze #查看安装的包
pip install 包名 #安装包
pip uninstall 包名 #卸载包
pip list #查看已安装的包

pip freeze > requirements.txt #导出依赖包
pip install -r requirements.txt #安装依赖包
```