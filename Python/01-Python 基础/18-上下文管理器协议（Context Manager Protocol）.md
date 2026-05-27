## 上下文管理器协议（Context Manager Protocol）

> [!NOTE]
>
> 上下文管理器协议（Context Manager Protocol）：
>
> 协议将 **“业务逻辑”** 与 **“资源清理逻辑”** 彻底解耦。
>
> - **解耦**：开发者只需要关注 `with` 缩进里的逻辑，不用担心忘记关闭连接
> - **安全**：即代码中途 `return` 或崩溃，`__exit__` 依然会被调用
> - **可复用**：一次编写协议，处处可用

### 实现方式

上下文管理器协议的**实现方式**有两种：

- **类实现**：只需要**在类中实现 `__enter__()` 和 `__exit__()` 方法**即可
- **`@contextmanager` 上下文管理器装饰器**：**修饰一个生成器函数，与生成器函数的 `yield` 结合**来快速构建一个上下文管理器

上下文管理器协议实现方式的选择取决于具体的需求和场景，且**必须配合 `with` 语句结构使用**。

### with 上下文管理语句

#### 基本概念

在 Python 中，`with` 语句是一个非常强大且优雅的语法结构，主要用于**外部资源的上下文管理**。

##### 与协议的关系

- **上下文管理器协议**：如何对外部资源进行自动化管理的**抽象规范**
- **`with`语句**：Python **在代码层面上**，**按照协议的规范进行自动化管理外部资源的工具**

> `with` 语句的工作原理是**基于上下文管理器协议（Context Manager Protocol）实现**的，它**可以配合上下文管理器类与上下文管理器装饰器**一同使用。

##### 为什么需要 with？

核心结论：`with` 的存在就是为了让代码更 **Pythonic**（简洁、易读），并极大降低了因忘记手动释放资源而导致的 Bug 风险。

###### 关注点分离

核心概念：通过**关注点分离机制**，将操作外部资源**代码中的『业务逻辑』与『外部资源管理』强行分离**。Python 内部**自动处理『外部资源管理』**，开发者**只需关注『业务逻辑』代码**。

在 Python 中，对于外部资源的操作，通常都会分为 3 个步骤：

- **『打开/连接 外部资源通道』→ 『操作外部资源逻辑』 → 『关闭/释放 外部资源通道』**

以上这 3 个步骤，**『打开/连接』 与 『关闭/释放』 **动作通常都是**必要的、“非业务逻辑” 的首尾模板代码**。

> 例如 文件操作 的传统做法：
>
> ```python
> f = open('./test.txt', 'r') # 打开/连接
> 
> # ... 文件读写业务逻辑
> 
> f.close()  # 关闭/释放；必须操作，若忘记释放资源会导致内存占用高，消耗无用的内存性能
> ```

以上传统写法对于开发者来说**在代码结构层面上『业务逻辑』与『外部资源管理』是强内聚的**；代码结构混乱且不清晰，可读性较差；

于是， Python 提供了 `with` 语法结构，它**可以帮助开发者将『业务逻辑』与『外部资源管理』在语法层面上强行解耦（关注点分离）**，让开发者**只关注『业务逻辑』**；通过 Python 的 `with` 语句**内部自动去处理外部资源的『打开/连接』与『关闭/释放』这2个必要的 “非业务逻辑” 的模板代码**，以确保外部资源被正确地清理释放（防止开发者忘记手动关闭资源导致内存溢出）。

> 改善后的 `with` 语句结构：
>
> ```python
> with open('./test.txt', 'r') as f:
> # ... 文件读写业务逻辑
> 
> # 离开缩进块后，文件会自动关闭，无需手动写 f.close()
> ```

简单来说，**Python 中的 `with` 主要用于管理程序中 “任何成对出现的操作”**。例如：上锁 / 解锁、打开 / 关闭...，从而让开发者省心省力的关注业务逻辑即可。

- 核心要点：**`with` 语句**会将资源的**生命周期**封装在一个 **“上下文环境”** 内；只要环境结束，资源必然回收。

