# python基础

### py项目基础

#### 项目创建、管理

poetry

<img src="assets/image-20260125003048176.png" alt="image-20260125003048176" style="zoom:80%;" />

toml文件概述

<img src="assets/image-20260125004906266.png" alt="image-20260125004906266" style="zoom:80%;" /> 

```toml
# 1. 项目核心元数据 (必须)
[tool.poetry]
name = "my-awesome-project"
version = "0.1.0"
description = "一个用于演示的AI项目"
authors = ["你的名字 <you@example.com>"]
readme = "README.md"
license = "MIT"

# 项目支持的Python版本范围，这是环境一致性的基础！
python = "^3.8"

# 2. 生产依赖声明 (必须)
[tool.poetry.dependencies]
python = "^3.8"
requests = "^2.28.2"     # “^” 表示兼容 2.28.2 及以上但低于 3.0.0
pandas = ">=1.5,<2.0"    # 范围指定，兼容 1.5 到 2.0 之间（不含2.0）
numpy = "*"              # “*” 表示安装任何版本（慎用）
flask = { version = "^2.0", optional = true } # 可选依赖，需手动启用

# 3. 开发依赖声明 (可选，仅用于开发、测试)
[tool.poetry.group.dev.dependencies]
pytest = "^7.0"
black = "^23.0"
jupyter = "^1.0"

# 4. 依赖分组 (Poetry 1.2.0+ 特性，用于组织更复杂的依赖)
[tool.poetry.group.docs.dependencies]
sphinx = "^5.0"

# 5. 构建脚本配置 (可选，定义项目如何被打包)
[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

管理依赖要在项目的根目录下执行

- poetry add dep_name 添加依赖
- poetry add --dev dep_name 添加生产环境依赖
- poetry add "flask>=2.0,<3.0"  指定版本范围
- poetry add "numpy@^1.21"  兼容1.21及以上，但低于2.0
- poetry add git+https://github.com/some/private.git 从其他源添加

当克隆一个已有Poetry项目，或需要重装环境时

- poetry install --no-dev 一致安装：测试、开发、生产环境一致
- poetry install  根据lock文件安装所有环境的依赖
- poetry lcok 修复lock文件

更新依赖时谨慎更新，先查看具体包的可用情况，在lock文件修改对应版本范围再执行更新，千万不要更新所有依赖到新版本

- poetry show --outdated 查看可用更新
- poetry update dep_name 更新具体包

规范的poetry项目结构

```
your_project/
├── pyproject.toml          # 项目声明（必须）
├── poetry.lock             # 版本锁（必须提交到Git）
├── .gitignore              # 忽略虚拟环境等
├── README.md
├── your_package/           # 你的主要代码
│   ├── __init__.py
│   └── main.py
└── tests/                  # 测试代码
    └── __init__.py
```

#### 命名空间

当py解释器访问变量时按`命名空间`顺序**局部命名空间->嵌套命名空间->全局命名空间->内置命名空间**依次查找 (**LEGB 规则**)，当这四个命名空间都未查找到时抛出NameError

| 命名空间类型      | 创建时机            | 销毁时机               | 说明与访问方式                                               |
| :---------------- | :------------------ | :--------------------- | :----------------------------------------------------------- |
| 内置（Built-in）  | Python 解释器启动时 | 解释器退出             | 包含所有内置函数（如 `print`）和异常。 `dir(__builtins__)` 查看。 |
| 全局（Global）    | 模块被导入或执行时  | 解释器退出或模块被删除 | 模块顶层定义的变量、函数、类。 `globals()` 返回字典。        |
| 嵌套（Enclosing） | 外层函数被调用时    | 外层函数返回           | 存在于嵌套函数中，外层函数的局部命名空间。 内层函数可通过 `nonlocal` 修改。 |
| 局部（Local）     | 函数/方法被调用时   | 函数返回（除非闭包）   | 函数内部定义的参数和变量。 `locals()` 在函数内返回字典。     |

**自动扫描机制**（静态作用域分析、变量收集）

变量作用域的决定方式却是**静态**的，这个过程发生在**编译时**（即代码被编译为字节码的阶段）

```python
a = 1

def test_local():
    print(a) # 运行此行时直接在局部命名空间查找，不会向上查找，Unresolved reference 'a' 
    a = 2 # 编译时认为a是一个局部命名空间变量
```

> “**编译时作用域分析**”是指 Python 编译器在将源代码（.py文件）转换成字节码（.pyc文件中的对象）的过程中，会扫描每个作用域（主要是函数作用域）的代码，并静态地决定其中每个名字的性质和查找方式
>
> “**变量收集**”是编译时作用域分析的直接产出物。编译器会为每个代码块（函数、类、模块）创建一个符号表，并将该作用域内的变量分门别类地“收集”起来，存储到代码对象（`PyCodeObject`）的不同元组中。这些分类直接决定了运行时使用何种字节码指令来访问变量

**globals()、locals()方法**

`globals()`、`locals()`是 Python 内置的两个函数，它们分别用于获取当前作用域下的全局命名空间和局部命名空间的字典视图，值得注意的是，`locals()` 在函数内部返回的是局部命名空间的**一次拷贝**（快照），而`globals()` 返回的是对当前模块全局命名空间的**直接引用**（而不是拷贝）

```py 
# 示例：globals() 的基本使用
x = 10
def foo():
    y = 20
    print(globals()['x'])  # 访问全局变量 x，输出 10
    globals()['z'] = 30    # 动态创建全局变量 z

foo()
print(z)  # 输出 30，说明 z 确实被添加到了全局命名空间
```

```py
def func(a, b):
    c = a + b
    local_vars = locals()
    print(local_vars)          # {'a': 1, 'b': 2, 'c': 3}
    local_vars['d'] = 100       # 尝试添加局部变量 d
    print(locals())            # 再次调用 locals()，仍然没有 d，且修改无效
    c = 999
    print(local_vars['c'])      # 输出 3，而不是 999，因为 local_vars 是之前的拷贝

func(1, 2)
```

**global关键字**

`global` 是 Python 中用于在函数内部操作全局变量的关键字。它告诉解释器：在函数内对该名称的赋值操作，应该作用在全局作用域（模块级别）的变量上，而不是创建一个新的局部变量（不添加global关键字时的默认行为）。

注意，global关键字语句必须**放在函数的第一行**！

```py
a= 250
def test1():
    a = 100
    print(a) #100
test1()
print(a) #250
```

```py
a= 250
def test1():
    global a
    a = 100
    print(a) #100
test1()
print(a) #100
```

```py
a= 250
def test1():
    print(a)
    global a
test1()
# global a
#     ^^^^^^^^
# SyntaxError: name 'a' is used prior to global declaration
```

#### 闭包closure

闭包指函数式编程中`函数与其词法环境的结合体`，闭包**允许函数访问并操作外部变量，即使外部函数已执行完毕**。从外层函数来看，本质上是**为函数状态的维持提供了封装支持**，且每次执行时的状态都可以拥有初始值。

产生闭包的条件

- **函数嵌套**：一个函数内部定义了另一个函数。
- **内部函数引用外部函数的变量**：内部函数使用了外部函数中声明的变量。
- **外部函数将内部函数作为返回值返回**：这样内部函数可以在外部函数执行结束后被调用。

当这三个条件满足时，即使外部函数已经返回，内部函数仍然“记住”了它当时所引用的外部变量，这些变量不会被回收内存，因为它们仍然被内部函数引用着。

闭包的意义是**提供了一种轻量级的、可维护的“状态封装”方式**

``` python
function outerFunction(x) {
  // 外部函数的变量
  let outerVariable = 'I am from outer';

  function innerFunction(y) {
    // 内部函数引用了外部函数的变量
    console.log(outerVariable);
    console.log(x + y);
  }

  return innerFunction; // 返回内部函数
}

const myClosure = outerFunction(10); // 执行外部函数，得到内部函数
myClosure(5); // 调用内部函数，输出：I am from outer 和 15
```

在这里总结下闭包的特点

- 持久性：闭包使得函数可以记住它被创建时的环境，即使环境已经消失。
- 封装性：可以用来模拟私有变量或方法，实现数据隐藏和模块化。
- **动态性**：每次外部函数执行都会创建一个新的闭包，它们彼此独立。

常见的应用场景：轻量级封装数据（**私有化**）、**函数工厂**（根据参数生成特定行为的函数，比如乘法器）、**回调函数中延迟执行**

``` py
def counter(start=0):
    count = start
    def increment():
        nonlocal count   # 声明 count 不是局部变量，而是来自外层
        count += 1
        return count
    return increment

