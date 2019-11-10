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

## 10.3 标准库：一些深受欢迎的模块

### 10.3.1 sys
- 模块sys中一些重要的函数和变量  

函数/变量   | 描述  
---------   | ---
argv        | 命令行参数，sys.argv[0] 为脚本名
exit([arg]) | 退出当前程序，可通过可选参数指定返回值或错误消息
modules     | 一个字典，将模块名映射到加载的模块
path        | 一个列表，包含要在其中查找模块的目录的名称
platform    | 一个平台标识符，图linux2/win32/darwin
stdin       | 标准输入流 -- 一个类似于文件的对象(stdout/stderr同)
stdout      | 标准输出流
stderr      | 标准错误流  

### 10.3.2 os
- 模块os中一些重要的函数和变量  

函数/变量       | 描述  
---------       | ---
environ         | 包含环境变量的映射
system(command) | 在子shell中执行操作系统命令
sep             | 路径中使用的分隔符
pathsep         | 分隔不同路径的分隔符
linesep         | 行分隔符('\n','\r'或'\r\n')
urandom(n)      | 返回n个字节的强加密随机数据  


- 使用webbrowser 模块打开网站  
```python
>>> import webbrowser as web
>>> web.open('https://www.baidu.com')
True
>>> web.open('https://notexitweb.com') #命令成正常执行，但是浏览器实际打开这个地址失败
True
>>> 
```

### 10.3.4 集合、堆和双端队列  
(1) 集合    

```python
#常见用法：
>>> a = set(range(1,6))
>>> a
{1, 2, 3, 4, 5}
>>> b={4,5,6,7,8}
>>> b
{1, 2, 3, 4, 5}
>>> a.union(b)
{1, 2, 3, 4, 5, 6, 7, 8}
>>> set.union(a,b)
{1, 2, 3, 4, 5, 6, 7, 8}
>>> a | b
{1, 2, 3, 4, 5, 6, 7, 8}
>>> a.intersection(b)
{4, 5}
>>> c = a & b
>>> c
{4, 5}
>>> c <= a
True
>>> c.issubset(a)
True
>>> a.issuperset(c)
True
>>> a.difference(b)
{1, 2, 3}
>>> a - b
{1, 2, 3}
>>> a.symmetric_difference(b)
{1, 2, 3, 6, 7, 8}
>>> a ^ b
{1, 2, 3, 6, 7, 8}
>>> d = a.copy()
>>> d
{1, 2, 3, 4, 5}
>>> d is a
False
>>> a.update(b)
>>> a
{1, 2, 3, 4, 5, 6, 7, 8}
>>> a.add(9)
>>> a
{1, 2, 3, 4, 5, 6, 7, 8, 9}
>>> a.discard(9)
>>> a
{1, 2, 3, 4, 5, 6, 7, 8}
>>> a.pop()
1
>>> a.pop()
2
>>> 
```

###### 集合是可变的，因此不能用作字典的键；集合只能包含不可变的(可散列)的值，因此不能包含其它集合。为了在集合中包含集合，可以使用frozenset类型：  
```python
>>> a={1,2,3}
>>> b={4,5,6}
>>> a.add(b)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'set'
>>> a.add(frozenset(b))
>>> a
{1, 2, 3, frozenset({4, 5, 6})}
>>> 
```

(2) 堆  

- 模块heapq中一些重要的函数  
用法： from heapq import *

函数       | 描述  
---------       | ---
heapify(heap)       | 让列表具备堆特征
heappush(heap,x)    | 将x压入堆中
heapop(heap)        | 从堆中弹出最小的元素
heapreplace(heap,x) | 弹出最小的元素，并将x压入堆中
nlargest(n, iter)   | 返回iter中n个最大的元素
nsmallest(n, iter)  | 返回iter中n个最小的元素

(3) 双端队列（及其它集合）
- 常见用法  
**注意** :用于extendleft的可迭代对象中的元素将按相反的顺序出现在双端队列中  
```python
>>> from collections import deque
>>> q = deque(range(5))
>>> q
deque([0, 1, 2, 3, 4])
>>> q.append(5)
>>> q.appendleft(0)
>>> q
deque([0, 0, 1, 2, 3, 4, 5])
>>> q.pop()
5
>>> q.popleft()
0
>>> q
deque([0, 1, 2, 3, 4])
>>> q.rotate(1)
>>> q
deque([4, 0, 1, 2, 3])
>>> q.rotate(-1)
>>> q
deque([0, 1, 2, 3, 4])
>>> q.rotate(-1)
>>> q
deque([1, 2, 3, 4, 0])
>>> q.extend(range(6,9))
>>> q
deque([1, 2, 3, 4, 0, 6, 7, 8])
>>> q.extendleft(range(100,103)) #用于extendleft的可迭代对象中的元素将按相反的顺序出现在双端队列中
>>> q
deque([102, 101, 100, 1, 2, 3, 4, 0, 6, 7, 8])
>>> 
```