<img src="media/with语句的核心概念.png" style="zoom:60%;" />

#### 基本语法结构

```python
# 处理单个或多个资源
with <实现上下文管理器协议的 类 | 装饰器生成器函数>,... as <上下文对象>,...:
    # 调用外部资源的具体执行逻辑...
```

核心作用：`with` 语句能**确保外部资源（文件、数据库连接...）在使用后无论是否发生异常**，Python 程序都能**自动处理外部资源的分配和释放**，从而能**避免资源泄漏**（比如文件未关闭、数据库连接未释放），让**代码更简洁、安全**。

##### 要点说明

- **`<实现上下文管理器协议的 类 | 装饰器函数>`**：需要**引用外部资源的类、函数**，可以**通过 `with` 语句统一自动化管理**
  - 上下文管理器类：**实现了上下文管理器协议 `__enter__` 和 `__exit__` 魔术方法的资源管理类**
    - 例如 `open()` 函数就返回了一个自带 `__enter__` 和 `__exit__` 方法的类实例
  - 上下文管理器装饰器函数：**通过 `@contextlib.contextmanager` 装饰器修饰的生成器函数**，内部**使用 `yield` 来引用资源**
- **`<上下文对象>`**：**上下文管理器类、函数返回的一个上下文对象**，用于**在 `with` 代码块的 “上下文环境” 内进行资源的业务逻辑处理**

##### 核心特性

- with 会**自动调用 `<实现上下文管理器协议的 类 | 装饰器生成器函数>` 内部的 `__enter__()` 方法**，并**将返回值（上下文管理器类实例对象）通过 `as` 关键字 传递给 `<上下文对象>`**
- 当 with 代码块内的代码语句执行完毕， Python 解释器**即将跳出 with 代码块**时，会**自动调用 `<实现上下文管理器协议的 类 | 装饰器生成器函数>` 内部的 `__exit__()` 方法**，with 会**自动执行外部资源的【关闭/释放】动作**，无需手动处理

###### `as` 变量的本质

在 **`with ... as f`** 中的 **`f` 变量**，其实是 `__enter__` 方法的**返回值**：

业务分离：

- **`with` 后面的部分**：负责管理环境（资源管理逻辑）
- **`as` 后面的变量**：负责提供操作接口（业务逻辑工具）

> 价值：即使外部拿到了操作文件的句柄 `f`，也无法干预它何时关闭，因为关闭的权限被控制在 `with` 那个“环境”的手里。

```python
class MultipleReturns:
    def __enter__(self):
        # 可以返回任何对象
        return {"message": "Hello", "code": 200}
    
    def __exit__(self, *args):
        pass

with MultipleReturns() as obj:
    print(obj["message"])  # 绑定的是 __enter__ 的返回值
```

##### 示例

- 处理单个资源：

```python
# 传统的文件操作
f = open('./test.txt', 'r', encoding='utf-8')

r1 = f.readline()

f.close() # 手动释放文件资源


# with 简化代码
with open('./test.txt', 'r', encoding='utf-8'):
    r1 = f.readline()
    
# 无需手动编写 f.close()，由 with 去自动管理文件资源的释放
```

- 处理多个资源：

```python
# 传统 try-except-finally 读取文件内容
try:
    f = open('./test.txt', 'r', encoding='utf-8')
    r1 = f.readline()

    f2 = open('./test2.txt', 'r', encoding='utf-8')
    r2 = f2.readline()
except Exception as e:
    print(e)
finally:
    f.close()
    f2.close()

    
# 简化的 with 语法结构
with open('./test.txt', 'r', encoding='utf-8') as f, open('./test2.txt', 'r', encoding='utf-8') as f2:
    r1 = f.readline()
    r2 = f2.readline()

print(r1)
print(r2)
```

注意点：**`with` 没有块级作用域**，所以**在 `with` 代码块内赋值的成员可以在 `with`代码块外部依旧使用**。

#### 使用场景