c = counter(10)
print(c())  # 11
print(c())  # 12
```

```py
function multiplyBy(n) {
  return function(x) {
    return n>0 ? x*n : x+n ;
  };
}
const double = multiplyBy(2);
console.log(double(5)); // 10
```

```py
import threading

def delayed_greeting(name, delay):
    def callback():
        print(f"Hello, {name}!")
    # Timer 会在指定时间后调用 callback
    timer = threading.Timer(delay, callback)
    timer.start()

delayed_greeting("Alice", 2)   # 2秒后打印 "Hello, Alice!"
```

`自由变量`即外层变量会被封装为一个`cell`对象（**闭包单元**），cell被保存在内层函数的**\_\_closure__属性**中（cell的元组），内层函数正是通过调用cell去获取自由变量的值的

**nonlocal关键字**

闭包变量在inner方法中可以读取不能修改，要想修改必须用`nolocal`关键字声明为nolocal变量。

``` py
# nolocal关键字
def outer_():
    a = 1

    def inner():
        nonlocal a # 缺少将报错
        a = a + 1
        return a

    return inner(), a

if __name__ == '__main__':
    print(outer_())  # (2, 2)
```

**延迟绑定在闭包中的问题**

在循环中使用闭包容易掉入“**延迟绑定**”的陷阱：函数在调用的时候才去查找变量的值，此时i的值为2，funcs中每一个show拥有的自由变量i值都为2，本质上来说每个show都在共享同一个i的cell

```py
# 一个经典的错误示例
funcs = []
for i in range(3):
    def show():
        print(i)
    funcs.append(show)
for f in funcs:
    f()  # 全部输出 2！因为所有闭包共享同一个变量 i 的最终值
# 正确做法：使用默认参数或函数工厂在每次迭代时“冻结” i 的值
funcs = []
for i in range(3):
    def show(x=i):  # 默认参数在定义时求值，绑定当前 i 的值
        print(x)
    funcs.append(show)
```

**函数装饰器**

原理是通过闭包来保持被增强函数的持久性，即增强动作执行时最初的func动作可以在适当的时机被调用

```py
# 装饰函数
def enhance(func):
    @wraps(func) # 把 func 的元数据复制给 enhance_content,执行help(enhance)时可以看到被装饰函数的信息
    def enhance_content():
        print('pre functioning')
        func()
        print('post functioning')
    return enhance_content

# 被装饰函数
# 这里的本质是func = enhance(func)
@enhance
def func():
    print('functioning')

if __name__ == '__main__':
    func()
    # pre functioning
    # functioning
    # post functioning
```

如果要给装饰器指向的函数传递参数，必须在该函数写层"**三层嵌套**"的形式

```py
def say_hello(msg):
    def outer(func):
        def wrapper(*args, **kwargs):
            print(f'你好，我要开始{msg}计算了')
            return func(*args, **kwargs)
        return wrapper
    return outer

# 装饰加法函数
@say_hello('加法')
def add(x, y, z):
    res = x + y + z
    print(f'{x}和{y}和{z}相加的结果是：{res}')
    return res

# 装饰减法函数
@say_hello('减法')
def sub(x, y):
    res = x - y
    print(f'{x}和{y}相减的结果是：{res}')
    return res

# 测试代码
result1 = add(10, 20, 30)
result2 = sub(20, 10)
'''
你好，我要开始加法计算了
10和20和30相加的结果是：60
你好，我要开始减法计算了
20和10相减的结果是：10
'''
```

如果函数有多个装饰器，按照装饰器位置的**先后顺序生效**

```py
def decorate1(func):
    def enhance():
        print('pre-enhance1')
        func()
    return enhance

def decorate2(func):
    def enhance():
        print('pre-enhance2')
        func()
    return enhance

@decorate1
@decorate2
def func():
    print('functioning ..')

func()
# pre-enhance1
# pre-enhance2
# functioning ..
```

在推导式、生成器表达式当中，解释器会创建新的**局部命名空间**，表达式当中的变量查找命名空间的顺序和在函数局部命名空间是一样的，推导式的这个作用域地位完全**等同于函数作用域**，因而以下代码的输出预期里a是全局变量a=1，推导式内的作用域和a=1的作用域构成隐式的嵌套函数。

``` py
a = 1
class C:
    a = 2
    x = [a * i for i in range(0,5)]
    print(x) # [0, 1, 2, 3, 4]

if __name__ == '__main__':
    C()
```

```py
x = [a * i for i in range(5)]

#以上内容相当于
def _comp():
    result = []
    for i in range(5):
        result.append(a * i)
    return result
x = _comp()
```

**导包**如果导入完整模块有三种写法，但是建议使用第一种

![image-20260131023543229](assets/image-20260131023543229.png) 

模块的命名空间工作机制：当前文件启动**name内置属性**为main，被导入则为模块名

```py
# 模拟 Python 解释器如何设置 __name__
def python_interpreter_runs_file(filename):
    """
    模拟 Python 解释器执行模块的过程
    """
    # 1. 创建模块的命名空间
    module_namespace = {
        '__name__': '__main__',  # 直接运行时设置为 '__main__'
        '__file__': filename,
        '__doc__': None
    }
    
    # 2. 如果模块被导入
    if is_imported(filename):
        module_namespace['__name__'] = get_module_name(filename)
    
    # 3. 执行模块代码在这个命名空间中
    exec(module_code, module_namespace)
    
    return module_namespace

# 实际中，Python 解释器内部会处理这些细节
```

#### 基本语法的差异

is、==的区别

- ==类似于Java的equal，用于比较值是否相同
- is用于判断是否为同一个对象，比较内存地址是否相同

运算符不支持自增运算符

- 条件表达式可以一定程度替换三目运算符

  ```py
  text = '成年' if age >= 18 else '未成年·'
  ```

- `/` 是标准的、精确的除法，结果总是浮点数（float）。

- `//` 是向下取整除法，结果是一个整数（或浮点数），但总是朝着负无穷的方向取整。

- match-case关键字，作用和switch类似

  ```py
  match code:
      case 1: a = 1
      case 2: a = 2
  ```

- isinstance、issubclass运算符用于判断是否为另一个类的示例、子类
- 连续使用逻辑运算符可以考虑使用all()、any()函数替代

快速迭代函数range(start, stop[, step])

迭代语句的本质是**迭代器**，以下是自定义迭代器的实现

```py
class MyIterator:
    """自定义迭代器 - 展示迭代器协议"""

    def __init__(self, *args):
        self.data = args
        self.index = 0

    def __iter__(self):
        """迭代器本身也是可迭代的（返回自己）"""
        return self

    def __next__(self):
        """返回下一个元素，或抛出 StopIteration"""
        if self.index >= len(self.data):
            raise StopIteration
        value = self.data[self.index]
        self.index += 1
        return value


# 使用自定义迭代器
my_iter = MyIterator(1, 2, 3)
print("自定义迭代器:")
for item in my_iter:
    print(item)  # 1, 2, 3
```

**可迭代对象和迭代器**

可迭代对象是可以循环遍历的对象，内部实现了\_\_iter\_\_()方法，该方法返回一个迭代器对象，验证对象是否为可迭代对象直接调用\_\_iter\_\_()，检查返回的是否为迭代器即可

迭代器是一个负责逐个返回`可迭代对象的元素`的对象，主要实现了两个方法

- `__iter__()`：返回自身（`self`）。
- `__next__()`：返回下一个元素；如果没有元素了，抛出 `StopIteration` 异常。

调用\_\_next\_\_()时迭代器返回当前元素，下标前移。因为\_\_iter\_\_()返回值是自身，因此可迭代器也是可迭代对象。

#### 函数

函数在python中是名副其实的"一等公民"，可以像给类设置属性一样类似地给函数设置属性

```py
def my_func():
    my_func.counter = getattr(my_func, 'counter', 0) + 1
    print(f"函数已被调用 {my_func.counter} 次")

my_func()
my_func()
print(my_func.counter)  # 输出 2
```

允许多返回值

py的函数允许返回多个返回值，有多个时会自动将他们转为元组

嵌套函数

重要意义是让内部函数独立于全局命名空间

```python
def outer(prefix):
    # 这个工具函数只在outer内部有意义
    def inner(name):
        return f"{prefix}: {name}"
    
    # 使用内部函数
    result = inner("World")
    return result

print(outer("Hello"))  # 输出: Hello: World
# 在全局无法直接调用 inner("World")，因为它被“隐藏”了
```

