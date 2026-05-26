## Python 语法规范

Python 的语法规范主要突出简洁、可读性高的特性，所以它与其他语言在语法风格上有很大不同，需重点注意。

### 语法规范

1. Python 字符串**不区分单双引号**

   ```python
   name_str = "Jack"
   name_str2 = 'Jack'
   
   # 两者是一致的
   ```

2. 每行代码**结尾不需要加 `;` 分号**

   ```python
   name = "Jack"
   age = 18
   def my_function(name: str) -> None:
       print("hello world", name)
   ```

3. Python 规定**使用 `Tab | Spaces` 字符缩进**来**表示代码块的作用域**。

   ```python
   # 变量作用域
   global_var = "全局变量"
   
   print(global_var)  # 输出：全局变量
   
   
   def my_function():
       # 按照缩进来判断作用域
       local_var = "局部变量"
   
       print(local_var)  # 输出：全局变量 局部变量
   
   
   my_function()
   
   # 全局作用域访问局部变量 local_var 会报错
   # print(local_var)  # NameError: name 'local_var' is not defined
   ```


### 命名规范

**Python 官方规范**：**PEP 8**（Python Enhancement Proposal 8）

核心命名规范：

| 类型               | 命名规范             | 示例                              | 公开/私有        |
| :----------------- | :------------------- | :-------------------------------- | :--------------- |
| **变量**           | 小写字母 + 下划线    | `user_name`, `total_count`        | 公开             |
| **常量**           | 全大写 + 下划线      | `MAX_SIZE`, `PI`                  | 公开             |
| **函数**           | 小写字母 + 下划线    | `get_user()`, `calculate_sum()`   | 公开             |
| **类**             | 大驼峰（PascalCase） | `UserManager`, `CarModel`         | 公开             |
| **模块**           | 小写字母 + 下划线    | `user_utils.py`, `data_parser.py` | 公开             |
| **包**             | 小写字母（无下划线） | `mypackage`, `utils`              | 公开             |
| **私有属性**       | 单下划线前缀         | `_internal_var`                   | 受保护           |
| **私有属性（强）** | 双下划线前缀         | `__private_var`                   | 私有（名称修饰） |
| **特殊方法**       | 双下划线包裹         | `__init__`, `__str__`             | 魔术方法         |

#### 示例

##### 变量命名

```python
# ✅ 正确：小写 + 下划线
user_name = "Alice"
total_count = 100
is_active = True
user_list = [1, 2, 3]

# ❌ 错误
userName = "Alice"      # 驼峰命名（这是Java风格）
UserName = "Alice"      # 首字母大写（像类名）
u = "Alice"            # 单字母（除了循环变量）
user-name = "Alice"     # 中划线（Python不支持）
```

单字母变量：

```python
# ✅ 允许：短循环变量
for i in range(10):
    print(i)

# ✅ 允许：坐标
x, y = 10, 20

# ✅ 允许：数学公式中的临时变量
for _ in range(5):      # _ 表示不需要的值
    print("Hello")
```

##### 常量命名

```python
# ✅ 正确：全大写 + 下划线
MAX_CONNECTIONS = 100
PI = 3.14159
DATABASE_URL = "postgresql://localhost"

# 模块级别常量
DEFAULT_TIMEOUT = 30

# ❌ 错误
maxConnections = 100   # 小写看起来像变量
MAXConnections = 100   # 大小写混乱
```

##### 函数与方法命名

```python
# ✅ 正确：小写 + 下划线
def calculate_average(scores):
    return sum(scores) / len(scores)

def get_user_by_id(user_id):
    # 查询用户
    pass

def is_valid_email(email):
    return '@' in email

# ❌ 错误
def CalculateAverage(scores):  # 大驼峰（应该用于类）
    pass

def getUserById(user_id):      # 驼峰命名
    pass
```

**特殊函数命名：**

```python
# 谓词函数（返回布尔值）- 通常以 is/has/can 开头
def is_authenticated(user):
    return user.token is not None

def has_permission(user, action):
    return action in user.permissions

def can_edit(user, document):
    return user.id == document.author_id
```