### 10.3.5 time
- 模块time中一些重要的函数

函数                        | 描述  
---------                   | ---
time()                      | 当前时间(从新纪元开始后的秒数，以UTC为准)
localtime([secs])           | 将秒数转换为表示当地时间的日期元组
gmtime([secs])              | 将秒数转换为国际标准时间的日期元组
asctime([tuple])            | 将时间元组转换为字符串
mktime(tuple)               | 将时间元组转换为当地时间秒数,与localtime功能相反
sleep(secs)                 | 休眠secs秒
strptime(string[, format])  | 将字符串转换为时间元组


### 10.3.6 random
函数                                | 描述  
-                                   |-
random()                            | 返回一个0~1(含)的随机实数
getrandbits(n)                      | 以长整数方式返回n个随机的二进制位
uniform(a,b)                        | 返回一个a~b(含)的随机实数
randrange([start], stop, [step])    | 从range(start, stop, step) 中随机地选择一个数
choice(seq)                         | 从序列seq中随机地选择一个元素
shuffle(seq[,random])               | 就地打乱序列seq
sample(seq, n)                      | 从序列seq中随机地选择n个值不同的元素

###### 2019.11.10 Sun

### 10.3.7 shelve 和 json
- shelve  
shelve 可以将数据存储到文件中，但是却让用户像操作普通dict那样操作它  
```python
>>> import shelve
>>> sh=shelve.open('mydata.dat', writeback=True)
>>> sh['cart']=['cake','beer']
>>> sh['cart']
['cake', 'beer']
>>> sh['cart'].append('egg') #open()函数中的‘writeback=True’使得变更value的操作能够反映到文件中；否则，变更value的操作只会返回一个新的value，原始的value不会有变化
>>> sh['cart']
['cake', 'beer', 'egg']
>>> sh.close() #正确地关闭shelve后，数据就被保留到了 mydata.dat 文件中
>>> 
```

作为对比，下面是不适用 writeback=True 参数的效果：  
```python
>>> sh2=shelve.open('mydata2.dat')
>>> sh2['test']=['a','b','c']
>>> sh2['test'].append('d')
>>> sh2['test']
['a', 'b', 'c']
>>> 
>>> tmp=sh2['test'] #必须将字典的value读取到内存中，单独操作，然后再赋值回去，
>>> tmp
['a', 'b', 'c']
>>> tmp.append('d')
>>> sh2['test']=tmp
>>> sh2['test']
['a', 'b', 'c', 'd']
>>> sh2.close()
>>> 
```

### 10.3.8 re
- 1- 要让特殊字符的行为与普通字符一样，可对其进行转义：在特殊字符前面加上反斜杠。  
模式'python\\\\.org' 只与 'python.org' 匹配。
> **注意** ： 为表示模块re要求的单个反斜杠，需要在字符串中书写两个反斜杠，让解释器对其进行转义。换而言之，这里包含两层转义：***解释器执行的转义*** 和 ***模块re执行的转义*** 。实际上，在有些情况下也可以使用单个反斜杠，让解释器自动对其进行转义，但请不要依赖解释器。如果不想使用两个反斜杠，可使用原始字符串，如 r'python\\.org'


- 2- 模块re中一些重要的函数

函数                        | 描述  
---------                   | ---
compile(pattern[, flags])   | 根据包含正则表达式的字符串创建模式对象
search(pattern, string[, flags])           | 在字符串中查找模式
match(pattern, string[, flags])     | 在字符串开头匹配模式
split(pattern, string[, maxsplit=0])    | 根据模式来分隔字符串
findall(pattern, string)                | 返回一个列表，其中包含字符串中所有与模式匹配的子串
sub(pat, repl, string[, count=0])       | 将字符串中与模式pat匹配的子串都替换为repl
escape(string)                          | 对字符串中所有的正则表达式特殊字符都进行转义

>函数re.compile将用字符串表示的正则表达式转换为模式对象，以提高匹配效率。调用search、match等函数时，如果提供的是用字符串表示的正则表达式，都必须在内部将它们转换为模式对象。通过使用函数compile对正则表达式进行转换后，每次使用它时都无需再进行转换。  
>>模式对象也有搜索/匹配方法，因此 re.search(pat, string) 与 pat.search(string) 等价 (其中，pat是使用re.compile创建的模式对象)

- 2.1- 程序示例
```python
#拆分字符串
>>> some_text="alpha,beta,,,gamma   delta"
>>> re.split('[, ]+', some_text)
['alpha', 'beta', 'gamma', 'delta']
#maxsplit 指定分割的次数
>>> re.split('[, ]+', some_text, maxsplit=2)
['alpha', 'beta', 'gamma   delta']
>>> 

#查找字符串中所有的单词
>>> pat='[a-zA-Z]+'
>>> text='"Hm... Err -- are you sure?" he said, sounding insecure.'
>>> re.findall(pat, text)
['Hm', 'Err', 'are', 'you', 'sure', 'he', 'said', 'sounding', 'insecure']
>>> 
#查找字符串中所有的标点符号
>>> pat=r'[.?\-",\']+'
>>> re.findall(pat, text)
['"', '...', '--', '?"', ',', '.']
>>> 
```