可变参数

- 使用*形参名来接收任意数量的『位置参数』，多个位置参数最终会被打包成一个『元组』。
- 使用**形参名来接收任意数量的『关键字参数』，多个关键字参数最终会被打包成一个『字典』。

『可变位置参数』和『可变关键字参数』，可以同时使用，但必须要 先写 『可变位置参数 』

``` py
def ff(*args, **kwargs):
    print(args) #('kk', 'ww')
    print(kwargs) #{'k': 'k', 'w': 'w', 'hello': 'world'}
def f(*args, **kwargs):
    print(args) #('kk', 'ww')
    print(kwargs) #{'k': 'k', 'w': 'w', 'hello': 'world'}
    print('______')
    # 注意，使用参数的时候要手动解包，解包的方式就是*、**
    ff(*args,**kwargs)

f('kk', 'ww', k='k', w='w', hello='world')
```

py的函数灵活性非常高，允许在定义完成之后给函数添加函数的局部变量

```py
def welcome():
	print("你好")
# 动态添加属性
welcome.desc = "这是一个用于打招呼的函数"
welcome.version = 1.0
```

py 支持匿名函数/Lambda表达式，注意代码块只能是单个表达式

```py
f = lambda a,b:a + b
print(f(1, 2)) #3
```

函数式编程工具支持map、filter、sorted、reduce等，可以返回一个迭代器，用数据容器接收，不支持链式调用

```py 
# 函数式编程工具函数
from functools import reduce

l = [1, 2, 3]
l = list(map(lambda i: i + 1, l))
print(l)  # [2, 3, 4]
l = list(filter(lambda i: i > 2, l))
print(l)  # [3, 4]
d = [
    {
        'name': '朱建民',
        'age': 28
    },
    {
        'name': '郭大敏',
        'age': 27
    },
    {
        'name': '朱纪雨',
        'age': 26
    }
]
# sorted、max、min等聚合函数
d = sorted(d, key=lambda p: p['age'], reverse=False)
print(d)  # [{'name': '朱纪雨', 'age': 26}, {'name': '郭大敏', 'age': 27}, {'name': '朱建民', 'age': 28}]
sum_age = reduce(lambda a, b: a + b, list(map(lambda person: person.get('age'), d)), 0)
print(sum_age)  # 81
```

#### 数据容器

list

![image-20260127221126671](assets/image-20260127221126671.png) 

![image-20260127234229690](assets/image-20260127234229690.png) 

![image-20260127235030124](assets/image-20260127235030124.png)  

```py
# 基本增删改查
l = []
l.append(1)
l.append(2)
l.insert(0,0)
ll = [-1]
ll.extend(l)
print(l) #[0, 1, 2]
print(ll) #[-1, 0, 1, 2]
print(ll.pop(1)) #0
print(ll) #[-1, 1, 2]
ll.append(1)
ll.remove(1)
print(ll) #[-1, 2, 1]
del ll[2]
print(ll) #[-1, 2]
ll.clear()
print(ll) #[]

# 顺序、统计操作
print(l.index(1)) #1
l.reverse()
print(l) #[2, 1, 0]
print(l.count(1)) #1
l.sort()
print(l) #[0, 1, 2]

# 数据容器内置函数
# 倒序
l.reverse()
print(l) #[2, 1, 0]
l = sorted(l) #函数本身不会改变参数的容器顺序
print(l) #[0, 1, 2]
# 聚合函数
print(max(l)) #2
print(min(l)) #0
print(len(l)) #3
print(sum(l)) #3
```

in运算符用于判断是否为某个容器的元素

元组

有序集合，内容不可修改。当元组中只有一个元素时，末尾必须写上逗号，否则会被认为是单纯的数字用括号括起来调整优先级

``` py
l = tuple()
l[0] = 1
print(l) #TypeError: 'tuple' object does not support item assignment
```

元组和序列都支持**序列解包**

```py
if __name__ == '__main__':
    t = (1,2,3)
    a,b,c = t
    print(f'{a},{b},{c}') #1,2,3
```

字符串

- strip 按字符集合移除字符串首尾的字符
-  join 将可迭代对象用原有字符串连接为新字符串
- str和可迭代对象互相转换
  - str(可迭代对象)
  - list(s)
- 注意str不能直接通过+拼接基本数据类型

```py
# str常用方法
# strip删除字符串两端指定字符，如未指定则删除空格
s = '  abbba   '
s = s.strip()
print(s)

# 移除字符的原则是按字符串首尾两端往中间移动，直至下标位置字符不在strip入参字符数组中，停止移除
s = '34215 尚 12 硅 34 谷 4132'
s = s.strip('5432')
print(s)# 15 尚 12 硅 34 谷 41

# 字符串和列表、元组之间互相转换
s = 'abc'
t = tuple(s)
l = list(s)
print(t) #('a', 'b', 'c')
print(l) #['a', 'b', 'c']
ss = ''.join(l)
print(ss) #abc

# join
parts = ["2023", "10", "25"] 
print('-'.join(parts)) #2023-10-25
```

字符串的三种格式化方式

```py
# 字符串的三种格式化用法
print('hello %d %d'%(1,2)) #hello 1 2
print('hello {} {}'.format(1,2)) #hello 1 2
n1 = 1
n2 = 2
print(f'hello {n1} {n2}') #hello 1 2
```

集合是无序、元素唯一的容器，set的元素允许修改，frozenset不允许元素被修改，frozenset只能通过frozenset函数来创建，set可以用字面量定义、创建。

集合要求元素必须是不可变的

<img src="assets/image-20260128121928842.png" alt="image-20260128121928842" style="zoom:50%;" /> 

set支持add、pop、remove、discard、clear等增删操作(frozenset不支持)，修改直接先remove再add，查询有成员运算符in、not in

集合运算：difference支持找出a集合中不在b集合中的元素，difference_update直接删除掉a集合中不在b集合中的元素，union支持合并两个集合，issubset判断是否为b集合的子集，issuperset判断是否为b集合的超集，isdisjoint判断和b集合有无交集。|求并集，&求交集，-求差m，^求对称差

```py
s1 = {10, 20, 30, 40, 50}
s2 = {30, 40, 50, 60, 70}
print(s1.difference(s2)) #{10, 20}

s1 = {10, 20, 30, 40, 50}
s2 = {30, 40, 50, 60, 70}
s1.difference_update(s2)
print(s1) #{20, 10}

s1 = {10, 20, 30, 40, 50}
s2 = {30, 40, 50, 60, 70}
print(s1.union(s2)) #{70, 40, 10, 50, 20, 60, 30}
```

字典dict的新增或修改直接使用d[key] = value即可完成，删除用del关键字，查询用get方法（第二个参数是未查找到时给出的默认值）或d[key]，如要批量修改，可以使用update方法

```py
d1.update({'李四': 40, '王五': 67})
```

通过keys()、values()、items()方法可以获取字典的全部键、值、键值对，结果以封装的特殊形式返回

```py
d = {1:2,2:2}
print(d.keys())
print(d.values())
print(d.items())
print(type(d.keys()))
print(type(d.values()))
print(type(d.items()))
# dict_keys([1, 2])
# dict_values([2, 2])
# dict_items([(1, 2), (2, 2)])
# <class 'dict_keys'>
# <class 'dict_values'>
# <class 'dict_items'>
```

列表推导式

用[]包裹简单迭代语句得到列表的语法糖，基本上可以替代map函数

```py 
l = {1, 2, 3}
l = [i + 1 for i in l]
print(l)
```

zip函数可以快速组合两个容器

```py
a = [1, 2, 3]
b = [4, 5, 6]
zipped = zip(a, b)
for i in zipped:
    print(i)
    # (1, 4)
    # (2, 5)
    # (3, 6)
```

enumerate() 函数的效果是给可迭代对象**按下标组合**得到元组

```py
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
for i in enumerate(seasons):
    print(i)
# (0, 'Spring')
# (1, 'Summer')
# (2, 'Fall')
# (3, 'Winter')
```

至此，来总结下py支持的基础数据类型

- 数字类型：整数、浮点数、复数
- 序列类型：str、list、tuple、range
- 集合类型：set、frozenset
- 映射类型：dict
- 布尔类型：boolean
- 二进制类型：bytes、bytesarray
- None类型
- 其他内置类型

此外剩下的module、class、object、type、method、function等等都是对象，str也是特殊的对象

**打包和解包**

在这里补充下py的星号表达式，类似Java的解包

