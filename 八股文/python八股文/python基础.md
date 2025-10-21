---
title: python基础
author: zhangxin
tags:
  - private
  - python
date: 2023-08-17 14:01:13
categories: python
---

### 函数式和面向对象编程有什么区别
- 函数式编程(FP)

  - 由数据驱动

  - 函数式编程强调的是将计算过程看作是数学函数的组合。它的主要特点包括：
    - 纯函数（Pure Functions）：函数式编程鼓励使用纯函数，即没有副作用（side effects）的函数。纯函数的输出仅依赖于输入，不会修改外部状态，这样可以降低代码的复杂度和维护成本，并提高代码的可测试性和可复用性。
    - 不可变数据（Immutable Data）：函数式编程中，数据被视为不可变的，即一旦创建就不可修改。这种不可变性使得代码更容易推理和并发执行，因为不存在对共享数据的竞争条件。
    - 高阶函数（Higher-order Functions）：函数式编程支持高阶函数，即可以将函数作为参数传递给其他函数，或者将函数作为返回值返回。这种功能可以让我们编写更具抽象和灵活性的代码。
    - 避免可变状态（Avoiding Mutable State）：函数式编程尽可能避免可变状态的使用，而是通过纯函数的组合来处理数据。这种方式避免了许多与并发编程相关的问题，并使代码更易于推理和调试。

- 面向对象编程(OOP)

  - 由操作驱动

  - 面向对象编程强调将问题看作是对象之间的交互。它的主要特点包括：
    - 封装（Encapsulation）：面向对象编程通过将数据和操作封装在对象中来实现封装。对象可以隐藏内部实现细节，只提供公共接口供其他对象使用。
    - 继承（Inheritance）：面向对象编程支持继承，即通过派生类可以继承父类的属性和方法。这样可以实现代码的重用和扩展。
    - 多态（Polymorphism）：面向对象编程支持多态，即同一个方法可以根据不同对象的类型执行不同的操作。这种特性可以提高代码的灵活性和可扩展性。
    - 可变状态（Mutable State）：面向对象编程通常使用可变状态，对象的属性可以在运行时修改。这使得面向对象编程在处理一些状态变化的问题上更直观和自然。

### 具名元组
- 示例

```python
import collections

Card = collections.namedtuple('card', ["rank", "suit"])


class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list("JQKA")
    suits = 'spades diamonds clubs hearts'.split()
    def __init__(self):
        self._card = [Card(rank, suit)
                      for rank in self.ranks for suit in self.suits]

    # 使用__len__和__getitem__之后 整个类的行为，就像标准python序列一样
    def __len__(self):
        return len(self._card)

    def __getitem__(self, item):
        return self._card[item]

A = FrenchDeck()
print(len(A))
print(A[2])
```



- 应用场景

  - 代替字典: 具名元组可以像字典一样存储一组相关的值,但它们的访问数据是更快的,因为它们是不可变的

  - 数据库结果查询: 数据库API检索行的时候,每行都可以表示为具有名称的元组,这样就可以使用名称访问值,而不是索引

  - 函数的返回值: 如果函数需要返回多个值,具名元组可以使返回值更清晰,并且可以使调用者更清晰的理解返回值的意思

- 代替类:
  - 但需要一个非常的简单的数据容器的时候,它比类更轻量级



### 特殊方法的使用
- 模拟数值使用

```python
# 二维向量类
import math

class Vector:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y
def __repr__(self):
    return f"Vector({self.x!r}, {self.y!r})"

def __abs__(self):
    # 计算直角三角形的斜边长
    return math.hypot(self.x, self.y)

def __bool__(self):
    return bool(abs(self))

def __add__(self, other):
    x = self.x + other.x
    y = self.y + other.y
    return Vector(x, y)

def __mul__(self, scalar):
    return Vector(self.x * scalar, self.y * scalar)

v1 = Vector(2, 4)
v2 = Vector(2, 1)
print(v1 + v2)

v = Vector(3, 4)
print(abs(v))
```