- 3- 匹配对象和编组  
MatchObject的重要方法  

方法                    | 描述  
---------               | ---
group([group1, ...])    | 获取与给定子模式(编组)匹配的子串
start([group])          | 返回与给定编组匹配的子串的起始位置  
end([group])            | 返回与给定编组匹配的子串的终止位置(与切片一样，不包含终止位置)
span([group])           | 返回与给定编组匹配的子串的起始和终止位置  

- 3.1- 程序示例 
```python
>>> m = re.match(r"www\.(.*)\..{3}", 'www.python.org')
>>> m.group()
'www.python.org'
>>> m.group(1)
'python'
>>> m.start(1)
4
>>> m.end(1)
10
>>> m.span(1)
(4, 10)
>>> 
```
- 4 替换组中的组号和函数  

**在模式中添加注释** 
>要让正则表达式更容易理解，一种办法是在调用模块re中和函数时使用标志VERBOSE。这让你能够在模式中添加空白(空格、制表符、换行符等)，而re将忽略它们 ---- 除非将它们放在字符类中或使用反斜杠对其进行转义。在这样的表达式中，还可以添加注释  
```python
>>> emphasis_pattern = re.compile(r'''
... \*  #起始突出标志--一个星号
... (   #与要突出的内容匹配的编组的起始位置
... [^\*]+      #与除星号外的其它字符都匹配
... )   #编组到此结束
... \*  #结束突出标志
...     ''', re.VERBOSE)
>>>
>>> re.sub(emphasis_pattern, r'<em>\1</em>', 'Hello, *world*!')
'Hello, <em>world</em>!'
>>> 
```

**贪婪和非贪婪模式**  
```python
>>> rstr="*This* is *it*!"
>>> emp1=r'\*(.+)\*' #贪婪模式
>>> re.sub(emp1, r'<em>\1</em>', rstr)
'<em>This* is *it</em>!'
>>> emp2=r'\*(.+?)\*'  #对于所有的重复运算符，都可在后面加上问号来将其指定为非贪婪模式
>>> re.sub(emp2, r'<em>\1</em>', rstr)
'<em>This</em> is <em>it</em>!'
>>> 
```

### 10.3.9 其它有趣的标准模块
- argparse
- cmd
- csv
- datetime
- difflib
- enum
- functools
- hashlib
- itertools
- logging
- statistics
- timeit
- profile
- trace
- 


# 第11章 文件
## 11.1 打开文件
- f = open('filename', mode)  
函数open的参数mode的最常见取值  

值  | 描述
-   | -
'r' | 读取模式(默认值)
’w' | 写入模式
'x' | 独占写入模式
'a' | 附加模式
'b' | 二进制模式(与其他模式结合使用)
't' | 文本模式(默认值,与其它模式结合使用)
'+' | 读写模式(与其它模式结合使用)

> 'r+': 不会截断文件  
> 'w+': 会截断文件

- 关闭文件  
f.close()

### 11.2.5 使用文件的基本方法
file方法    | 描述
-           | -
read([n])   | 读取n个字节；如果不指定n，则从当前位置读取文件的全部内容
readline()  | 读取文件的一行
readlines()  | 读取文件的所有行，返回一个list，list的每一个元素就是文件的一行内容
wirte(string)  |  将字符串内容写入文件
writelines(list)    | 将list中的每一行按顺序都写入到文件中，不会自动添加换行符

### 11.3.5 文件迭代器


- 可以使用fileinput 模块迭代文件，这种做法成为‘延迟迭代’ -- 因为它只读取实际需要的文本部分  

```python
import fileinput
for line in fileinput.input(filename):
    process(line)
```

- 其它迭代文件的方式  
```python
#迭代文件
with open(filename) as f:
    for line in f:
        process(line)
        
#在不将文件对象赋给变量的情况下迭代文件
for line in open(filename):
    process(line)
    
#sys.stdin 也是可以迭代的
for line in sys.stdin:
    process(line)
```

- 可对迭代器做的事情基本上都可以对文件做  
如将其转换为字符串列表，其效果与readlines相同  
>**代码示例**  
```python
>>> f = open('testfile.txt', 'w')
>>> print("good better best", file=f)
>>> print("never let it rest", file=f)
>>> print("till good is better", file=f)
>>> print("and better is best", file=f)
>>> f.close()
>>> lines = list(open('testfile.txt'))
>>> lines
['good better best\n', 'never let it rest\n', 'till good is better\n', 'and better is best\n']
>>> line1,line2,line3,line4=open('testfile.txt')
>>> line1
'good better best\n'
>>> line2
'never let it rest\n'
>>> line3
'till good is better\n'
>>> line4
'and better is best\n'
>>> 
```