##### 类命名

```python
# ✅ 正确：大驼峰（每个单词首字母大写）
class UserAccount:
    pass

class CarModel:
    pass

class HTTPSConnection:    # 缩略词可以全大写
    pass

# ❌ 错误
class userAccount:        # 小写开头
    pass

class user_account:       # 下划线分隔
    pass
```

##### 模块与包命名

```python
# 模块文件（.py 文件）：
# ✅ 正确
user_utils.py
data_parser.py
db_connection.py

# ❌ 错误
UserUtils.py    # 不要用驼峰
user-utils.py   # 不要用中划线
utils.py        # 太通用，容易冲突

# 包（目录）：
# ✅ 正确
mypackage/
utils/
database/

# ❌ 错误
my-package/     # 不要用中划线
MyPackage/      # 不要用大写
```

##### 私有成员命名

```python
class User:
    def __init__(self, name, password):
        self.name = name           # 公开属性
        self._age = 0              # 受保护属性（单下划线）
        self.__password = password # 私有属性（双下划线）
    
    def get_info(self):
        """公开方法"""
        return self.name
    
    def _internal_method(self):
        """受保护方法（内部使用）"""
        pass
    
    def __encrypt_password(self):
        """私有方法（名称修饰）"""
        pass

# 单下划线：约定"不要直接访问"
user = User("Alice", "123")
print(user._age)      # 可以访问，但不推荐

# 双下划线：名称修饰（name mangling）
print(user._User__password)  # 实际被重命名为 _User__password
```

**名称修饰（Name Mangling）示例：**

```python
class MyClass:
    def __init__(self):
        self.__private = 42

obj = MyClass()
print(dir(obj))  # 可以看到 '_MyClass__private'
# print(obj.__private)  # AttributeError
print(obj._MyClass__private)  # 42（可以访问，但不应该）
```

##### 魔术方法

```python
class MyClass:
    # ✅ 正确：双下划线包裹
    def __init__(self):
        pass
    
    def __str__(self):
        return "MyClass instance"
    
    def __len__(self):
        return 0
    
    def __enter__(self):
        # 上下文管理器
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        pass

# ❌ 错误：不要自定义这种命名
def __my_custom__(self):  # 只有Python内置的魔术方法才能这样
    pass
```

#### 特殊场景规范

##### 异常类命名

```python
# ✅ 正确：以 Error/Exception 结尾
class ValidationError(Exception):
    pass

class DatabaseConnectionError(Exception):
    pass

class UserNotFoundError(Exception):
    pass

# 使用
raise ValidationError("Invalid email format")
```

##### 全局变量

```python
# 模块级全局变量
_module_counter = 0      # 单下划线：模块私有
PUBLIC_CONFIG = "value"  # 全大写：公开常量

def increment_counter():
    global _module_counter
    _module_counter += 1
```

##### 类型变量（泛型）

```python
from typing import TypeVar, Generic

# ✅ 正确：单个大写字母或加上 _co/_contra
T = TypeVar('T')           # 常用
K = TypeVar('K')           # Key 类型
V = TypeVar('V')           # Value 类型
T_co = TypeVar('T_co', covariant=True)

class Stack(Generic[T]):
    def __init__(self):
        self.items: list[T] = []
    
    def push(self, item: T) -> None:
        self.items.append(item)
```

##### 测试函数命名

```python
# ✅ 测试函数：test_ 前缀 + 描述性名称
def test_user_creation():
    pass

def test_calculate_average_with_empty_list():
    pass

def test_login_with_valid_credentials():
    pass

def test_should_raise_error_when_user_not_found():
    pass
```

#### 错误示例

```python
# ❌ 1. 使用关键字作为变量名
class = "Python"     # SyntaxError
def = 123           # SyntaxError
from = "abc"        # SyntaxError

# ✅ 解决方法：加下划线
class_ = "Python"
def_ = 123
from_ = "abc"

# ❌ 2. 混淆相似的名称
l = 1              # 容易看成数字 1
O = 0              # 容易看成数字 0
I = 1              # 容易看成数字 1

# ✅ 避免使用
length = 1
value = 0
index = 1

# ❌ 3. 使用双下划线开头和结尾（除了魔术方法）
__myvar__ = 42     # 看起来像魔术方法，但实际不是

# ❌ 4. 过于通用的名称
data = "something"
temp = 42
foo = calculate()  # foo/bar/baz 只在极简示例中使用

# ✅ 使用描述性名称
user_input = "something"
retry_count = 42
result = calculate()
```