只要代码符合**“先……后……”**的**对称逻辑结构**，就可以**使用 `with` 语句来自动化管理**：

- **打开**文件 -> **关闭**文件
- **获取**锁 -> **释放**锁
- **连接**数据库 -> **断开**数据库
- **修改**环境变量 -> **还原**环境变量

在实际应用中，可以使用 `with` 上下文管理器来管理文件、网络连接、数据库连接等资源，确保资源在使用完毕后能够正确释放。

| **场景**                    | **with 的作用**                                  |
| --------------------------- | ------------------------------------------------ |
| **文件操作**                | 自动关闭文件流，防止内存泄漏。                   |
| **线程锁 (Threading Lock)** | 自动获取锁并在结束后释放锁，避免死锁。           |
| **数据库连接**              | 确保事务提交或回滚，并关闭连接。                 |
| **网络请求**                | 确保 Socket 连接在使用后被关闭。                 |
| **临时环境修改**            | 比如临时修改系统路径或环境变量，结束后自动恢复。 |

- 文件操作：

  ```python
  # 传统写法需要手动关闭
  f = open('file.txt', 'r')
  try:
      content = f.read()
  finally:
      f.close()
  
  # 使用 with，自动关闭
  with open('file.txt', 'r') as f:
      content = f.read()
  # 离开 with 代码块后，文件自动关闭
  
  # 处理多个文件资源
  with open('input.txt', 'r') as infile, open('output.txt', 'w') as outfile:
      outfile.write(infile.read())
  ```

- 数据库连接：

  ```python
  import sqlite3
  
  with sqlite3.connect('database.db') as conn:
      cursor = conn.execute('SELECT * FROM users')
      # 退出时自动提交事务并关闭连接
  ```

- 线程锁：

  ```python
  import threading
  
  lock = threading.Lock()
  with lock:
      # 自动获取和释放锁
      print('线程安全操作')
  ```

### 类实现协议

协议（Protocol）在 Python 中通常指一组特定的魔术方法。

#### 基本组成

上下文管理器协议包含以下 `__enter__` 和 `__exit__` 魔术方法，只要任何类实现了这 2 个魔术方法，它就会成为了一个 **上下文管理器**。

核心价值：**任何实现了 `__enter__` 和 `__exit__`方法的类都可以与 `with` 语句一起使用**。

> 注意：`__enter__` 和 `__exit__` 魔术方法是**成双成对实现的，不能只实现其一！**

##### 基本语法结构

```python
# 协议实现定义
class <上下文管理器类>:
    
    def __enter__(self): pass

	def __exit__(self, exc_type, exc_val, exc_tb): pass


# 协议使用
with <上下文管理器类>() as <上下文管理对象实例>:
    # 业务逻辑...
```

核心价值：**`with` 语句就是一个“兜底协议”。** 它把那些 “必须要做的琐碎清理工作” 从主逻辑里抽离出来，包在了 `__enter__` 和 `__exit__` 这两个隐形的盒子里。

##### \__enter__(self) 进入 with

###### 核心特性

- 触发时机：**进入 `with` 代码块之前触发**
- 作用：**准备资源**（打开文件、获取线程锁）
- 返回值：它的**返回值会被绑定到 `with ... as` 中 `as` 后面声明的变量上**，该变量会成为**一个上下文对象（Content Object）**

###### 语法结构

```python
# 上下文管理器协议实现
class MyContextManager:
    # 进入 with 代码块时触发
    def __enter__(self):
        print("进入 with 上下文")
        return self # 返回资源对象本身

    # 退出 with 代码块时触发
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("离开 with 上下文")
        # 释放资源处理...

    # with 代码块内的业务逻辑
    def do_something(self):
        print("with 代码块业务逻辑")
        # 业务逻辑...


# 使用上下文管理器
with MyContextManager() as manager:
    manager.do_something()

# 输出：
# 进入 with 上下文
# with 代码块业务逻辑
# 离开 with 上下文
```

###### as 变量的绑定