- 将列表、元组等解包为**位置参数**，将字典解包为关键字参数

  ```py
  nums = [1, 2, 3]
  print(*nums)       # 相当于 print(1, 2, 3)
  def show(a, b):
      print(a, b)
  d = {'a': 10, 'b': 20}
  show(**d)          # 输出 10 20
  ```

- 赋值语句中的星号，字面量解包，字面量解包合并

  ```py
  a, *b = [1, 2, 3, 4]   # a = 1, b = [2, 3, 4]
  *c, d = [1, 2, 3, 4]   # c = [1, 2, 3], d = 4
  e, *f, g = [1, 2, 3, 4] # e = 1, f = [2, 3], g = 4
  
  list1 = [1, 2]
  list2 = [3, 4]
  merged = [*list1, *list2]      # [1, 2, 3, 4]
  tup = (*list1, 5)              # (1, 2, 5)
  s = {*list1, *[2, 3]}          # {1, 2, 3}（集合去重）
  
  d1 = {'x': 1, 'y': 2}
  d2 = {'y': 3, 'z': 4}
  merged_dict = {**d1, **d2}     # {'x': 1, 'y': 3, 'z': 4}（后面的值会覆盖前面的）
  
  list1 = [1, 2]
  list2 = [3, 4]
  merged = [*list1, *list2]      # [1, 2, 3, 4]
  tup = (*list1, 5)              # (1, 2, 5)
  s = {*list1, *[2, 3]}          # {1, 2, 3}（集合去重）
  
  d1 = {'x': 1, 'y': 2}
  d2 = {'y': 3, 'z': 4}
  merged_dict = {**d1, **d2}     # {'x': 1, 'y': 3, 'z': 4}（后面的键会覆盖前面的）
  ```

- 迭代中的星号解包

  ```py
  data = [(1, 'a', 100), (2, 'b', 200)]
  for id, *rest in data:
      print(f"ID: {id}, Others: {rest}")
  # 输出：
  # ID: 1, Others: ['a', 100]
  # ID: 2, Others: ['b', 200]
  ```

- 在函数定义中，可以使用单独的 `*` 来强制其后面的参数必须以关键字参数形式传递。

  ```py
  def func(a, b, *, option=True):
      print(a, b, option)
  
  func(1, 2, option=False)   # 正确
  func(1, 2, False)          # 错误，因为 False 会被当作位置参数，但 option 只能以关键字形式传入
  ```

与解包（unpacking）相对的操作是打包（packing）。在 Python 中，星号（`*`）和双星号（`**`）既可以用于解包，也可以用于打包，它们是互为逆过程的两个概念

```py
def func(*args, **kwargs):
    print(args)   # 打包成元组
    print(kwargs) # 打包成字典
func(1, 2, 3, a=4, b=5)
# 输出: (1, 2, 3)
#       {'a': 4, 'b': 5}
```

#### 面向对象

py的构造方法是\_\_new\_\_，\_\_init\_\_方法是在创建对象后自动调用的"魔法方法”，用作实例的初始化，不能被重载

对象属性在进行声明init方法，一般传入self和属性

```py
def __init__(self, field):
    self.filed = field
```

**类方法**是定义在类中的方法，通过装饰器`@classmethod`来标识。它的第一个参数是`cls`（表示类本身），而不是实例对象。类方法可以访问类的属性，并且可以在没有实例的情况下被调用

```py
class Man:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    @classmethod
    def from_date(cls, name, age):
        return cls(name, age-1979)

    def getJson(self):
        return {'name':self.name, 'age':self.age}

man = Man.from_date('科比', 2026)
print(man.getJson())
```

py的**实例方法**第一个参数指向实例本身，在实例方法中访问实例属性必须通过来self.field来实现

注意py的**静态方法**（用@staticmethod修饰）既没有self入参也没有cls入参，因此没办法访问实例属性也没办法访问类属性，常用于与当前类有关的工具方法

类的继承

子类的init方法形参列表应当大于父类init方法，super()方法可以帮助快速调用父类方法，当然这个父类可以有多个，从给定的**MRO顺序**（方法解析顺序）获取目标方法。访问属性时本质上就是查找对象的**\_\_dict__属性**，调用方法时要先查找类的mro列表。py支持多继承，如果存在多继承MRO由**C3算法**计算，由集成拓扑得到线性列表，不过注意集成自object的类只能有一个。

```python
class Animal:
    branch = 1
    def __init__(self, branch:int):
        self.branch = branch
    def move(self):
        print('I am moving')
        pass

class Dog(Animal):
    def __init__(self, branch:int):
        print(super()) #super() 让你能不必显式地指定父类名字，就能调用父类的方法
        super().__init__(branch) #<super: <class 'Dog'>, <Dog object>>

    def move(self):
        self.run()
    def run(self):
        print('I am running')

dog = Dog(4) #<super: <class 'Dog'>, <Dog object>>
dog.move() #I am running
print(dog.branch) #4
print(dog.__dict__) #{'branch': 4}
print(Dog.mro()) #[<class '__main__.Dog'>, <class '__main__.Animal'>, <class 'object'>]
```

```py
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.__mro__)
# 输出：(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
```

**属性的访问权限**

<img src="assets/image-20260129045620460.png" alt="image-20260129045620460" style="zoom:67%;" /> 

**属性的封装**

py的对象封装策略也是将属性私有化，并提供必要的getter、setter，具体实现依赖给和属性同名的方法添加装饰器来实现

- @Property使得属性允许访问
- @methodname.setter使得属性允许修改
- @methodname.deleter可以修改del filed的执行逻辑

以上装饰器在使用时本质上是在执行被装饰的方法

```py
class Rect:
    def __init__(self, area):
        self.__area = area

    @property
    def area(self):
        return self.__area

    @area.setter
    def area(self, value):
        self.__area = value

    @area.deleter
    def area(self):
        self.__area = 0


rect = Rect(30)
print(rect.area)  # 30
rect.area = 10
print(rect.area)  # 10
del rect.area
print(rect.area)  # 0
```

魔法函数

“魔法函数”（Magic Methods）或“特殊方法”（Special Methods）是指以**双下划线开头和结尾**的函数（如 `__init__`）,由Python解释器在特定时机（如调用了对应的无下划线的内置函数）自动调用

<img src="assets/image-20260129051334832.png" alt="image-20260129051334832" style="zoom:50%;" /> 

```py
# 魔法方法
class C:
    def __init__(self, string):
        self.string = string
    def __len__(self):
        print('func inner statement:'+""+str(len(self.string)))
        return len(self.string)

c = C('hello')
print(len(c))
# func inner statement:5
# 5
```

**py多态**

py基于"鸭子类型"的设计理念，拒绝了Java的严格的实现接口的多态实现方式，而是选择关注行为本身，只要对象提供相应的方法，就可以调用。（可以用hasattr(obj, attr_name_str)来验证对象有无指定方法）

```py
class Duck:
    def quack(self):
        print("嘎嘎")

class Person:
    def quack(self):
        print("模仿嘎嘎")

def make_it_quack(animal):
    animal.quack()  # 不关心类型，只关心有没有 quack 方法

make_it_quack(Duck())    # 正常运行
make_it_quack(Person())  # 也正常运行
```

现在总结下多态的多态实现方式

- 继承+方法重写（本质上仍然是鸭子类型，不过有了类型判断）

- 鸭子类型

- 运算符重载

  ```py
  class Vector:
      def __init__(self, x, y):
          self.x = x
          self.y = y
      def __add__(self, other):
          return Vector(self.x + other.x, self.y + other.y)
  
  v1 = Vector(1, 2)
  v2 = Vector(3, 4)
  v3 = v1 + v2      # 多态：+ 被重载为向量加法
  print(v3.x, v3.y) # 输出 4 6
  ```

**抽象类**

py的抽象类中允许`具体方法`存在，通过继承**ABC类或将元类设置为ABCMeta类**说明当前类是抽象类，借助@abstractmethod来说明方法是`抽象方法`。抽象方法必须在非抽象子类中进行实现，否则无法实例化。

py的抽象方法、具体方法都可以有方法体和具体实现，区分点只有是否被装饰器@abstractmethod修饰，以及方法实现内容的抽象/具体程度

```py
class AbstractBase(ABC):
    @abstractmethod
    def method(self):
        pass
class ConcretSub(AbstractBase):
    def method(self):
        print('i am implement')

ConcretSub().method()
```

**拷贝机制**

 copy.copy()提供浅拷贝

![image-20260130015821760](assets/image-20260130015821760.png) 

元素是不可变对象可以完全拷贝，元素是对象，则拷贝的是引用