### 缩进规范

**缩进是 Python 的灵魂** - 与其他语言使用花括号 `{}` 表示代码块不同，Python **强制使用缩进**来表示**代码的层次结构**。

即**缩进 = 代码块作用域**。在编写多层嵌套代码块时，IDE 编辑器也会自动在下一行进行缩进。

注：如果在编写代码过程中，代码前没有缩进，那么则无法识别出它的作用域。

> [!WARNING]
>
> Python **对缩进要求严格**，**混用空格和 Tab** 是**导致缩进错乱**的最常见原因。

```python
# Python：使用缩进表示代码块
if True:
    print("这是缩进")
    print("这也是同一个代码块")
print("这是外层代码")
```

```javascript
// JavaScript：使用花括号
if (true) {
    console.log("这是代码块");
    console.log("这也是同一个代码块");
}
console.log("这是外层代码");
```

```java
// Java：使用花括号表示代码块
if (true) {
    System.out.println("这是代码块");
    System.out.println("这也是同一个代码块");
}
System.out.println("这是外层代码");
```

- 嵌套代码块

```python
# 多层嵌套（最多建议3-4层）
def outer_function():
    x = 10
    
    if x > 0:
        for i in range(3):
            if i % 2 == 0:
                print(f"偶数：{i}")
            else:
                print(f"奇数：{i}")
    
    return x

# 缩进级别计数：
# 0级：函数定义
# 1级：if 语句
# 2级：for 循环
# 3级：if/else 语句
```

#### 基本规则

| 规则          | 说明                       | 示例                 |
| :------------ | :------------------------- | :------------------- |
| **统一数量**  | **同一代码块缩进必须一致** | 都是4空格或都是2空格 |
| **推荐4空格** | PEP 8 官方推荐             | 使用4个空格          |
| **禁止混用**  | **不能混用空格和Tab**      | ❌ 空格+Tab           |
| **冒号触发**  | **`:` 后下一行必须缩进**   | `if x > 0:`          |

#### 规范示例

```python
# 1. if/elif/else 语句
if condition:
    print("缩进")  # 必须缩进
else:
    print("也要缩进")

# 2. for/while 循环
for i in range(3):
    print(i)  # 必须缩进

# 3. 函数定义
def my_function():
    return "缩进"  # 必须缩进

# 4. 类定义
class MyClass:
    def method(self):
        pass  # 必须缩进

# 5. with 语句
with open('file.txt') as f:
    content = f.read()  # 必须缩进

# 6. try/except/finally
try:
    risky_code()  # 必须缩进
except Exception:
    handle_error()  # 必须缩进
```

#### 错误示例

```python
# ❌ 错误1：忘记缩进
if True:
print("没有缩进")  # IndentationError

# ❌ 错误2：缩进不一致
if True:
    print("4个空格")
  print("2个空格")  # IndentationError

# ❌ 错误3：混用空格和Tab
if True:
    print("空格")  # 假设这里是4个空格
	print("Tab")   # 这里是Tab - 错误！

# ❌ 错误4：多余缩进
print("不应该缩进")  # 顶格写
    print("多余缩进")  # IndentationError
```

#### 缩进规则配置

##### .editorconfig 项目统一配置

```bash
# ✅ 整个项目统一使用4空格
# 在项目根目录创建 .editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.py]
indent_size = 4
```

##### Cursor 修改缩进规则

如果只想为 Python 修改缩进规则：

1. 可以通过 `Ctrl + Shift + P` 打开命令面板，输入`settings.json` 
2. 进入 **`settings.json` 用户配置**模式
3. 添加或修改以下配置，这能确保 Python 文件始终使用 4 个空格：