```python
class MultipleReturns:
    def __enter__(self):
        # 可以返回任何对象
        return {"message": "Hello", "code": 200}
    
    def __exit__(self, *args):
        pass

with MultipleReturns() as obj:
    print(obj["message"])  # 绑定的是 __enter__ 的返回值
```

##### \__exit__(self,...) 离开 with

###### 核心特性

- 触发时机：**退出 `with` 代码块时触发（无论是否发生异常都会触发）**
- 作用：**释放资源**（关闭文件、释放锁）

###### 异常处理

在 `with` 代码块内抛出的异常可以被 `__exit__()` 方法捕获到：

- 异常处理：如果 **`with` 代码块内发生异常**，异常信息会**通过 `exc_type`、`exc_val`、`exc_tb` 方法参数传入到 `__exit__()` 方法中** ，**依然进行关闭资源处理**
  - **`exc_type`**：**异常类型** （例如 `ValueError`）
  - **`exc_val`**： **异常实例** （例如 `ValueError("invalid input")`）
  - **`exc_tb`**：**追踪栈对象** （Traceback）
- 返回值：
  - **True**：`with` 代码块内发生的异常**会被吞掉**；IDE **终端控制台不会报错**
  - **False**：`with` 代码块内的异常会**继续向上抛出**；IDE **终端控制台会报错**

###### 语法结构

```python
# 上下文管理器协议实现
class MyContextManager:
    # 进入 with 代码块时触发
    def __enter__(self):
        print("进入 with 上下文")
        return self # 返回资源对象本身

    # 退出 with 代码块时触发
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("离开 with 上下文")
        if exc_type is not None:  # 如果有异常抛出
            print(f"异常类型: {exc_type}")  # 异常类型: <class 'AttributeError'>

            # 异常对象信息: 'MyContextManager' object has no attribute 'xx'
            print(f"异常对象信息: {exc_val}")

            # Trackback 追踪栈信息: <traceback object at 0x0000024654AF55C0>
            print(f"Trackback 追踪栈信息: {exc_tb}")

            return False  # 返回 False 表示异常未被处理，程序会崩溃

        return True  # 返回True表示异常被处理，程序不会崩溃

    # with 代码块内的业务逻辑
    def do_something(self):
        print("with 代码块业务逻辑")


# 使用上下文管理器
with MyContextManager() as manager:
    manager.do_something()
    manager.xx()  # 这里会抛出异常，但是不会影响上下文管理器的退出
	# raise ValueError("数据格式错误") # __exit__() 方法会捕获它

# 输出：
# 进入 with 上下文
# with 代码块业务逻辑
# 离开 with 上下文
#
# Traceback (most recent call last):
#   File "C:\Users\win\Desktop\Test\Python\first_demo\with_demo.py", line 27, in <module>
#   manager.xx()  # 这里会抛出异常，但是不会影响上下文管理器的退出
#   ^^^^^^^^^^
# AttributeError: 'MyContextManager' object has no attribute 'xx'
```

#### 上下文对象（Context Object）

核心定义：上下文对象广义上是指**实现了上下文管理器协议的类实例对象**，但在运行时，**上下文对象负责维护 `with` 代码块内执行的 “环境状态”**。

##### 运行机制

```python
# 上下文管理器协议实现
class MyContextManager:
    # 进入 with 代码块时触发
    def __enter__(self):
        print("进入 with 上下文")
        return self # 返回资源对象本身

    # 退出 with 代码块时触发
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("离开 with 上下文")
        return True  # 返回True表示异常被处理，程序不会崩溃

    # with 代码块内的业务逻辑
    def do_something(self):
        print("with 代码块业务逻辑")


# 使用上下文管理器
with MyContextManager() as manager: 
    # manager: 就是 MyContextManager() 执行 __enter__() 方法后返回的值【通常是 self 即上下文管理器类自己】
    manager.do_something()
```

当运行 `with MyContextManager() as manager` 时：

1. Python **调用 `MyContextManager()` 创建一个类实例对象**

2. 准备**进入 `with` 代码块**时，**调用类实例对象的 `__enter__()` 魔术方法**

