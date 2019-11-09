# 《Python基础教程(第3版)》学习笔记
[TOC]
###### 2019.11.07 Thu
# 第9章 魔法方法、特性和迭代器
## 9.2 构造函数
### 9.2.2 调用未关联的超类构造函数

```python
class Bird:
    def __init__(self):
        self.hungry = True def eat(self):
    if self.hungry:
        print('Aaaah ...')
        self.hungry = False
    else:
        print('No, thanks!')

class SongBird(Bird): 
    def __init__(self):
        Bird.__init__(self)  #未关联的超类的构造函数
        self.sound = 'Squawk!' 
    def sing(self):
        print(self.sound)
```

### 9.2.3 调用函数super
```python
class SongBird(Bird): 
    def __init__(self):
        super().__init__() #调用函数super
        self.sound = 'Squawk!' 
    def sing(self):
        print(self.sound)
```

>***使用函数super的优点：***  
(1) 使用函数super更直观  
(2) 即使有多个超类，也只需调用函数super一次(条件是所有的超类的构造函数也使用函数super)  
(3) 函数super返回的是一个super对象，这个对象将负责为你执行方法解析。当访问它的属性(包括成员变量和成员函数)时，它将在所有的超类中查找，直到找到指定的属性或引发AttributeError异常  


## 9.3 元素访问
### 9.3.1 基本的序列和映射协议
序列和映射基本上是 **元素(item)** 的集合，要实现它们的基本行为(协议)，不可变对象需要实现下面的前2个方法，而可变对象需要实现下面的4个方法  
- 序列和映射的方法
1. \_\_len__(self):  
    这个方法返回集合包含的项数，对序列来说为元素个数，对映射来说为 *键-值* 对数
2. \_\_getitem__(self, key):  
    这个方法返回与指定键相关联的值
3. \_\_setitem(self, key, value):  
    这个方法应以与键相关联的方式存储，以便以后能够使用__getitem__来获取；仅当对象可变时才需要实现这个方法。
4. \_\_getitem__(self, key):  
    这个方法在对对象的组成部分使用__del__语句时被调用，应删除与key相关联的值。同样，仅当对象可变(且允许其项被删除)时，才需要实现这个方法。  
### 9.3.2 从 list、dict和str派生
1. 从list派生
```python
class CounterList(list):
    def __init__(self, *args):
        super().__init__(*args)
        self.counter = 0
    def __getitem__(self, index):
        self.counter += 1
        return super(CounterList, self).__getitem__(index)
```
## 9.5 特性
### 9.5.1 函数 property
通过调用函数property并将存取方法作为参数( **获取方法** 在前， **设置方法** 在后) 创建一个特性，然后将名称the_size(任意名称均可)关联到这个特性。  
```python
class Rectangle:
    def __init__ (self):
        self.width = 0
        self.height = 0
    def set_size(self, size):
        self.width, self.height = size def get_size(self):
        return self.width, self.height 
    the_size = property(get_size, set_size) # 注意，本行并没有缩进两次；与def处在同级缩进
```

## 9.6 迭代器
### 9.6.1 迭代器协议
> **定义**  
实现了方法 \_\_iter__ 的对象是 **可迭代** 的，而实现了方法 \_\_next__ 的对象是 **迭代器**  
如果迭代器没有可供返回的值，应引发 StopIteration 异常 (在 for ... in ... 循环以及其它使用迭代器的函数中，该异常能够被妥善处理)
```python
class MyIter:
  value = 0
  def __next__(self):
    tmp = self.value
    self.value += 1
    if self.value > 10:
      self.value = 0
      raise StopIteration
    else:
      return tmp
  def __iter__(self):
    return self
```

```python
#上面的迭代器的特点是迭代完毕后，可以从头重新迭代，实例如下
>>> a=MyIter()
>>> list(a)
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list(a)
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> next(a)
0
>>> next(a)
1
>>> 
```  
###### 2019.11.08 Fri
## 9.7 生成器
### 9.7.1 创建生成器

- 例1  
包含yield语句的函数都被称为生成器
```python
nested = [[1, 2], [3, 4], [5]]
def flatten(nested):
    for sublist in nested:
        for element in sublist: 
            yield element
```
- 例2  
生成器推导（也叫生成器表达式）：其工作原理与列表推导相同，但不是创建一个列表（即不立即执行循环），而是返回一个生成器，后续可以逐步执行计算
```python
>>> g = ((i + 2) ** 2 for i in range(2, 27))
>>> next(g)
16
>>> next(g)
25
>>> 
```
- 例3  
直接在一对既有的圆括号内(如 在函数调用中)使用生成器推导时，无需再添加一对圆括号
```python
>>> sum(i ** 2 for i in range(10))
285
>>> 
```
- 例4  
递归式生成器
```python
def flattern(nested):
  try:
    for sublist in nested:
      for element in flattern(sublist):
        yield element
  except TypeError:
    yield nested

    
>>> b=[1,2,3,[4,5,5],[7,8,[9,10,23,[3,4,5,]]]]
>>> list(flattern(b))
[1, 2, 3, 4, 5, 5, 7, 8, 9, 10, 23, 3, 4, 5]
>>> 
```
>这个递归版本的生成器函数flattern，不应该对类似于字符串的对象进行迭代，主要原因有两个。首先，类似于字符串的对象应该视为原子值，而不是应该展开的序列。其次，对这样的对象进行迭代会导致无穷递归，因为字符串的第一个元素是一个长度为1的字符串，而长度为1 的字符串的第一个元素是字符串本身！  
- 例5  
要处理例4对字符串的无穷递归问题，必须在生成器开头进行检查。最快捷的方式是：尝试将参数nested与一个字符串拼接，并检查这是否会引发TypeError异常。如下：  
```python
def flattern(nested):
  try:
    #不迭代类似于字符串的对象
    try:
        nested + ''
    except TypeError:
      #说明不是字符串，可以继续执行函数
      pass
    else:
      #说明是字符串，直接返回字符串本身
      raise TypeError
    for sublist in nested:
      for element in flattern(sublist):
        yield element
  except TypeError:
    yield nested
    
    
    
>>> list(flattern('this is a string'))
['this is a string']
>>> 
```
##### summary:
>生成器由两个单独的部分组成： **生成器的函数** 和 **生成器的迭代器** 。生成器的函数是由def语句定义的，其中包含yield。生成器的迭代器是这个函数返回的结果。这两个实体通常被视为一个，统称为生成器。  