```json
{
  "[python]": {
    "editor.tabSize": 4, // 缩进字符个数
    "editor.insertSpaces": true // 是否插入 space 空格字符
  }
}
```

##### Cursor 缩进字符可视化

如果觉得使用缩进代替 `{}` 代表代码块会比较容易混淆，可以通过 IDE 编辑器自带的缩进字符可视化功能来优化视觉效果。

可以让编辑器将看不见的缩进符号（空格或制表符）显示出来，这在排查因混用空格和 Tab 导致的错误时非常有用。

- 使用快捷键 `Ctrl + Shift + P`。
- 输入 `Toggle Render Whitespace` 并执行。
- **效果**：开启后，所有空格会显示为淡灰色的点 `·`，制表符会显示为 `→`。

![](media/缩进字符可视化.png)

#### 代码自动格式化工具

Cursor 支持通过**`black`** 和 **`autopep8`** 依赖包 和相应的 **IDE 插件**来配置两种方式。

区别在于：

- **依赖包**：需要每次都通过控制台来**手动代码格式化**
- **IDE 插件**：可以通过配置来实现 **`Ctrl + S`** 或 **`Shift + Alt + F`自动代码格式化**

##### black 代码格式化

###### black 依赖包

作用：用于**格式化某个 `.py` 文件或者整个 Python 项目的代码规范**。

- 通过 `pip` 安装：

  ```bash
  # 安装 black（自动格式化）
  >>> pip install black
  ```

- 使用：

  ```bash
  # 格式化单个文件
  >>> black myfile.py
  
  # 格式化整个项目
  >>> black .
  ```

> [!WARNING]
>
> 注：`black` **只能对代码进行格式化**，**无法检查并修复缩进规范**。可以通过 `autopep8` 依赖包来实现。

###### Black Formatter (IDE 插件)

在插件市场中搜索 `Black Formatter` 插件并安装，之后便不再需要手动检查代码规范了。

> 这个插件其实内部使用的就是 `black` 依赖包的规则，相当于格式化方式从控制台变成了快捷键。

![](media/Black Formatter 插件.png)

##### autopep8 修复缩进

**Python 官方规范**：**PEP 8**（Python Enhancement Proposal 8）

###### autopep8 依赖包

作用：用于**修复某个 `.py` 文件的缩进错误**。

- 通过 `pip` 安装：

  ```bash
  # 安装 autopep8（修复缩进）
  >>> pip install autopep8
  ```

- 使用：

  ```bash
  # 修复缩进问题
  autopep8 --in-place --aggressive myfile.py
  ```

###### autopep8 (IDE 插件)

在插件市场中搜索 `autopep8` 插件并安装。

> 这个插件其实内部使用的就是 `autopep8` 依赖包的规则，相当于格式化方式从控制台变成了快捷键。

![](media/autopep8 IDE 插件.png)

##### Ruff  (IDE 插件)

在插件市场中搜索 `Ruff` 插件并安装，目前 Python 社区更推荐使用 **Ruff** 来取代 `autopep8`。它不仅能做格式化，还集成了代码检查（Linting）功能，速度极快，正逐渐成为主流的 Python 开发工具。

![](media/Ruff 插件.png)

##### 配置插件

安装好插件之后，可通过 `Shift + Alt + F`  手动进行代码格式化，如果没有配置，Cursor 会进行提示：

![](media/Cursor 配置代码格式化插件.png)

也可以在 **`settings.json` 用户配置**中手动编写配置 `"editor.defaultFormatter"` ：

<img src="media/settings.json 手动配置 python 规范.png" style="zoom: 67%;" />

> 跟 VS Code 使用一致。

##### Cursor 配置保存时自动格式化

如果不想通过 `Shift + Alt + F`  手动进行代码格式化，而是通过 `Ctrl + S` 保存文件时自动对进行代码格式化。

1. 可以通过 `Ctrl + Shift + P` 打开命令面板，输入`settings.json` 
2. 在 **`settings.json` 用户配置**中配置 `"editor.formatOnSave": true`

<img src="media/Cursor 配置自动代码格式化.png" style="zoom: 67%;" />