3. 如果**此时使用了 `as manager`**，则**将 `__enter__()` 魔术方法返回的值赋给 `as` 后面声明的 `manager`**变量

   > 注意：**`as manager`中 `manager` 的值取决于 `__enter__` 魔术方法的  `return` 返回值**。即`manager` 不一定就是 `MyContextManager` 上下文管理器类的实例对象，也可以是别的值。

4. **进入 `with` 代码块，执行 `with` 代码块内的业务逻辑**

5. **无论 `with` 代码块发生什么**，在 **`with` 代码块内的语句执行完后**都会**调用** `MyContextManager` 类中的 **`__exit__()` 魔术方法进行释放资源处理**

<img src="media/with语句-上下文对象运行机制.png" style="zoom:55%;" />

##### 生命周期

上下文对象的生命周期极其明确，它严格遵循 **“由 `with` 启动，由缩进结束触发销毁”**。

- **诞生 (Creation)**：执行 `with <上下文管理器类>()`。此时对象被实例化，内存被分配
- **激活 (Activation)**：调用 `__enter__`。这是环境正式生效的时刻
- **存活 (Execution)**：执行 `with` 内部的代码块。这是资源的“服务期”
- **临终 (Teardown)**：调用 `__exit__`。无论块内是否报错，这个方法都会执行
- **销毁 (Destruction)**：`with` 块结束。如果没有其他强引用，该对象将被 Python 的垃圾回收机制（GC）处理

> 上下文对象的生命周期也可以通过手动调用 `__enter__` 和 `__exit__` 方法来控制。

```python
class ContextManager:
    def __init__(self, name):
        self.name = name
        print('__init__', self.name)

    def __enter__(self):
        print(f"进入 {self.name} 上下文")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"离开 {self.name} 上下文")

        return True


context_manager = ContextManager('context_manager')  # 强引用，上下文对象会一直存在
with context_manager:
    print("with 代码块业务逻辑")

print(context_manager)  # <__main__.ContextManager object at 0x00000161C0899010>

with ContextManager('context_manager') as c:
    print("with 代码块业务逻辑")

print(c)  # <__main__.ContextManager object at 0x00000161C0899010>
c.__enter__() # 强制进入
c.__exit__(None, None, None) # 强制离开
print(c)  # <__main__.ContextManager object at 0x00000161C0899010>
```

##### 作用域

`with` 并不会创建一个新的局部作用域，且 Python 也没有块级作用域。

所以**在 `with` 块内部赋值的变量（包括 `as` 后面的变量）**，在 `with` 块**结束后依然是可以访问的**。

```python
with open('test.txt', 'w') as f:
    data = "Hello"
    f.write(data)

print(data) # 依然可以访问，输出 "Hello"
print(f)    # 依然可以访问文件对象，但 f.closed 已经是 True 了
```

##### 状态保持

- 实例级的状态共享：

  ```python
  class TracebackManager:
      def __init__(self):
          self.start_time = None # 初始状态
  
      def __enter__(self):
          self.start_time = time.time() # 记录进入状态
          return self
  
      def __exit__(self, exc_type, exc_val, exc_tb):
          duration = time.time() - self.start_time # 计算并保持最终状态
          print(f"该环境运行了 {duration} 秒")
  ```

- 在 `with` 语句外部对上下文对象进行**复用**：利用 `with` 没有块级作用域，内部创建的变量外部依然可以访问的特性。

  可以**将上下文对象作为一个全局变量**，**在不同的 `with` 语句中使用**，而**多个 `with` 代码块内访问的上下午对象是同一个实例**，所以它们之间可以**实现上下文对象的状态保持**。

  ```python
  class Reusable:
      def __init__(self):
          self.counter = 0
  
      def __enter__(self):
          self.counter += 1
          print(f"进入第 {self.counter} 次")
          return self
  
      def __exit__(self, *args):
          print(f"退出第 {self.counter} 次")
          return False
  
      def do_something(self):
          print("with 代码块业务逻辑", self.counter)
  
  
  ctx = Reusable()
  with ctx:
      ctx.do_something()
  with ctx:  # 同一个对象可以再次使用
      ctx.do_something()
  
  '''
  输出结果：
  进入第 1 次
  with 代码块业务逻辑 1
  退出第 1 次
  进入第 2 次
  with 代码块业务逻辑 2
  退出第 2 次
  '''
  ```