- 字符串的表现形式

  - 特殊方法__ repr __供内置方法repr调用,获取对象的字符串形式

    ```python
    class StrTest:
        def __init__(self):
            self.a = 1
    
        def __repr__(self):
            return "str_test类"
        
    class StrTest1:
        def __init__(self):
            self.a = 1
    
    
    print(StrTest())  # str_test类
    print(StrTest1())  # <__main__.StrTest1 object at 0x10464cfd0>
    ```

    

  - __ repr __方法中f字符串使用!r以标准的方式显示属性
    - f"Vector({self.x!r}, {self.y!r})"
  - __ repr __ 和 __  str __的区别
    - __ str __ 方法由内置的str()函数调用,在背后供print()使用,返回对终端用户友好的字符串,如果没有定义 __ str __ 方法,则会使用默认的__ repr __方法来返回字符串
    - __ repr __用于返回对象的“官方”字符串形式,repr(4)返回“4”,repr("4")返回“ ‘4’ ” 供交互解释器使用

  - __ bool __
    - 如果没有实现__ bool __方法,则会调用x.len()方法如果该方法返回零值则bool函数返回False,否则返回True

  - __ len __
    - len之所以不是方法调用就是因为,内置方法 __ len __ 的实现,运行速度更快

  - __ iadd __
    - 背后支持+=运算符的就是 __ iadd __ 特殊方法(就地相加)

  - __ new __
    - 创建实例
  - __ init __
    - 初始化实例

### list

- 可变
- list.sort()和内置函数sorted()的区别
  - list.sort()就地排序,不创建副本返回值为None
  - sorted()返回新创建的列表,接收任何可迭代的对象,包括不可变序列和生成器
  - 均接收两个参数
  - reverse: 默认False升序
  - key: 只接收一个参数的函数,应用到每一项上,作为排序依据

### tuple

- 不可变

- 不要在元组中存放可变元素

  ```python
  t = (1, 2, [3, 4])
  t[2] += [5, 6]
  # 报错'tuple' object does not support item assignment
  # 但是此时t = (1, 2, [3, 4, 5, 6])
  # 导致整个问题的原因就是使用了+=
  ```

  

### array

- from array import array
- 快速加载项和保存项array.fromfile,array.tofile,二者运行速度非常快



### 双端队列deque
- collections.deque 类实现一种线程安全的双端队列,长度固定,从一端添加新项的时候,将从另一端丢弃一项

  ```python
  from collections import deque
  dq = deque(range(10),maxlen=10)
  ```

  

  - dq.rotate(3) 从右端取几项放到左端

  - dq.rotate(-4) 从左端取几项放到右端

  - dp.appendleft(1) 从左端添加项,如果序列是满的,则从右端丢弃一项

  - dq.extendleft(iter) 依次把iter中的参数从左边添加到dq中



### 队列queue

- 提供几个同步(线程安全)的队列类SimpleQueue、Queue、LifoQueue 和 PriorityQueue。这些类可在线程之间安全通信。除 SimpleQueue 之外，其他几个类都可以有界——为构造函数提供 maxsize 参数，设为大于 0 的值

- 与deque不同的是但队列元素满了之后,会阻塞直到队列被取出一项利用这个行为可以限制活动线程的数量



### 数据类构建器

- from collections import namedtuple 具名元组

  ```python
  coornatide = namedtuple('Coordinate', 'lat lon')
  issubclass(coornatide, tuple)  # True
  moscow = coornatide(34.6, 34.6)
  moscow  # Coordinate(lat=34.6, lon=34,6)
  ```

  

- from typing import NamedTuple

  ```python
  coornatide = NamedTuple('Coordinate', [("lat", float), ("lon", float)])
  moscow = coornatide(34.6, 34.6)
  issubclass(coornatide, tuple)  # True
  typing.get_type_hints(coornatide)  # {'lat': <class 'float'>, 'lon': <class 'float'>} 
  ```

  ```python
  from typing import NamedTuple
  
  class Coordinate(NamedTuple):
      lat: float                
      lon: float
      reference: str = 'WGS84'  
  ```

- from dataclasses import dataclass

  ```python
  from dataclasses import dataclass
  
  @dataclass
  class DemoDataClass:
      a: int           
      b: float = 1.1  
      c = 'spam' 
  ```

  ```python
  @dataclass(init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False)
  ```

  - init：默认将生成__init__方法。如果传入False，那么该类将不会有__init__方法。

  - repr：__repr__方法默认生成。如果传入False，那么该类将不会有__repr__方法。

  - eq：默认将生成__eq__方法。如果传入False，那么__eq__方法将不会被dataclass添加，但默认为object.__eq__。

  - order：默认将生成__gt__、__ge__、__lt__、__le__方法。如果传入False，则省略它们

  - frozen默认值false

    

  - Frozen 实例是在初始化对象后无法修改其属性的对象

    ```python
    import math
    
    @dataclass
    class FloatNumber:
        val: float = 0.0
        def __post_init__(self):
            self.decimal, self.integer = math.modf(self.val)
    ```

    