如果要实现深拷贝需要使用copy.deepcopy函数

函数的传参本质上是值传递，在函数创建对象的副本，如果是不可变对象，本质上是拷贝了其引用，指向的对象空间仍然不变

#### str的intern机制

str是不可变对象，每次创建后维护到内存中的**字符串池**中（**interned字典**），内容相同的字符串变量直接引用池中的字符串对象，而非重新分配内存

驻留池 `interned` 是一个全局字典，键和值都是字符串对象（键是 PyUnicodeObject 的哈希值，值是对象本身），确保每个字符串只保留一份。当字符串被 intern 时，其引用计数会增加（因为池持有引用），所以不会在正常引用计数归零时被回收，直到解释器关闭或者字符串析构函数判定应该释放时才释放。

除了str变量，其他如函数名、类名、属性名等，以及长度为1的短字符串、空串，也会在编译时自动intern（驻存），如果希望手动intern，可以调用sys.intern(your_str)显式地完成字符串驻存。运行时动态生成的字符串不会被intern，比如用户控制台输入，如要查看是否被intern可以使用sys.__is_interned()来进行判断。

interned字典是全局可访问的，并一直存在内存中直至解释器退出。

#### 函数装饰器、类装饰器

**这部分内容一定要结合代码实践直接刷面试题，贼吉尔重要贼吉尔难**

装饰器本质上是一个函数或类，它接受一个函数或类作为输入，并返回一个新的函数或类。装饰器的主要作用是在不修改原函数或类的代码的前提下，为其添加额外的功能，比如日志记录、性能测试、权限验证等。

类装饰器本质上是一个可调用对象（通常是**函数**或实现了 `__call__` 方法的类），它接收一个类作为参数，并返回一个**修改后的类**（可以是原类，也可以是修改属性、方法后全新的类）。装饰器的执行时机是在类定义完成之后立即执行，等价于：

```py
class MyClass:
    pass

MyClass = decorator(MyClass)
```

具体的使用有以下几类：

- 函数形式装饰

  ```py
  def add_method(cls):
      def new_method(self):
          print('new_method functioning ..')
      cls.new_method = new_method
      cls.new_field = 'new_field_value'
      return cls
  
  @add_method #等价于cls = add_method(cls)
  class cls:
      pass
  
  cls().new_method() #new_method functioning ..
  print(cls().new_field) #new_field_value
  ```

- 带参数的函数形式类装饰器

  ```py
  new_fields = ['f1', 'f2']
  new_field_values = ['v1', 'v2']
  functions = [lambda self, a, b: a + b, lambda self, a, b: a - b]
  
  # 外层函数相当于装饰器工厂，decorate才是实际执行装饰的函数
  def add_method(new_fields, new_field_values, functions):
      def decorate(cls):
          for i in range(0, len(new_fields)):
              setattr(cls, new_fields[i], new_field_values[i])
          for i in range(0, len(functions)):
              setattr(cls, functions[i].__name__ + '_' + str(i), functions[i])
          return cls
  
      return decorate
  
  @add_method(new_fields, new_field_values, functions)  # 等价于cls = add_method('new_field','new_field_value')
  class cls:
      pass
  
  obj = cls()
  for field in new_fields:
      print(f"{field}: {getattr(obj, field)}")
      # f1: v1
      # f2: v2
  for i in range(0, len(functions)):
      print(getattr(obj, functions[i].__name__ + '_' + str(i))(1, 1))
      # 2
      # 0
  ```

- 类作为装饰器

  > \_\_**call**\_\_允许将一个类的实例像函数一样被调用，这样的话就可以在不改变被装饰函数的使用方式下用类的形式去增强函数
  >
  > ```py
  > class Counter:
  >     def __init__(self):
  >         self.count = 0
  >     def __call__(self):
  >         self.count += 1
  >         print(f"已调用 {self.count} 次")
  > 
  > counter = Counter()
  > counter()  # 输出：已调用 1 次
  > counter()  # 输出：已调用 2 次
  > ```

  - 类用作装饰器的应用场景
    - 维护跨多次函数调用的状态。原本需要借助外部变量（自由变量、全局变量）
    
    - 类装饰器可以在返回的可调用对象上附加其他方法，供外部使用。函数装饰器虽然也能返回一个带有属性的函数（通过闭包），但类的方式更清晰。
    
    - 当装饰器本身需要接收参数时，类装饰器通常比嵌套两层函数的装饰器更直观。实现方式是在 `__init__` 中接收装饰器参数，在 `__call__` 中接收被装饰函数。
    
    - 其他，碰到再补充
    
      ```py
      class cls:
          def __init__(self, func):
              self.func = func
              self.flag = 0 #为func设置可维护的状态
      
          def __call__(self, *args, **kwargs):
              self.flag += 1
              return self.func(*args, **kwargs)
      
      
      @cls # func = cls(func)
      def func():
          print("Hello!")
      
      
      func()
      func()
      print(func.flag) #2
      ```
    
      ```py
      class cls:
          def __init__(self, func):
              self.func = func
          def __call__(self, *args, **kwargs):
              func()
          def other_func(self):
              print('other func is functioning.. ')
      
      @cls #等价于func = cls(func)
      def func():
          print('functioning ..')
      
      func() #一次call调用
      func.other_func() #支持其他方法
      ```
    
      



用类装饰类的话还能增强一个类，使得原有类拥有更多的成员

```py
def add_method(cls):
    """添加一个方法到类中"""

    def extra_method(self):
        return f"这是添加的方法，类名：{cls.__name__}"

    cls.extra_method = extra_method
    return cls

# 等同于MyClass=add_method(MyClass)
@add_method
class MyClass:
    def __init__(self, value):
        self.value = value
    def original_method(self):
        return f"原始方法，值：{self.value}"


# 使用
obj = MyClass(10)
print(obj.original_method())  # 输出：原始方法，值：10
print(obj.extra_method())  # 输出：这是添加的方法，类名：MyClass
```

**\_\_str\_\_和_\_repr__**

在 Python 中，`__str__` 和 `__repr__` 是两个用于定义对象字符串表示的魔法方）。它们的主要区别在于目标受众和使用场景，`__str__` 面向用户用于显示字面量，\_\_repr\_\_用于开发、调试查看str对象真实值，比如repr('\n')打印结果就是'\n'，连引号都带完整的。

```py
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __repr__(self):
        return f"Person('{self.name}', {self.age})"

    def __str__(self):
        return f"{self.name} ({self.age} years old)"

p = Person("Alice", 30)

print(repr(p))   # Person('Alice', 30)
print(str(p))    # Alice (30 years old)
print(p)         # 同上（调用 __str__）
# 在交互式环境中：Person('Alice', 30) （调用 __repr__）
p                
```

#### 类型注解

经常有必要给代码添加注解，虽然不会限制数据类型，但是可以极大的提高可读性

```py
from typing import Union

# 变量注解
a: int
a = 1
b: str = ''


# 函数参数注解
def func(n: int) -> (int,int):
    return n, n+1
# def add(*args: int) -> int:
#     return sum(args)
def show_info(**kwargs: str | int):
    print(kwargs)
# 获取函数的注解信息
print(add.__annotations__)

# 容器类型的注解
letter: list[str] = ['a', 'b', 'c']

# Union支持多个类型注解，作用和|类似
hobby: list[Union[str, int]] = ['抽烟', '喝酒', '烫头']
citys: set[str | float | bool] = {'北京', '上海', '深圳'}
# 元组可以给多个具体的元素进行注解
scores: tuple[int, int, int] = (60, 70, 80)
scores: tuple[int, ...] = (60, 70, 80, 90, 100)
scores: tuple[int | str, ...] = (60, '70', 80, '90', 100)
# 字典类型的key、word类型之间用，隔开
persons: dict[str | int, int] = {'张三': 18, '李四': 19, '王五': 20}
```

#### 异常处理机制

py异常体系