### 9.7.4 生成器的方法
- **外部世界**  
外部世界可访问生成的方法send，这个方法类似于next，但接受一个参数(要发送的‘消息’，可以是任何对象)。
- **生成器**  
在挂起生成器的内部，yield可能用作 **表达式** 而不是 **语句** 。换而言之，当生成器重新运行时，yield返回一个值 -- 通过send从外部世界发送的值。如果使用的是next，yield将返回None。  
>需要注意的是，仅当生成器被挂起(即遇到第一个yield)后，使用send(而不是next)才有意义。要在此之前向生成器提供信息，可以使用生成器的函数的参数。

- 例1  
```python
def repeater(value):
  while True:
    value += 1
    new = (yield value)
    print("I got new={}, now value = {}".format(new, value))
    if new == 'bye':
      print("bye bye~")
    elif new is not None:
      value = new
      
      
#运行实验     
>>> r = repeater(100)
>>> next(r)
before yield, value=101
101
>>> next(r)
I got new=None, now value = 101

before yield, value=102
102
>>> next(r)
I got new=None, now value = 102

before yield, value=103
103

#从以上输出可以看出，调用生成器函数后，函数并没有实际运行，直到调用了next()函数，
生成器函数返回的生成器才第一次真正运行，并且在yield后挂起。再次调用next，
将从上一次挂起的yield语句后继续运行，直到再次遇到yield语句，则输出结果并挂起；
且调用next()函数使yeild语句返回值为None。

>>> r.send(200)
I got new=200, now value = 103
new(200) is assigned to value
before yield, value=201
201
>>> next(r)
I got new=None, now value = 201

before yield, value=202
202
>>> next(r)
I got new=None, now value = 202

before yield, value=203
203

#从以上输出可以看出，对挂起的生成器调用send()函数，参数将作为yield语句的返回值。
同时也可以看出，yield语句挂其后，其并没有返回值，
必须等到r.send(new_value) 或 next(r) 调用后，
才从yield语句返回，且这时的yield语句的生成值已经在上一轮迭代中输出了。

>>> 
>>> s=repeater(300)
>>> s.send(400)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't send non-None value to a just-started generator  
>>>
#从以上输出结果可以看出，仅当生成器被挂起后，使用send(而不是next)才有意义。
 
```

###### 2019.11.09 Sat
# 第10章 开箱即用
### 10.1.4 包
包是另一种模块，并且可以包含其它模块。模块存储在.py文件中，而包则是一个目录。要被Python视为包，目录必须包含文件__init__.py。如果像普通模块一样导入包，文件__init__.py的内容就将是包的内容。

- 一种包的布局   

 文件/目录                          | 描述
------------------------------      | ----
~/python/                           | PYTHONPATH的目录
~/python/drawing/                   | 包目录(包drawing)
~/python/drawing/\_\_init__.py      | 包代码(模块drawing)
~/python/drawing/colors.py          | 模块colors
~/python/drawing/shapes.py          | 模块shapes


完成上面的准备工作后，下面的语句都是合法的：   
导入方式                    | 描述
----------------------------| --------
import drawing              |   (1）导入drawing包(并没有导入colors和shapes)
import drawing.colors       |   (2) 导入drawing包中的模块colors
from drawing import shapes  |   (3) 导入模块shapes


### 10.2.1 模块包含什么  
假设 mypack/Magic.py 内容如下:  
```python
#/usr/bin/python
'''
good better best
never let it rest
'''
__all__ = ['repeater']
class MySequence:
  '''
  this is my special sequence
  '''
  def __init__(self, start=0, step=1):
    self.start = start
    self.step = step
    self.changed = {}
    print("start={}, step={}".format(start, step))
    
    ......
```
- 使用dir  
```python
>>> import mypack.Magic as magic
>>> dir(magic)
['MyIter', 'MyList', 'MyMethod', 'MySequence', 'Rect', '__all__', '__builtins__', '__cached__', '__doc__', '
__file__', '__loader__', '__name__', '__package__', '__spec__', 'flattern', 'repeater']
>>>   
```
- 使用help  
```python
>>>help(magic)

Help on module mypack.Magic in mypack:

NAME
    mypack.Magic

DESCRIPTION
    good better best
    never let it rest

FUNCTIONS
    repeater(value)

DATA
    __all__ = ['repeater']

FILE
    /Volumes/Files/icode/python3/mypack/Magic.py
```

- 变量 \_\_all__ , \_\_doc__ 和 \_\_file__
```python
>>> magic.__all__
['repeater']
>>> magic.__doc__
'\ngood better best\nnever let it rest\n'
>>> magic.__file__
'/Volumes/Files/icode/python3/mypack/Magic.py'
>>> 
```


##### 使用列表推导查看模块中对外提供的接口  
```python
>>> import copy
>>> [n for n in dir(copy) if not n.startswith('_')]
['Error', 'copy', 'deepcopy', 'dispatch_table', 'error']
>>> 
```