### # 注释

**注释** 是代码中用于解释说明的文字，**不会被 Python 解释器执行**。

| 作用             | 说明                                 |
| :--------------- | :----------------------------------- |
| **解释代码逻辑** | 让其他人（或未来的自己）理解代码意图 |
| **调试代码**     | 临时禁用某些代码段                   |
| **生成文档**     | 配合工具自动生成 API 文档            |
| **TODO 标记**    | 标记待完成的功能                     |

以下是 Python 常用的注释类型：

#### 普通注释

- **单行注释：**

  ```python
  # 这是单行注释
  print("Hello World")  # 行尾注释
  
  # 可以有多行连续的注释
  # 这是第二行注释
  # 这是第三行注释
  ```

- **行尾注释：**

  ```python
  print("Hello World")  # 行尾注释
  ```

- **多行注释：**

  - **连续 `#` 单行注释（推荐）**：

    ```python
    # 这是推荐的多行注释方式
    # 每行都以 # 开头
    # 清晰明了，符合 PEP 8 规范
    def my_function():
        pass
    ```

  - **三引号字符串（不推荐）：**

    ```python
    """
    这是三引号字符串
    实际上是一个字符串字面量
    不会被赋值给任何变量
    """
    print("Hello")
    
    '''
    也可以使用三个单引号
    同样不会被执行
    '''
    ```

    > 三引号字符串并不是真正的注释，它本质上是一个**字符串字面量**，它会被 Python 解析器解析，只是没有被任何变量引用，不产生实际效果。
    >
    > 因此：
    >
    > - 会占用一定的解析时间
    > - 在字节码中仍然存在
    > - 可能干扰文档字符串（docstring）

- **注释的缩进：**

  ```python
  def my_function():
      # 注释与代码保持相同缩进级别
      if condition:
          # 缩进4个空格
          do_something()
          
          # 空行后的注释也保持相同缩进
          do_another()
  ```

#### 特殊注释

- **TODO：注释**：标记**待完成的任务**

  ```python
  # TODO: 添加错误处理
  # TODO(zhangsan): 优化查询性能
  def get_user_data():
      pass
  ```

- **FIXME：注释**：标记**此处存在问题，需要修复**

  ```python
  # FIXME: 这里存在内存泄漏问题
  def leaky_function():
      large_list = [0] * 10000000
      return large_list  # 返回后未释放
  ```

- **BUG：注释**：标记**此处的函数执行会有某个 BUG**

  ```python
  # BUG: 当输入为负数时会崩溃
  # 原因：sqrt() 函数不支持负数
  # 临时方案：添加输入验证
  def calculate_sqrt(x):
      return x ** 0.5  # 负数会出错
  ```

- **HACK：注释**：标记**此处使用了一个 HACK 写法，需要后续优化**

  ```python
  # HACK: 临时解决方案，等待官方修复
  # 绕过第三方库的 bug
  def workaround():
      pass
  ```

#### 文件编码声明

##### Shebang

**Shebang**（也称为 Hashbang）是一个由 `#!` 开头的特殊注释，用于**指定脚本的解释器路径**。

```python
#!/usr/bin/env python3
# 告诉系统用哪个解释器执行此脚本

print("Hello World")
```

##### 编码声明

声明文件编码（Python 3 默认 utf-8，通常不需要）

```python
# -*- coding: utf-8 -*-
# 或
# coding: utf-8

print("Hello World")
```

#### Docstring 文档字符串（用户手册）

Docstring 文档是**紧贴在模块、类、函数或方法定义后的第一行说明的字符串字面量，且没有被变量引用**，这种 docstring 会作为**当前类、函数对象的`__doc__`属性**，用于生成 API Doc 官方文档作为用户手册。

```python
def my_function(name: str) -> None:
  """
  这里的注释会被自动生成到 DOC 文档中
  :param name: 传入的参数
  :return -> None
  """
  print('hello world', name)
```

IDE 会自动根据Docstring 生成内容，当用鼠标悬停在类名、函数名时可以看到：

<img src="media/docstring 说明文档.png" style="zoom:67%;" />