``` 
BaseException
 ├── BaseExceptionGroup
 ├── GeneratorExit
 ├── KeyboardInterrupt
 ├── SystemExit
 └── Exception
      ├── ArithmeticError
      │    ├── FloatingPointError
      │    ├── OverflowError
      │    └── ZeroDivisionError
      ├── AssertionError
      ├── AttributeError
      ├── BufferError
      ├── EOFError
      ├── ImportError
      │    └── ModuleNotFoundError
      ├── LookupError
      │    ├── IndexError
      │    └── KeyError
      ├── MemoryError
      ├── NameError
      │    └── UnboundLocalError
      ├── OSError
      │    ├── BlockingIOError
      │    ├── ChildProcessError
      │    ├── ConnectionError
      │    │    ├── BrokenPipeError
      │    │    ├── ConnectionAbortedError
      │    │    ├── ConnectionRefusedError
      │    │    └── ConnectionResetError
      │    ├── FileExistsError
      │    ├── FileNotFoundError
      │    ├── InterruptedError
      │    ├── IsADirectoryError
      │    ├── NotADirectoryError
      │    ├── PermissionError
      │    ├── ProcessLookupError
      │    └── TimeoutError
      ├── ReferenceError
      ├── RuntimeError
      │    ├── NotImplementedError
      │    └── RecursionError
      ├── StopAsyncIteration
      ├── StopIteration
      ├── SyntaxError
      │    └── IndentationError
      │         └── TabError
      ├── SystemError
      ├── TypeError
      ├── ValueError
      │    └── UnicodeError
      │         ├── UnicodeDecodeError
      │         ├── UnicodeEncodeError
      │         └── UnicodeTranslateError
      └── Warning
           ├── DeprecationWarning
           ├── FutureWarning
           ├── UserWarning
           └── ...
```

try、except、finally机制

``` py
try:
    a = 1/0
# 不指明具体异常类型亦可
except:
    print('')

    
try:
    a = 1/0
# 指明具体异常类型
except BaseException:
    print('')


try:
    a = 1/0
# 捕获多个异常
except ValueError:
    print('程序异常：您输入的必须是数字！')
except ZeroDivisionError:
    print('程序异常：0 不能作为除数！')

try:
    a = 1/0
# 捕获多个异常
except (ValueError, ZeroDivisionError) as e:
    match e:
        # 这里的case有个trick，ClassName()在case后面执行并不会创建新的实例
        case ValueError():
            print('程序异常：您输入的必须是数字！')
        case ZeroDivisionError():
            print('程序异常：0 不能作为除数！')


try:
    a = 1/1
except:
    pass
# 不论try语句是否有异常都将执行finally语句
finally:
    print('finally statement ..')
```

如果在try或except中出现了return语句，解释器会先记录返回值，并执行finally语句，最后返回。如果finally语句有return，会用新的返回值覆盖旧的返回值

```py
def func():
    try:
        print(1)
        return 1
        print(2)
    except:
        print('except')
    finally:
        print('finally')
        return 3

if __name__ == '__main__':
    print(func())
    # 1
    # finally
    # 3
```

打印异常情况

- e.args

- e.value

- e.\_\_str__

- 交互式环境下e

  ![image-20260303112131125](assets/image-20260303112131125.png) 



 `BaseException` 定义了默认的 `__str__` 方法。其行为大致如下：

- 如果异常实例有非空的 `args` 元组（即构造时传入的参数），
  - 当 `len(args) == 1` 时，`__str__` 返回 `str(args[0])`；
  - 当 `len(args) > 1` 时，`__str__` 返回 `repr(args)`（即所有参数的表示）。
- 如果 `args` 为空（即没有提供任何参数），`__str__` 返回空字符串 `''`。

- 打印异常对象e时返回的是e.\_\_str__()

`BaseException` 的 `__repr__` 大致逻辑是：

- 如果异常实例的 `args` 属性（即构造时传入的参数元组）为空，则返回 `f"{self.__class__.__name__}()"`。
- 如果 `args` 有一个参数，则返回 `f"{self.__class__.__name__}({repr(args[0])})"`。
- 如果 `args` 有多个参数，则返回 `f"{self.__class__.__name__}{repr(args)}"`。



所有的py异常都是运行时异常，即都能通过编译。如果未try-except，异常会自动上抛，如果要主动抛出异常，使用raise关键字

```py
>>> raise
Traceback (most recent call last):
  File "<pyshell#1>", line 1, in <module>
    raise
RuntimeError: No active exception to reraise
>>> raise ZeroDivisionError
Traceback (most recent call last):
  File "<pyshell#0>", line 1, in <module>
    raise ZeroDivisionError
ZeroDivisionError
>>> raise ZeroDivisionError("除数不能为零")
Traceback (most recent call last):
  File "<pyshell#2>", line 1, in <module>
    raise ZeroDivisionError("除数不能为零")
ZeroDivisionError: 除数不能为零
```

#### 生成器

1. 生成器函数：函数体中如果出现了 yield 关键字，那该函数是『生成器函数』
2. 生成器对象：调用『生成器函数』时，其函数体不会立刻执行，而是返回一个『生成器对象』
  1. 调用『生成器对象』的__next__方法，会让『生成器函数』中的代码开始执行
  2. 当『生成器函数』中的代码开始执行后，遇到 yield 会“暂停”，并会记录“暂停”的位置
  3. 后续调用__next__方法时，都会从上一次“暂停”的位置，继续运行，直到再次遇到yield

  4. 遇到 return 会抛出 StopIteration 异常，并将 return 后面的表达式，作为异常信息。
  5. yield 后面所写的表达式，会作为本次__next__方法的返回值。

```py
# 生成器函数
def demo():
    print("a ..")
    yield 'a'
    print("b ..")
    yield 'b'
    return 250
# 生成器对象
obj = demo()
# 执行next
print(next(obj))
# a ..
# a
print(next(obj))
# b ..
# b
try:
    next(obj)
except StopIteration as stop_iteration:
    print(stop_iteration) #250

# 本质上生成器函数是通过yield关键字生成了一个迭代器
print(hasattr(obj, '__iter__')) #True
```

yield from 能把一个『可迭代对象』里的东西依次 yield 出去。(替代：for + yield)

```py
def demo():
    nums = [10, 20, 30, 40]
    yield from nums
d = demo()
r1 = next(d)
print(r1)
r2 = next(d)
print(r2)
r3 = next(d)
print(r3)
r4 = next(d)
print(r4)
# 10
# 20
# 30
# 40
```

生成器对象的send(data)方法作用和\_\_next__类似，也是执行到下一处yield，不过还补充了入参值data，yield语句可以接收结果，每次yield接收的结果就是send方法是入参值。需要注意的是，第一次启动生成器，必须使用next启动，或者用send传None

```py
def demo():
    msg = yield '10'
    print(msg)
    yield '20'
d = demo()
r1 = d.send(None)
print(r1)
r2 = d.send('200')
print(r2)
# 10
# 200
# 20
```

用生成器函数实现迭代器

```py
class Person:
    def __init__(self, name, age, gender, address):
        self.name = name
        self.age = age
        self.gender = gender
        self.address = address
        self.__attr = [name, age, gender, address]
    def __iter__(self):
        yield from self.__attr
p1 = Person('张三', 18, '男', '北京昌平')

print(list(p1)) #['张三', 18, '男', '北京昌平']
for attr in p1:
    print(attr)
# 张三
# 18
# 男
# 北京昌平
```

生成器表达式本质上是快速创建生成器对象的方式

```py
result = (n * 2 for n in nums)
```

#### IO

文件的open操作是和close配对使用的，为减少异常处理语句的大量使用，引入with语句，针对对象的配对方法a，调用前使用with关键字，:后面的代码块被执行完成后，对象会自动调用与a对应配对的方法b

```py
with open('/path/to/file', 'r') as f:
    print(f.read())
```

等同于

``` py
try:
    f = open('/path/', 'r')
    print(f.read())
finally:
    if f:
        f.close()
```

py的open函数mode中的修改参数值参考，如要同时读写需要使用+参数，具体组合如下

![image-20260131064856171](assets/image-20260131064856171.png)

递归扫描路径下的文件和目录

```py
result = os.walk('D:/demo')
for item in result:
	print(item)
```

#### 项目构建与包管理

**pip**

安装pip

```bash
sudo apt install python3-pip
```

设置镜像源

```sh
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

**poetry**

安装poetry

``` sh
# powershell 运行
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -
```

设置镜像源

```sh
# 放在toml最上面
[[tool.poetry.source]]
name = "mirrors"
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
priority = "primary"
```

创建poetry项目

```sh
poetry new proj-name -n
```

添加依赖

```sh
poetry add modulename
```

移除依赖

```sh
poetry remove modulename --lock 
```

查看项目依赖

```sh
poetry show
poetry show --tree
```

#### 虚拟环境

创建虚拟环境

```sh
 sudo apt install python3.12-venv
 python3 -m venv myenv