- 类型提示主要为第三方类型检查工具提供支持，例如 Mypy 和 PyCharm IDE 内置的类型检查器。这些是静态分析工具，在“静止”状态下检查 Python 源码，不运行代码

### 深拷贝和浅拷贝

```python
a = [1,2,3,[4,5]]
b = a[:]
# 浅拷贝
# b和a的id是不一样的
# 但是a和b中[4,5]的id是一样的
```

```python
from copy import deepcopy

a = [[1], 2, 3, 4]
b = deepcopy(a)
b[0].append(1) 
# 深拷贝(值类型一样,id不同包括内部list)
# a和b的id不同
# 其中的[1]的id也不同
```



### del和垃圾回收
- del不是函数而是语句只是删除应用而不是删除对象

  ```python
  a = [1, 2]
  
  b = a
  del a
  print(b) # [1,2]
  # a不存在
  ```

- 弱引用
  - import weakref 对象弱引用不增加引用计数



### 装饰器
- 装饰器本质上是一个高级Python函数，通过给别的函数添加@标识的形式实现对函数的装饰
- 装饰器的功能：它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。它经常用于有切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等场景。装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。

- python中装饰器是随着程序的加载运行而自动加载的，跟调不调用方法没有关系.所以只要是装饰器内部函数以外的部分都会自动加载执行，不用调用。
- 装饰器是随着程序的执行而加载的，不是调用函数也会自动加载。
- 装饰器原理：@装饰器名（@makeBold） 放在一个函数头上相当于将这个函数整体当做参数传递给这个装饰函数去执行，即等价于test_Bold=makeBold(test_Bold)，装饰器的使用大大简化了程序的代码。

- 标准库中的装饰器

  - @functools.cache(3.9新增)
  - @functools.lru_cache(3.8)(maxsize控制内存大小)
  - @functools.lru_cache()(3.8之前) 实现备忘,把耗时函数得到的结果进行保存,避免传入相同参数时

  ```python
  import functools
  import time
  
  
  def clock(func):
      def clocked(*args):
          t0 = time.perf_counter()
          result = func(*args)
          elapsed = time.perf_counter() - t0
          name = func.__name__
          arg_str = ', '.join(repr(arg) for arg in args)
          print(f'[{elapsed:0.8f}s] {name}({arg_str}) -> {result!r}')
          return result
      return clocked
  
  @functools.cache
  @clock
  def fibonacci(n):
      if n < 2:
          return n
      return fibonacci(n - 2) + fibonacci(n - 1)
  ```

  

- @functools.singledispatch
  - 使用 @singledispatch 装饰的普通函数变成了泛化函数（generic function，指根据第一个参数的类型，以不同方式执行相同操作的一组函数）的入口。这才称得上是单分派。如果根据多个参数选择专门的函数，那就是多分派

- 装饰器类

  ```python
  class Decorator:
      def __init__(self, arg1, arg2):
      print('执行类Decorator的__init__()方法')
      self.arg1 = arg1
      self.arg2 = arg2
  
  def __call__(self, f):
      print('执行类Decorator的__call__()方法')
  
      def wrap(*args):
          print('执行wrap()')
          print('装饰器参数：', self.arg1, self.arg2)
          print('执行' + f.__name__ + '()')
          f(*args)
          print(f.__name__ + '()执行完毕')
  
      return wrap
  
  @Decorator('Hello', 'World')
  def example(a1, a2, a3):
      print('传入example()的参数：', a1, a2, a3)
  
  
  print('装饰完毕')
  print('准备调用example()')
  example('Wish', 'Happy', 'EveryDay')
  print('测试代码执行完毕')
  # 执行结果
  # 执行类Decorator的__init__()方法
  # 执行类Decorator的__call__()方法
  # 装饰完毕
  # 准备调用example()
  # 执行wrap()
  # 装饰器参数： Hello World
  # 执行example()
  # 传入example()的参数： Wish Happy EveryDay
  # example()执行完毕
  # 测试代码执行完毕
  ```

  

### memoryview

- 一种共享内存的序列类型