##### 编写规范

- **Sphinx/reStructuredText (标准库风格)**：Python 官方文档以及众多大型开源项目（如 Django、Requests）的首选

  使用 **`:param`** 和 **`:return`** 等标记，虽然机器解析友好，但纯文本阅读体验稍逊。

  | **标记**             | **用途**             | **示例**                             |
  | -------------------- | -------------------- | ------------------------------------ |
  | **`:param <name>:`** | 定义参数描述         | `:param path: 文件的绝对路径`        |
  | **`:type <name>:`**  | 指定参数的类型       | `:type path: str`                    |
  | **`:returns:`**      | 描述返回值           | `:returns: 处理后的字符串列表`       |
  | **`:rtype:`**        | 指定返回值的类型     | `:rtype: list`                       |
  | **`:raises <Err>:`** | 声明可能抛出的异常   | `:raises ValueError: 当路径不存在时` |
  | **`:var <name>:`**   | 描述类变量或成员变量 | `:var status: 当前连接状态`          |

  ```python
  def fetch_user_data(user_id, timeout=10):
      """
      通过 API 获取用户信息。
  
      :param user_id: 用户的唯一识别 ID。
      :type user_id: int
      :param timeout: 请求超时时间（秒），默认为 10。
      :type timeout: int
  
      :returns: 包含用户数据的字典。
      :rtype: dict
  
      :raises ConnectionError: 当 API 无法访问时抛出。
      :raises ValueError: 当 user_id 格式错误时抛出。
  
      .. note::
         此函数需要有效的网络连接和 API Key。
  
      .. versionadded:: 2.0
      """
      pass
  ```

###### autoDocstring 插件

这是目前最主流的解决方案。VS Code (Cursor) 自带的 Python 插件虽然支持基础功能，但这个专用扩展能提供更精细的风格控制。

1. 在 Cursor 的插件市场搜索并安装 **"Python Docstring Generator"** (作者是 Nils Werner)。
2. 进入插件设置（`Ctrl+,` 然后搜索 `autoDocstring.docstringFormat`）。
3. 将选项从 `google` 修改为 **`sphinx`**。

> **输入触发**：在函数定义的下一行输入三个引号 `"""` 然后按下 `Enter`，可以**根据当前函数自动生成 Docstring**

##### 输出 API Doc

- 通过**`print(my_function.__doc__)`** 输出函数的 API Doc

```python
print(my_function.__doc__)
# 输出：
# 这里的注释会被自动生成到 DOC 文档中
# :param name: 传入的参数
# :return -> None
```

- 通过 **`help(my_function)`** 输出函数的 API Doc

```python
help(my_function)
# Help on function my_function in module __main__:
#
# my_function(name: str) -> None
#   :param name: 传入的参数
#   :return -> None
#
# None
```

### 代码检查工具

#### pylint 检查风格

作用：可以对指定 `.py` 文件的**代码规范进行检查**，并给出评分。

```bash
# 安装、下载
pip install pylint

# pylint 检查缩进
pylint myfile.py
```

![](media/pylint 检查缩进.png)

##### .pylintrc 自定义统一规则

```bash
# .pylintrc
[BASIC]
# 变量名最小长度
variable-rgx=[a-z_][a-z0-9_]{1,30}$
# 常量名格式
const-rgx=(([A-Z_][A-Z0-9_]*)|(__.*__))$
# 函数名格式
function-rgx=[a-z_][a-z0-9_]{2,30}$
# 类名格式
class-rgx=[A-Z_][a-zA-Z0-9]+$
```

#### flake8 代码规范检查

作用：可以对指定 `.py` 文件的**代码规范进行检查**，并给出相应错误提示。

- 通过 `pip` 安装到本地项目

```bash
# 安装、下载
pip install flake8

# 使用
flake8 myfile.py
```

![](media/flake8 代码检测工具.png)

#### IDE settings.json 配置

`settings.json` 用户配置：

```json
{
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true,
    "python.linting.flake8Enabled": true,
    "editor.rulers": [79],  // PEP 8 建议每行79字符
    "python.formatting.provider": "black"
}
```