```

#### 多进程

py的os、`multiprocessing`模块负责管理进程，该模块的一些基本使用如下

获取进程id

```py
# 获取当前进程的id
print(os.getpid())
# 获取父进程的id，在pycharm中父进程是pycharm
print(os.getppid())
```

使用Process类**创建多进程**，可以指定以下参数

- group： 默认值为 None（应当始终为 None）。在 Python 的 multiprocessing 模块中，`group` 参数是一个为了保持API一致性而存在的占位符，在实际使用中没有任何作用，必须始终设置为 None
- target：子进程要执行的可调用对象，默认值为 None。
- name： 进程名称，默认为 None ，如果设置为 None，Python 会自动分配名字。
- args： 给 target 传的位置参数（元组）
- kwargs：给 target 传的关键字参数（字典）。
- daemon：标记进程是否为`守护进程`，取值为布尔值（默认为 None，表示从创建方继承）。守护进程，即生命周期依赖于父进程的进程，不能再创建子进程，常常用于设置工作在后台的进程

```py
from multiprocessing import Process

def func(arg: tuple):
    print(f'I\'m process {arg}')

if __name__ == '__main__':
    p1 = Process(target=func, arg=(1,))
    p2 = Process(target=func, arg=(2,))
    p1.start()
    p2.start()
    # I'm process 1
    # I'm process 2
```

```py
print(current_process())
print(current_process().name)
print(current_process()._args)
print(current_process()._kwargs)
print(current_process()._target)
print(current_process().daemon)
# <_MainThread(MainThread, started 8740)>
# MainThread
# ()
# {}
# False
# None

# AttributeError: '_MainThread' object has no attribute 'group'
# print(current_process().group)
```

进程对象**常用方法**

- join()方法可以阻塞当前进程，直至调用join的进程执行完毕，或者指定的时间到期（可以在join方法参数指定时间，单位为s）
- is_alive()方法可以判断进程是否存活
- terminate()可以强制终止进程，且后续出现finally语句不会执行

**进程通信**的重要手段：`多进程队列`、`管道`

多进程队列支持put、get方法用于入队、出队，Queue是阻塞队列，存取元素碰到qsize()为满/空时当前进程被阻塞，直至不再为满/空。如果想要执行非阻塞读取使用put_nowait(item)、get_nowait()

```py
import time
from multiprocessing import Queue, Process

q = Queue(maxsize=1)

def consumer(q: Queue):
    while(True):
        num = q.get()
        print(f'消费: {num}，现在队列长度为{q.qsize()}')
        time.sleep(1)

def productor(q: Queue):
    num = 0
    while(True):
        num += 1
        res = q.put(num)
        print(f'生产：{num}，现在队列长度为{q.qsize()}')
        time.sleep(5)

if __name__ == '__main__':
    p1 = Process(target=consumer, args=(q,))
    p2 = Process(target=productor, args=(q,))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
    # 生产：1，现在队列长度为1
    # 消费: 1，现在队列长度为0
    # 生产：2，现在队列长度为1
    # 消费: 2，现在队列长度为0
    # 生产：3，现在队列长度为1
    # 消费: 3，现在队列长度为0
    # 生产：4，现在队列长度为1
    # 消费: 4，现在队列长度为0
    # ...
```

管道**multiprocessing.Pipe**是进程间通信的重要手段，本质上是内核管理的一个**先进先出**的缓冲区。multiprocessing.Pipe()函数用于创建一个管道，并返回两个连接对象，分别代表管道的两端。默认情况下，管道是双向的，可以同时进行读写操作

```py
from multiprocessing import Pipe
# 创建全双工管道
parent_conn, child_conn = Pipe()
# 创建半双工管道
recv_conn, send_conn = Pipe(duplex=False)
```

send方法、recv方法用于发送、接收消息

```py
import random
import time
from multiprocessing import Queue, Process, Pipe
from multiprocessing.dummy import current_process

conn1, conn2 = Pipe()

def consumer(p: Pipe):
    while (True):
        coordinate = p.recv()

        print(f'现在当前进程{current_process.__name__}收到坐标是({coordinate[0]},{coordinate[1]})')
        p.send(f'进程{current_process.__name__}已收到坐标！')
        time.sleep(1)

def productor(p: Pipe):
    while (True):
        coordinate = (random.randint(0,10),random.randint(0,10))
        p.send(coordinate)
        print(p.recv())
        time.sleep(3)

if __name__ == '__main__':
    p1 = Process(target=consumer, args=(conn1,))
    p2 = Process(target=productor, args=(conn2,))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
    # 现在当前进程current_thread收到坐标是(7,10)
    # 进程current_thread已收到坐标！
```

双向管道一定要注意避免死锁，一旦陷入互相等待消息，管道就会进入`死锁`状态

**避免死锁**状态的方法有

- 请求-响应模式：一方作为主动方，先发送请求；另一方作为被动方，先接收再回复

  ```py
  # 主动方（如生产者）
  while True:
      data = generate()
      conn.send(data)       # 先发送
      ack = conn.recv()     # 再接收确认
  
  # 被动方（如消费者）
  while True:
      data = conn.recv()    # 先接收
      process(data)
      conn.send("ACK")      # 再发送确认
  ```

- 使用半双工管道

- 读消息前非阻塞地探听管道是否有消息

  ```py
  if conn.poll(timeout=2.0):       # 等待最多2秒
      msg = conn.recv()
  else:
      # 超时处理，例如重试或记录日志
      print("No data received, continue...")
  ```

封装为对象的进程创建方式：**继承Process类**，这种方式创建进程可以深度绑定任务和进程，并拥有维护状态、快速调用其他行为的能力，是更优雅、更健壮的设计模式

```py
class BaseWorker(Process):
    def __init__(self, config):
        super().__init__()
        self.config = config

    def run(self):
        self.setup()
        self.loop()
        self.cleanup()

    def setup(self): pass
    def loop(self): pass
    def cleanup(self): pass

class SpecificWorker(BaseWorker):
    def loop(self):
        # 具体实现
        pass
```

**进程池**创建进程是适用最宽泛的合理实践方式，这样做很好地避免了频繁创建/销毁进程，进程池会预先创建好若干数量的存活进程，当要执行任务时直接拿来使用即可。这里注意，map返回的是一个生成器对象，submit和map都不是阻塞方法，调用result或迭代时会阻塞。

```py 
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor() as executor:
    future = executor.submit(pow, 2, 10)
    print(future.result())          # 1024

    results = executor.map(pow, [2, 3, 4], [10, 10, 10])
    print(list(results))             # [1024, 59049, 1048576]
```

`concurrent.futures.ProcessPoolExecutor` 的 `shutdown` 方法用于**清理进程池并释放资源**。当你不再需要执行器时，应调用此方法来停止接受新任务、等待或取消未完成的任务，并终止工作进程。

```py
shutdown(wait=True, *, cancel_futures=False)
```

wait 控制是否在返回前等待所有待处理的任务完成，cancel_futures 控制是否取消尚未开始运行的任务

`as_completed`用于在一组 Future 对象中**按完成顺序逐个获取已经完成的任务结果**，返回一个迭代器，当任意一个 Future 完成时，迭代器就会立即产生该 Future，这样你可以第一时间处理已完成的任务，而不必按照任务提交的顺序等待

```py
as_completed(futures, timeout=None)
```

```py
executor = ProcessPoolExecutor(3)
futures:list[Future] = [executor.submit(pow, i,2) for i in range(0, 3)]
rst = [future.result() for future in as_completed(futures)]
print(rst) #结果无固定顺序
```

可以使用add_done_callback为任务添加完成时的回调函数

```py
result_list = []
def done_func(futrue):
	result_list.append(futrue.result())
for index in range(1, 8):
	f = executor.submit(work, index)
f.add_done_callback(done_func)
executor.shutdown(wait=True)
```

#### 多线程

创建线程的方式

- Thread(target=func,args=(arg0, arg1, ...))
- 继承Thread类，和继承Process类一样，需要重写run方法
- 线程池ThreadPoolExecutor获得Future对象，Future的用法和前面进程池是一样的。同样支持as_completed、add_done_callback以及map批量提交任务

由于GIL的存在，CPython 解释器中的多线程模型，**本质上是并发**，而不是并行

py多线程的意义在于提高`IO密集型任务`的吞吐量，而CPU密集型的任务应当交给真正具有解释器并行执行能力的多进程来完成

```py
# CPU密集型基本没啥卵用
# cpthread_demo.py
import time
from threading import Thread

def cpu_heavy(n):
    """模拟 CPU 密集型任务：大量数值计算"""
    count = 0
    for i in range(n):
        count += i ** 2
    return count