### 生成器函数装饰器实现协议

Python 提供了一个名为 `contextlib` 的工具库，它有一个 **`@contextmanager` 装饰器可以修饰一个带 `yield` 的生成器函数**，快速创建上下文管理器。

#### @contextmanager 装饰器

核心价值：通过**一个生成器函数来定义一个一次性使用、逻辑简单的上下文管理器**，而不需要花费更多的代码去实现一个包含 `__enter__` 和 `__exit__` 方法类结构体。

##### 基本语法结构

```python
from contextlib import contextmanager

@contextmanager
def <生成器函数>():
    print("生成资源") # 等价于 __enter__()
    
    try:
        yield <上下文对象的值> # 等价于 __enter__() 的 return 返回值，由外部 with 的 as 变量接收
    finally:
        # 清理资源逻辑
        
    print("回收资源") # 相当于 __exit__ 的逻辑
    
with <生成器函数>() as <yield 返回的值>:
    # 业务逻辑...
```

##### 核心特点

以生成器函数内的 **`yield` 作为分界点**：

- **`yield` 之前的代码 = `__enter__()` 方法**
- **`yield` 返回的值 = `as` 后面的变量**
- **`yield` 之后的代码 = `__exit__()` 方法**

注意点：在装饰器内部，**务必使用 `try...finally` 包裹 `yield`**。如果**不加 `try...finally`，一旦 `with` 块内部报错，`yield` 之后的清理代码将永远不会被执行**。

```python
from contextlib import contextmanager

@contextmanager
def my_resource():
    print("【准备】资源初始化")
    try:
        yield "可用资源" # 这里是分水岭
    finally:
        print("【清理】无论是否报错都会执行")
```

#### 与类实现协议对比

| **维度**     | **contextmanager 装饰器**                        | **传统的类 (__enter__)**           |
| ------------ | ------------------------------------------------ | ---------------------------------- |
| **代码量**   | 极简，通常只有几行。                             | 较多，需要定义类和多个方法。       |
| **状态保持** | 利用闭包，变量直接在函数内访问。                 | 需要通过 `self` 传递。             |
| **适用性**   | 适合 **一次性、简单** 的逻辑。                   | 适合 **复杂、需多次复用** 的逻辑。 |
| **可读性**   | 像读故事一样（先做这个，产出那个，最后做那个）。 | 逻辑分散在不同的魔术方法中。       |

#### 使用场景

- 数据库事务管理：

  ```python
  @contextmanager
  def transaction(db_conn):
      cursor = db_conn.cursor()
      try:
          yield cursor
          db_conn.commit() # 成功则提交
      except Exception:
          db_conn.rollback() # 失败则回滚
          raise # 重新抛出异常供上层处理
  ```

- 文件操作：

  ```python
  from contextlib import contextmanager
  
  @contextmanager
  def with_file_func(filename):
      f = open(f"./{filename}", 'r', encoding='utf-8')
      try:
          yield f  # 把 f 文件对象传递出去
      finally:
          f.close()
  
  
  with with_file_func('《登黄鹤楼》.txt') as f: # f: yield f 返回的 f 文件对象
      print(f.read())
  ```

- 性能计时器（调试工具）：

  ```python
  import time
  from contextlib import contextmanager
  
  @contextmanager
  def time_it(label):
      start = time.perf_counter()
      try:
          yield
      finally:
          end = time.perf_counter()
          print(f"核心逻辑 [{label}] 耗时: {end - start:.4f}s")
  
  with time_it("数据库查询"):
      # 执行复杂的 SQL 或 API 请求
      time.sleep(1)
  ```