def run_in_threads(task, n_threads, task_args):
    """启动多个线程执行同一个任务"""
    threads = []
    start = time.perf_counter()
    for _ in range(n_threads):
        t = Thread(target=task, args=task_args)
        t.start()
        threads.append(t)
    for t in threads:
        t.join()
    end = time.perf_counter()
    print(f"线程数: {n_threads}, 耗时: {end-start:.2f}秒")

if __name__ == "__main__":
    # 单线程
    run_in_threads(cpu_heavy, 1, (10**7,))
    # 双线程
    run_in_threads(cpu_heavy, 2, (10**7,))
    #线程数: 1, 耗时: 0.85秒
	#线程数: 2, 耗时: 1.70秒
    


#IO密集型大显身手，IO或sleep挂起时，线程会主动释放GIL让其他线程运行，这样很快大家都可以进入处理IO任务的状态，从而提高整体的执行效率
import time
from threading import Thread

def io_heavy(sec):
    """模拟 I/O 密集型任务：比如下载图片"""
    time.sleep(sec)          # 模拟 I/O 等待
    # 这里可以做少量计算，但主要是等待

def run_io_tasks(n_tasks, task_time):
    start = time.perf_counter()
    threads = []
    for _ in range(n_tasks):
        t = Thread(target=io_heavy, args=(task_time,))
        t.start()
        threads.append(t)
    for t in threads:
        t.join()
    end = time.perf_counter()
    print(f"总耗时: {end-start:.2f}秒")

if __name__ == "__main__":
    run_io_tasks(4, 2)   # 4个任务，每个“睡”2秒
    #线程数: 1, 耗时: 0.03秒
	#线程数: 2, 耗时: 0.03秒
	#线程数: 4, 耗时: 0.03秒
```

#### 锁

在多进程中声明临界区代码可以使用`Lock`进程锁，具体用法

创建锁

```py
lock = Lock()
```

声明临界区

```py
lock.acquire() #上锁
# 临界区代码...
lock.release() #释放锁
```

或者

```py
with lock:
    #临界区代码...
```

进程锁的验证案例必须体现在`多进程共享资源`上，控制台是典型的共享资源，在无锁无队列来保证同步的情况下会出现竞用的情形，从而出现同步问题——打印的顺序错乱不符合预期

![image-20260305153251102](assets/image-20260305153251102.png)

给使用共享资源的不同代码加上相同锁之后，加锁的代码要竞争到锁才能执行共享资源操作，而锁只有一个，因此只有一个进程可以在同一时刻执行

![image-20260305153655227](assets/image-20260305153655227.png)

**可重入锁**`RLock`

Lock不支持重入多次加锁，如果有`链式调用`或`递归调用`就可能会因为需要重新获得锁从而出现一直阻塞的情形，可重入锁可以很好地解决这个问题。

RLock在内部维护了`线程标识Thread ID`和`递归层级`每次acquire()时层级都会+1，release()时层级-1，其他线程尝试获取Rlock时会一直阻塞，使用时注意获取锁和释放锁操作的配对，嵌套层级结构尽可能清晰

**进程共享变量Value、Array**

`multiprocessing.Value` 是 Python 多进程模块提供的一个**共享内存对象**，用于在多个进程之间安全地共享一个简单的值（如整数、浮点数等）。它底层基于操作系统的共享内存机制，效率比通过管道或队列传递数据更高，特别适合需要频繁读写的小型数据。

创建Value对象

```py
from multiprocessing import Value

# 方式一：使用类型码字符串
counter = Value('i', 0)       # 'i' 表示有符号整数，初始值 0

# 方式二：使用 ctypes 类型
from ctypes import c_double
pi = Value(c_double, 3.14159)  # 双精度浮点数
```

使用Value对象

```py
counter.value += 1
print(counter.value)
```

常见的类型码

| 类型码 | C 类型           | Python 类型     |
| :----- | :--------------- | :-------------- |
| `'c'`  | `char`           | 字节串（长度1） |
| `'b'`  | `signed char`    | 整数            |
| `'B'`  | `unsigned char`  | 整数            |
| `'h'`  | `signed short`   | 整数            |
| `'H'`  | `unsigned short` | 整数            |
| `'i'`  | `signed int`     | 整数            |
| `'I'`  | `unsigned int`   | 整数            |
| `'l'`  | `signed long`    | 整数            |
| `'L'`  | `unsigned long`  | 整数            |
| `'f'`  | `float`          | 浮点数          |
| `'d'`  | `double`         | 浮点数          |

Value对象默认自带一个绑定的Rlock，用于保证对对象的vlue的读写是原子的，所有的针对c type的原子操作都能做到同步，但是像i += 1操作本身就不是原子操作，因此也不能光靠绑定的锁来实现共享对象的同步

Value对象绑定的这个锁可以显示的修改

```py
counter_no_lock = Value('i', 0, lock=lock)   
```

验证多进程共享Value对象时的同步问题

```py
import time
from multiprocessing import Process, Lock, Value

def add_without_lock(counter:Value, loop_times:int):
    for i in range(0, loop_times):
        counter.value += 1
def add_with_lock(counter:Value, loop_times:int, lock:Lock):
    for i in range(0, loop_times):
        with lock:
            counter.value += 1

if __name__ == '__main__':

    count_processes = 5 # 进程数
    increase_times = 10000 # 每个进程让counter加1的次数
    counter = Value('i', 0) # counter是0的Value对象

    # 不加锁测试用Value对象共享变量执行非原子操作是否会出现同步问题
    # 准备进程
    processes = [Process(target=add_without_lock, args=(counter, increase_times)) for i in range(0, count_processes)]
    # 执行进程
    for p in processes:
        p.start()
    for p in processes:
        p.join()
    print('————未给increase操作上锁————')
    print(f'期望的counter最终数值{0+count_processes*increase_times}')
    print(f'实际的counter最终数值{counter.value}')
    # ————未给increase操作上锁————
    # 期望的counter最终数值50000
    # 实际的counter最终数值18979

    counter = Value('i', 0) # 重新给counter值设置为0
    lock = Lock() # 创建锁

    # 加锁测试用Value对象共享变量是否会出现同步问题
    # 准备进程
    processes = [Process(target=add_with_lock, args=(counter, increase_times, lock)) for i in range(0, count_processes)]
    # 执行进程
    for p in processes:
        p.start()
    for p in processes:
        p.join()
    print('————给increase操作上锁————')
    print(f'期望的counter最终数值{0 + count_processes * increase_times}')
    print(f'实际的counter最终数值{counter.value}')
```

如果进程通信交换的是集合内容比较大，可以考虑使用multiprocessing.Array，具体使用和Value对象类似

```py
from multiprocessing import Array
arr = Array(typecode_or_type, size_or_initializer, lock=True)
```

`size_or_initializer`：指定数组长度，或提供一个可迭代对象（如列表）来初始化数组。

共享Array容器的读写和list容器类似

```py
arr = Array('i', [10, 20, 30, 40])

print(arr[::]) # [10, 20, 30, 40]

# 读取单个元素
print(arr[0])        # 10

# 修改元素
arr[1] = 99
print(arr[1])        # 99

# 切片返回一个新的列表（不是共享数组）
sliced = arr[1:3]
print(sliced)        # [99, 30]

# 获取长度
print(len(arr))      # 4

# 遍历
for value in arr:
    print(value)
```

**GIL**是全局解释器锁（互斥锁），GIL的存在导致同一进程的不同线程必须竞争这把锁来使用CPU执行字节码，所以python多线程和操作系统的多线程不是一一对应的，无法做到并行。

GIL存在的意义是为了保证**内存管理的线程安全**和**C扩展模块的便利性**

- 引用计数问题

  CPython 使用**引用计数**来管理内存。每个 Python 对象都有一个引用计数，记录有多少地方引用了它。当引用计数降为 0 时，对象的内存会被回收。

  引用计数管理如果没有GIL的话会有`内存泄漏`或`程序崩溃`的风险

  ![image-20260306163708312](assets/image-20260306163708312.png)

  ![image-20260306163738285](assets/image-20260306163738285.png)

- 方便扩展C扩展模块

  许多py的底层库如numpy、pandas是c代码，这些扩展代码通常是假定在GIL保护下运行可以安全地访问py对象，而不必考虑多线程同步。如果**没有GIL编写这些扩展代码将需要处理大量线程安全细节**

### 实用包

pprint

[一文弄懂Python中的pprint模块 - 知乎](https://zhuanlan.zhihu.com/p/508317313#:~:text=pprint的英文全称Data pretty printer，顾名思义就是让显示结果更加直观漂亮。)

