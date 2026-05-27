## OOP 面向对象

### 基本概念

Python 是一门**OOP**（Object-Oriented Programming，面向对象编程）的编程语言，支持**类(class)**和**对象(object)**的概念。

面向对象：一种**以对象为中心去思考和组织代码的方式**，使得复杂的程序变得更容易**扩展、维护和协作**。

- **类 class**：用于描述一类事物的 **“模板”**，它**规定了一类事物所具有的 【属性】 和 【行为】**。
- **对象 object**：**一个拥有 【属性】 和 【行为】 的个体**，是构成现实世界与程序世界的基本单位。
  - **实例 instance**：**根据【类】创建的一个具体的【对象】就是【实例】（实例对象）；类是抽象的，实例是具体的**。
  - **实例化**：根据**类 “制造出” 对象的过程**，就叫实例化

<img src="media/类与对象的关系.png" style="zoom:67%;" />

### 类

Python 使用 **`class` 关键字来定义一个类**。类名规范使用**大驼峰写法（`XxxYyy`）**。

#### 基本结构

```python
# 注意缩进！
class Person:
    """类的文档字符串"""
    
    # 类属性 (所有实例对象共享)
    <类属性名>=<属性值>
    
    # 构造方法（初始化实例方法） 必写！
    def __init__(self):
        pass
    
    # 实例方法（自定义方法）
    def <自定义方法名>():
        pass
```

> 要想为一个类创建实例对象，`__init__()` 构造方法是必须的！

![](media/类对象内存分析.png)

#### \__init__() 构造方法

*类似于 Java 的 constructor()*

> `__init__` 是 Python 特有的魔术方法，表示为 Python 内置提供的类方法，通常固定命名。

描述：**固定命名**，用于**类的实例对象初始化**，为类实例对象的初始化过程中**添加一些自定义属性、执行方法**等操作...

```python
class Person:
    def __init__(self, <自定义形参>):
        pass
```

接收参数：

- **`self`**：表示**指向当前对象的实例**，相当于其他语言的`this`关键字。**必须显式声明**！
- **`自定义形参`**：**在类实例对象初始化过程中，为类实例对象添加的一些自定义初始化属性**

##### 有参与无参构造

- **无参构造**：Python 为**每个类都默认内置了 `__init__(self)` 无参构造方法**，如果**类实例对象不需要初始化属性，则可以忽略它**
- **有参构造**：当类的实例对象**需要在初始化时添加一些自定义属性，则必须显式定义 `__init__(self)` 构造方法**

```python
# 无参构造
class Person:
    pass

person = Person()

# 有参构造
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age

student = Student('张三', 18)
```

### 创建一个实例

在 Python 中没有 `new` 关键字，想要**对一个类构造一个实例化对象**，需通过 **`<类名>()`** 的语法来实现，并由其他变量接收。

```python
class Person:
    def __init__():
        pass

person = Person()
```

#### 实例对象的基本信息

##### print() 输出

通过 `print()` 直接输出实例对象，可以得到如下信息：

```python
person = Person()

print(person) # <__main__.Person object at 0x00000225DE7F8D70>
```

> 逐个拆解：
>
> - **`at 0x00000225DE7F8D70`**：表示**该实例对象在内存中的位置**
> - **`Person object`**：表示**该实例对象的所属类**是 Person
> - **`__main__.Person`**：表示**Person类的所属模块是 `__main__`（主模块）**
>
> 总结：`person` 变量是 `__main__` 模块中 Person 类的实例对象，它在内存中的位置是 `0x00000225DE7F8D70`

##### \__dict__ 实例对象的属性

默认情况下，Python 会为每个实例创建 `__dict__` 字典，绝大多数**实例对象自身的属性都存在这个字典里**。

```python
class Animal:
    """这是一个动物类的文档"""

    species = "Unknown"  # 类属性

    def __init__(self, name, age):
        self.name = name      # 实例属性
        self.age = age

    def speak(self):
        pass


class Dog(Animal):
    def __init__(self, name, age, breed):
        super().__init__(name, age)
        self.breed = breed


dog = Dog('旺财', 3, '金毛')
print(dog.__dict__)   # {'name': '旺财', 'age': 3, 'breed': '金毛'}
print(dog.__dict__['name'])   # '旺财'

# 甚至可以直接操作它（虽然不推荐）
dog.__dict__['name'] = '小黑'
print(dog.name)       # 小黑
```

##### \__class__ 找到所属类

作用：**实例对象**可以通过 **`__class__` 魔术属性**，它**指向当前实例对象所属的类定义，用来访问类的属性、方法**。

```python
class Test:
    def get_name(self, name):
        # 为当前操作的实例添加/修改属性
        self.name = name
        print(name)


test = Test()

test.get_name('jack') # 'jack'
test.__class__.get_name(test, 'jack') # 'jack'

# 类型判断
print(test.__class__ is Test) # True
```

<img src="media/__class__ 魔术属性的机制.png" style="zoom: 50%;" />

##### 类型检查

可以通过 `type()` 和 `isinstance()` 方法检查实例对象的类型。

###### type(obj)

作用：**输出 `obj` 实例对象的类型**。

```python
class Test:
    def __init__():
        pass

test = Test()

print(test) # <class '__main__.Test'>
```

拆解：`__main__.Test` => test 变量是`__main__` 模块的 Test 类实例对象

###### isinstance(obj,Obj)

作用：**判断 `obj` 变量是否是 `Obj`类或其子类的实例对象**

```python
class Test:
    def __init__():
        pass

test = Test()

print(isinstance(test, Test))  # True
print(isinstance(test, object))  # True object 是所有类型的基类

print(isinstance(test, dict))  # False
```

#### 实例对象操作

Python 规定**通过 `.` 点语法可以访问/修改实例对象的属性、调用方法**等操作。

假设有以下类：

```python
class Test:
    name: str = 'test'

    def __init__(self):
        pass
```

##### 属性操作

###### . 访问

直接通过 **`<实例对象>.<属性名>`** 的语法来访**问实例对象的属性**：

```python
test = Test()

# 访问实例属性
print(test.name)
```

###### . 增加/修改

直接通过 **`<实例对象>.<属性名> = <属性值>`** 的语法来**对实例对象的属性进行增加/修改操作**：

- **属性名不存在**：**添加并赋值**
- **属性名存在**：**修改属性值**

```python
test = Test()

# 修改实例属性
test.name = 'test1'
print(test.name)  # test1

# 添加实例属性
test.age = 18
print(test.age)  # 18
```

> Python **为所有类实例默认提供了 `__dict__` 魔术属性**，该魔术属性会**将类实例对象中的所有自定义属性都打包成一个 `dict` 字典**

```python
print(test.__dict__)  # {'name': 'test1', 'age': 18}
```

###### del 删除

Python 规定需要**通过 `del` 关键字来删除实例对象的属性、方法**。

```python
test = Test()	

# 删除实例属性
del test.age
print(test.__dict__)  # {'name': 'test'}

# print(test.age) # 报错
```

##### 方法操作

直接通过 **`<实例对象>.<方法名>()`** 的语法来**调用实例对象的方法**：

```python
class Test:
    name: str = 'test'

    def __init__(self):
        pass

    # 实例方法
    def my_func(self):
        print('my_func')
        
test = Test()

# 调用方法
test.my_func() # my_func
```

### 方法

> [!NOTE]
>
> Python 规定在**类中使用 `def` 关键字定义的函数称为类的方法**，根据不同功能与特性，分为 5 种方法类型：
>
> - **`__<方法名>__(self,...)`魔术方法**
> - **`<方法名>(self,...)` 实例方法**
> - **`@classmethod <方法名>(cls,...)` 类方法**
> - **`@staticmethod <方法名>()` 静态方法**
> - **`@abstractmethod <方法名>()` 抽象方法**
>
> 关于 `self` 形参：
>
> > Python 没有 `this` 关键字，而是**为类中所有`__init__` 等魔术方法、实例方法默认传递了 `self` 形参**，它指向**当前调用该方法的实例对象本身**。
> >
> > 概念：基于 Python 的 **“显式优于隐式”** 规则，要**想使用 `self` 形参，必须在方法第一个参数中显式声明**。
> >
> > - 与其他语言的 `this` 关键字对比：
> >
> > | 特性         | Python (`self`)            | Java/C++ (`this`)                       |
> > | :----------- | :------------------------- | :-------------------------------------- |
> > | 是否显式声明 | ✅ 必须在方法参数中显式声明 | ❌ 隐式存在                              |
> > | 能否改名     | ✅ 可以，但约定用 `self`    | ❌ 关键字，不能改                        |
> > | 访问实例变量 | `self.var`                 | 可直接使用 `var`（同名时需 `this.var`） |
> > | 设计哲学     | 显式优于隐式               | 偏向隐式                                |
>
> 关于 `cls` 形参：用于**指向当前类**

#### 实例方法

##### 基本语法

实例方法指的是**由实例对象调用的方法**，只需在**类中像定义普通函数一样定义**即可。

```python
class <类名>:
    def <实例方法名>(self, <自定义形参>):
        pass
```

接收参数：

- **`self`**：表示**指向当前对象的实例**，相当于其他语言的`this`关键字，**必须显式声明！**
- **`自定义形参`**：**用来接收在调用实例方法时传入的一些实参值**

核心特征：

- ✅ 可以访问和修改**实例属性**（**`self.xxx`**）
- ✅ 调用权限：
  - **通过 `self` 参数**：
    - 可以**调用其他实例方法**

  - **通过 `self.__class__`属性 |  `<类名>.xxx`**：
    - 访问**类属性**、调用**类方法**
    - 调用**类中的静态工具方法**


> **调用实例方法时**：
>
> - `test.instance_func()` 隐式地把 `test` 实例传给了 `self`。
> - **能看到什么**：它能看到 `test` 里的所有小秘密（实例变量）。

```python
class Dog:
    id: id = id('Dog')

    # 构造方法（添加实例对象的一系列属性、执行实例方法）
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.hello()

    # 实例方法
    # 访问实例属性，调用实例方法
    def set_var(self, name, age):
        self.name = name
        self.age = age
        self.hello()

    # 访问类属性 id
    def get_class_id(self):
        return self.__class__.id or Dog.id

    def hello(self):
        print(f"我是一个狗，我的名字是{self.name}，我今年{self.age}岁了")


dog = Dog('旺财', 3) # 我是一个狗，我的名字是旺财，我今年3岁了
dog.set_var('小黑', 2) # 我是一个狗，我的名字是小黑，我今年2岁了

dog_id = dog.get_class_id()  # 140734944614848
```

##### self 参数隐式传递机制

Python 规定，实例对象调用实例方法时，**不需要显式传递 `self`**，Python 解释器**内部会自动将实例对象传递给实例方法的第一个参数 `self`** ，**指明当前操作的是哪个实例对象身上的属性、方法**。

```python
class Test:
    def get_name(self, args):
        # 为当前操作的实例添加/修改属性
        self.args = args
        print(args)


test = Test()

test.get_name('jack') # jack
```

<img src="media/方法 self 参数隐式传递机制.png" style="zoom: 50%;" />

##### 查找链机制

*类似于 JavaScript 的原型链查找*

Python 规定**类中定义的实例方法是存储在类身上的，不会放在实例对象身上**；所以**实例对象调用实例方法，其实调用的是类身上的同名方法**。

核心机制：**当类的实例对象调用某个具体实例方法时，会先查找实例对象自身有没有该方法，再去类身上去查找并调用**。

核心逻辑：

- **实例对象**：**只保存自己的 “私人数据”（属性），不保存方法**；

  ​	若实例对象**调用的方法与类中的方法同名**，则**优先使用实例对象自己的方法**。

- **类对象**：保存所有实例对象共用的属性、方法

<img src="media/类实例方法的查找链机制.png" style="zoom:50%;" />

核心原因：这是因为 Python 面向对象设计的核心节能方案 =>**方法复用**。

> 方法复用是 OOP 的精髓：**逻辑只写一遍，但可以处理无数个不同的数据个体。**
>
> 在内存中，方法（函数代码）只在类中存储一份，并不会为每个实例都复制一遍；这就是**方法复用**，也可以说是代码复用。

###### 示例

```python
class Person:
    # 实例方法
    def my_func(self):
        print('my_func')
     
	def get_name_age(self, name, age):
        print(f"{name}:{age}")


person = Person()

# person 会查找 Person 类中的 my_func 、get_name_age 方法并调用
person.my_func()  # 输出 my_func 
person.get_name_age('Jack', 18)  # 输出 Jack:18

print(person.__dict__)  # {}
print(Person.__dict__)  
# {'__module__': '__main__', 'my_func': <function Person.my_func at 0x0000011AC546D850>, 'get_name_age': <function Person.get_name_age at 0x0000011AC546E820>...


# 此时 person 没有 get_name_age 方法，会去 Person 类中查找并调用
person.get_name_age('Jack', 18)  # 输出 Person 类的 get_name_age 方法：Jack:18
print(person.__dict__)  # {}
```

###### 通过类调用实例方法

**当实例对象自身有同名方法时，如果只想用类的同名实例方法**；可以通过：

1. **`<类名X>`.`<实例方法>(<实例对象x>, <自定义参数>...)` 类名直接调用**
2. 使用 **`<实例对象x>.__class__.<实例方法>(<实例对象x>, <自定义参数>...)` 魔术属性找到它的所属类来调用**

但需要**显式传入实例对象**给实例方法的**第一个 `self`参数**，让 Python 内部明确知道**目前是操作的哪个实例对象身上的属性、方法**。

```python
class Person:
    # 实例方法
    def my_func(self):
        print('my_func')
     
	def get_name_age(self, name, age):
        print(f"{name}:{age}")

person = Person()

def fn(name, age):
    print(f"person 实例的 get_name_age 方法：{name}:{age}")

# 为 person 实例对象自己添加一个与类方法同名的 get_name_age 方法
person.get_name_age = fn

person.get_name_age('Jack', 18)  # 输出 person 实例的 get_name_age 方法：Jack:18
print(person.__dict__)  # {'get_name_age': <function fn at 0x000002825268F8A0>}

# 如果只想用 Person 类的 get_name_age 方法，那么需要由 Person 类来主动调用
Person.get_name_age(person, 'jack', 18)  # Person 类的 get_name_age 方法：Jack:18
# 注意：需将 person 实例传递给 `self` 参数，让 Python 内部知道目前是操作的哪个实例对象身上的属性、方法

# 或者通过 person.__class__ 找到它的所属类来调用
person.__class__.get_name_age(person, 'Jack', 18)  # Person 类的 get_name_age 方法：Jack:18
```

#### 类方法

##### 基本语法

Python 类中**使用 `@classmethod` 装饰器定义一个类方法**，**第一个参数为 `cls`（指向类本身）**。

```python
class <类名>:
    
    @classmethod
    def <类方法名>(cls, <自定义形参>): # cls => class 的缩写
        pass
```

接收参数：

- **`cls`**：表示**指向类本身**，**必须显式声明！**
- **`自定义形参`**：**用来接收在调用类方法时传入的一些实参值**

核心特征：

- ✅ 可以通过 **`cls` 参数**访问和修改**类属性**
- ✅ 调用权限：
  - **类方法**：可以**通过 `cls` 参数调用其他类方法**
  - **静态方法**：可以**通过 `cls` 参数调用类中的静态工具方法**
- ❌ 访问限制：
  - 类方法中**不能直接访问实例属性、方法（没有 `self`）**
- ✅ 可以被**子类继承和重写**（`cls` 指向调用它的类）

> 注：**实例对象也可以访问类方法，但是极其不推荐（容易造成歧义）**
>
> > [!NOTE]
> >
> > 这其实是 Python 为了**方便**提供的语法糖。
> >
> > - **实例方法**必须绑定在实例上，因为它需要处理每个对象不同的数据。
> > - **类方法**是属于“家族”的，身为“家族成员”的实例对象，理所应当拥有访问家族公共资源的权限。
> >
> > **实例对象调用类方法时**：
> >
> > - `test.class_func()` 虽然是用实例调用的，但 Python 会通过 `test.__class__` 找到 `Test` 类，并隐式地把 `Test` 类传给 `cls`。
> > - **能看到什么**：它只能看到类级别的共享信息，完全不知道具体是哪个 `test` 实例在调用它。

```python
class Animal:
    # 类属性
    name: str = 'animal'
    age: int = 0

    # 类方法（用于处理 “全局/类级别” 逻辑的方法）
    @classmethod
    def create(cls, name, age):  # cls: 指向 Animal 类
        # 访问类属性，并修改
        cls.name = name
        cls.age = age
        cls.hello()  # 我是dog，我今年3岁了
        
        # cls 指向类，所以可以直接使用 cls() 来创建一个 Animal 类的实例对象
        return cls()

    # 调用其他类方法
    @classmethod
    def hello(cls):
        print(f"我是{Animal.name}，我今年{Animal.age}岁了")
    

# 通过类方法（工厂方法）来创建一个 Animal 类的实例对象
animal = Animal.create('dog', 3)
print(Animal.__dict__) # { 'name':'dog', 'age': 3,... }
print(animal.__dict__) # {}

# 实例对象也可以访问类方法，但是不推荐
# animal.create('cat', 5)
```

##### cls 隐式传递机制

Python 规定，类调用类方法时，**不需要显式传递 `cls`**，Python 解释器**内部会自动将类传递给类方法的第一个参数 `cls`** 。

```python
class Animal:
    
    @classmethod
    def create(cls, name, age):
		pass

Animal.create('dog', 3)
```

<img src="media/类方法的cls参数隐式传递机制.png" style="zoom:50%;" />

##### 与实例方法的对比

| **特性**         | **实例方法 (Instance Method)**     | **类方法 (Class Method)**                |
| ---------------- | ---------------------------------- | ---------------------------------------- |
| **定义装饰器**   | 无                                 | **使用 `@classmethod`**                  |
| **首个参数约定** | `self` (指向**实例对象**本身)      | `cls` (指向**类对象**本身)               |
| **访问权限**     | 既能访问实例属性，也能访问类属性   | **只能**访问类属性，无法直接访问实例属性 |
| **主要用途**     | 操作特定对象的数据（如：修改姓名） | 操作类级别的数据（如：计数器、工厂模式） |

#### 静态方法（工具方法）

##### 基本语法

Python 类中**使用 `@staticmethod` 装饰器定义一个静态方法**，**它只接收自定义参数**。

```python
class <类名>:
    
    @staticmethod
    def <静态方法名>(<自定义形参>):
        pass
```

接收参数：

- **`自定义形参`**：**用来接收在调用静态方法时传入的一些实参值**

核心特征：

- ✅通过类名调用，通常用于定义**关于类的工具方法**，常用于一些辅助性操作
- ✅ 调用权限：
  - **类方法**：可**通过 `cls` 参数调用**静态工具方法
  - **实例方法**：可**通过 `self.__class__` 调用**静态工具方法
- ❌ 访问限制：
  - 静态方法中**不能访问实例属性、方法（没有 `self` 参数）**
  - 静态方法中**不能访问类属性、方法（没有 `cls` 参数，除非显式使用类名 `MathUtils.pi`）**

> 注：**实例对象也可以访问静态方法，但是极其不推荐（容易造成歧义）**
>
> > [!NOTE]
> >
> > 这其实是 Python 为了**方便**提供的语法糖。
> >
> > - **实例方法**必须绑定在实例上，因为它需要处理每个对象不同的数据。
> > - **静态方法**是属于“家族”的，身为“家族成员”的实例对象，理所应当拥有访问家族公共资源的权限。
> >
> > **实例对象调用静态方法时**：
> >
> > - `test.class_func()` 虽然是用实例调用的，但 Python 会通过 `test.__class__` 找到 `Test` 类，并调用静态方法。

```python
class Person:
    id_card: str = ''
    name: str = ''
    age: int = 0

    def __init__(self, id_card, name, age):
        self.id_card = id_card
        self.name = name
        self.age = age

    # 类方法中调用静态工具方法
    @classmethod
    def get_idcard(cls):
        return cls.mask_idcard(cls.id_card)

    # 实例方法中调用静态工具方法
    def get_info(self):
        id_card = self.__class__.mask_idcard(self.id_card)
        return f"身份证号：{id_card}，姓名：{self.name}，年龄：{self.age}"

    # 静态方法：工具方法，常用于一些辅助性操作
    @staticmethod
    def mask_idcard(id_card):
        return id_card[:6] + '****' + id_card[-4:]


person = Person(name='张三', id_card='123456789012345678', age=18)
print(person.get_info())    # 身份证号：123456********345678，姓名：张三，年龄：18

# 类名直接调用，作为外部工具
print(Person.mask_idcard(person.__dict__['id_card']))  # 123456********345678
```

#### 魔术方法（钩子函数）

> [!IMPORTANT]
>
> Python 的魔术方法是 **Python 内置的“生命周期钩子”或“事件监听器”**。可以理解为 JavaScript 的**钩子系统**。
>
> > 在 JavaScript（尤其是 Vue, React 或 Node.js）中，钩子（Hooks）允许你在特定的时刻（如组件挂载、数据更新）插入自己的逻辑。Python 的魔术方法也是如此，只不过它拦截的是**语言层面的底层操作**。
>
> Python 的魔术方法通常也不需要手动调用，而是由 **Python 解释器在特定事件发生时自动触发**。
>
> | **概念**        | **JavaScript (以 Vue/React 为例)** | **Python 魔术方法**                |
> | --------------- | ---------------------------------- | ---------------------------------- |
> | **创建/初始化** | `constructor` / `onMounted`        | `__new__` / `__init__`             |
> | **销毁**        | `onUnmounted` / `useEffect` return | `__del__`                          |
> | **访问属性**    | `Proxy` 中的 `get` 陷阱            | `__getattr__` / `__getattribute__` |
> | **类型转换**    | `toString()`                       | `__str__` / `__int__`              |
>
> 总结：可以完全把魔术方法看作是 **Python 给开发者留出的“内部接口”**。通过这些接口，可以定义对象在面对加减乘除、循环、打印、上下文切换时的行为。
>
> 这种“钩子”系统的存在，是为了实现 **协议（Protocols）**：
>
> - 如果你实现了 `__iter__`，你就实现了“迭代协议”，你的对象就可以用在 `for...in` 循环里。
> - 如果你实现了 `__call__`，你的对象就变成了“可调用协议”，你可以像调用函数一样调用它：`my_obj()`。
>
> 这让你编写的自定义类可以完美地伪装成 Python 原生对象，这种灵活性正是 Python 强大扩展性的来源。

##### 基本概念

> [!NOTE]
>
> Python 的魔术方法是**隐式调用的协议钩子**，在其他语言中往往对应**运算符重载、构造/析构、内置类型转换、属性拦截器、上下文管理器、可调用对象**等分散在不同语法/接口中的特性。Python 通过统一的**“双下划线方法”机制**将它们集中表达。

魔术方法是 Python 中**以双下划线 `__` 开头和结尾的特殊钩子方法**，用于**自定义类的行为**。

- 核心机制：它们**不需要手动调用**，而是在**特定阶段、特定操作（实例初始化、比较值、`getter/setter`拦截...）**时**自动触发**。

- 核心价值：**可以重写类的魔术方法，使得它们的实例对象在特定阶段、特定操作时能添加一些我们的自定义处理**。

> 注：魔术方法是 Python 语言**预先定义好**的，有**固定名称**（如 `__init__`、`__add__`... 等），**不能创建新的魔术方法名称**。
>
> ```python
> # ❌ 错误尝试：自定义魔术方法名
> class Dragon:
>  def __fly__(self):      # Python 不会自动调用这个
>      print("Flying!")
> 
>  def __breathe_fire__(self):
>      print("Fire!")
> 
> d = Dragon()
> # 这些不会在特定操作时自动触发，需要手动调用：
> d.__fly__()  # 必须显式调用，和普通方法没区别
> ```

根据**不同数据类型与功能应用**，魔术方法有如下分类：

参数说明：

- **`cls`：指向当前类**
- **`self`：指向当前运行时隐式传入的实例对象**
- **`...`：其他参数**

| 类别                   | 方法                                       | 触发时机                             |
| :--------------------- | :----------------------------------------- | :----------------------------------- |
| **实例创建/销毁**      | `__new__(cls,...)`                         | 创建实例前                           |
|                        | `__init__(self,...)`                       | 初始化实例                           |
|                        | `__del__(self,...)`                        | 销毁实例                             |
| **类属性的访问控制**   | `__get__(self,...)`                        | 访问类属性                           |
|                        | `__set__(self,...)`                        | 修改类属性                           |
| **所有属性的访问控制** | `__setattr__(self,name,value)`             | **设置任何属性值时**触发             |
|                        | `__getattr__(self,name)`                   | 访问实例对象中**不存在的属性时**触发 |
|                        | `__getattribute__(self,name)`              | **访问任何属性时触发**               |
|                        | `__delattr__(self,name)`                   | **删除属性时**触发                   |
| **字符串**             | `__str__(self,...)`                        | `print()`, `str()`                   |
|                        | `__repr__(self,...)`                       | `repr()`, 交互式环境                 |
|                        | `__format__(self,...)`                     | `format()`, f-string                 |
| **容器 **              | `__len__(self,...)`                        | `len()`                              |
|                        | `__getitem__(self,...)`                    | `obj[key]`                           |
|                        | `__setitem__(self,...)`                    | `obj[key]=value`                     |
|                        | `__delitem__(self,...)`                    | `del obj[key]`                       |
|                        | `__contains__(self,...)`                   | `item in obj`                        |
| **算术**               | `__add__(self,...)`                        | `+`                                  |
|                        | `__sub__(self,...)`                        | `-`                                  |
|                        | `__mul__(self,...)`                        | `*`                                  |
|                        | `__truediv__(self,...)`                    | `/`                                  |
|                        | `__floordiv__(self,...)`                   | `//`                                 |
|                        | `__mod__(self,...)`                        | `%`                                  |
|                        | `__pow__(self,...)`                        | `**`                                 |
| **比较**               | `__eq__(self,...)`                         | `==`                                 |
|                        | `__ne__(self,...)`                         | `!=`                                 |
|                        | `__lt__(self,...)`                         | `<`                                  |
|                        | `__le__(self,...)`                         | `<=`                                 |
|                        | `__gt__(self,...)`                         | `>`                                  |
|                        | `__ge__(self,...)`                         | `>=`                                 |
| **其他**               | `__call__(self,...)`                       | `obj()`                              |
|                        | `__iter__(self,...)`                       | `iter()`, 循环                       |
|                        | `__next__(self,...)`                       | `next()`                             |
|                        | `__enter__(self,...)`/`__exit__(self,...)` | `with` 语句                          |
|                        | `__hash__(self,...)`                       | `hash()`                             |
|                        | `__bool__(self,...)`                       | `bool()`, `if obj`                   |

##### 阶段钩子

###### 实例对象生命周期

这些方法控制对象的 **创建、初始化和销毁 **阶段（不建议重写）：

- **`__new__(cls, ...)`**： 真正创建实例的方法。它是一个静态方法，返回类的一个新实例
- **`__init__(self, ...)`**： 实例初始化方法。当对象被创建后，用来设置初始属性
- **`__del__(self)`**： 析构方法。当对象被垃圾回收时调用

```python
class Person:
    def __new__(cls):
        print('创建实例对象，在__init__之前执行...')
        return super().__new__(cls)

    def __init__(self, name, age):
        self.name = name
        self.age = age
        print('初始化实例中...，添加一些属性')

    def __del__(self):
        print('程序结束，对象被销毁...')

p = Person("Alice", 25)
# 输出: 
# 创建实例对象，在__init__之前执行...
# 初始化实例中...，添加一些属性
# 程序结束，对象被销毁...
```

###### 字符串表示与调试

通过重写以下方法，可以让**实例对象在 `print()` 或 `str`、`f""` 字符串格式化输出时** 更有可读性：

| **方法**         | **触发场景**                                   | **用途**                                       |
| ---------------- | ---------------------------------------------- | ---------------------------------------------- |
| **`__str__`**    | **`print(obj)` 或 `str(obj)`**                 | 自定义对象内容的输出                           |
| **`__repr__`**   | **直接输入变量名或 `repr(obj)`**               | 自定义面向**开发者**的正式描述（通常用于调试） |
| **`__format__`** | **使用 `f""` / `format()` 方法输出对象内容时** | 自定义对象内容的格式化输出                     |

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    # __str__: print/str() 调用
    def __str__(self):
        return f"《{self.title}》 by {self.author}"

    # __repr__: repr()/调试时调用
    def __repr__(self):
        return f"Book('{self.title}', '{self.author}', {self.pages})"

    # __format__: 格式化字符串时调用
    def __format__(self, format_spec):
        if format_spec == "short":
            return self.title
        return str(self)


book = Book("Python编程", "小张", 300)
print(str(book))      # 《Python编程》 by 小张
print(repr(book))     # Book('Python编程', '小张', 300)
print(f"{book:short}")  # Python编程
```

###### 运算符重载

通过重写以下方法，**外部可以直接用 `+`, `-`, `*`, `==` 等运算符操作对象**。

比较操作：

- **`__eq__(self, other)`**：**对象通过 `==` 比较内容时**触发
- **`__lt__(self, other)`**：**对象通过 `<` 比较内容时**触发
- **`__gt__(self, other)`**：**对象通过 `>` 比较内容时**触发

数值运算：

- **`__add__(self, other)`**：**对象通过 `+` 计算/合并内容时**触发
- **`__sub__(self, other)`**：**对象通过 `-` 计算数值时**触发
- **`__mul__(self, other)`**：**对象通过 `*` 计算/重复生成内容时**触发

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # 算术运算符
    def __add__(self, other):      # self + other
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):      # self - other
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):     # self * scalar
        return Vector(self.x * scalar, self.y * scalar)
    
    def __rmul__(self, scalar):    # scalar * self（右乘）
        return self.__mul__(scalar)
    
    def __truediv__(self, scalar): # self / scalar
        return Vector(self.x / scalar, self.y / scalar)
    
    def __neg__(self):             # -self
        return Vector(-self.x, -self.y)
    
    def __abs__(self):             # abs(self)
        return (self.x ** 2 + self.y ** 2) ** 0.5
    
    # 比较运算符
    def __eq__(self, other):       # self == other
        return self.x == other.x and self.y == other.y
    
    def __lt__(self, other):       # self < other（可用于排序）
        return abs(self) < abs(other)
    
    def __hash__(self):            # hash(self)（用于字典键）
        return hash((self.x, self.y))

v1 = Vector(2, 3)
v2 = Vector(3, 4)
print(v1 + v2)      # 需要定义 __repr__ 才能看到
print(2 * v1)       # 右乘：调用 __rmul__
print(abs(v1))      # 3.605...
```

###### 容器/序列协议

通过重写以下方法，可以**让自定义类像 `list` 或 `dict` 一样可以被索引、切片或迭代**：

- **`__len__(self)`**：定义 `len()` 的行为
- **`__getitem__(self, key)`**：定义获取元素的操作（如 `obj[key]`）
- **`__setitem__(self, key, value)`**：定义设置元素的操作
- **`__iter__(self)`**：返回一个迭代器，让对象支持 `for item in obj`

```python
class Playlist:
    def __init__(self):
        self._songs = []
    
    # __len__: 支持 len()
    def __len__(self):
        return len(self._songs)
    
    # __getitem__: 支持索引访问、切片、迭代
    def __getitem__(self, index):
        return self._songs[index]
    
    # __setitem__: 支持索引赋值
    def __setitem__(self, index, value):
        self._songs[index] = value
    
    # __delitem__: 支持 del
    def __delitem__(self, index):
        del self._songs[index]
    
    # __contains__: 支持 in 操作
    def __contains__(self, item):
        return item in self._songs
    
    # __iter__: 返回迭代器（通常有 __getitem__ 就够）
    def __iter__(self):
        return iter(self._songs)
    
    # __reversed__: 支持 reversed()
    def __reversed__(self):
        return reversed(self._songs)

playlist = Playlist()
playlist._songs = ["Song A", "Song B", "Song C"]
print(len(playlist))    # 3
print(playlist[0])      # Song A
print("Song B" in playlist)  # True
for song in playlist:
    print(song)
```

###### 所有属性访问控制

通过重写以下方法，可以**自定义控制外部操作实例对象中的实例属性、类属性时的行为**：

> 类似于 JavaScript 的 `getter/setter` 拦截器

- **`__setattr__(self,name,value)`**：**设置任何属性时**触发
- **`__getattr__(self,name)`**：**访问实例对象中不存在的属性时**触发
- **`__getattribute__(self,name)`**：**访问任何属性时触发**
- **`__delattr__(self,name)`**：**删除属性时**触发

```python
class SafeDict:
    def __init__(self):
        self._data = {}
    
    # __getattr__: 访问不存在的属性时调用
    def __getattr__(self, name):
        print(f"Getting {name}")
        return self._data.get(name, None)
    
    # __setattr__: 设置任何属性时调用（谨慎使用，容易无限递归）
    def __setattr__(self, name, value):
        print(f"Setting {name} = {value}")
        if name == "_data":
            super().__setattr__(name, value)
        else:
            self._data[name] = value
    
    # __getattribute__: 访问任何属性时调用（更底层，慎用）
    def __getattribute__(self, name):
        print(f"Accessing {name}")
        return super().__getattribute__(name)
    
    # __delattr__: 删除属性时调用
    def __delattr__(self, name):
        print(f"Deleting {name}")
        del self._data[name]

obj = SafeDict()
obj.name = "Alice"   # Setting name = Alice
print(obj.name)      # Accessing name -> Getting name -> Alice
```

###### 可调用对象

通过重写以下方法，**让实例对象可以像函数一样被调用**：

- **`__call__(self,...)`**：**当对象被作为函数被调用时**触发

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor
    
    # __call__: 使实例可以像函数一样被调用
    def __call__(self, x):
        return x * self.factor

double = Multiplier(2)
triple = Multiplier(3)

print(double(5))   # 10（像调用函数一样）
print(triple(5))   # 15
print(callable(double))  # True

# 装饰器就是一个常见应用
class TraceDecorator:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        print(f"Calling {self.func.__name__}")
        return self.func(*args, **kwargs)

@TraceDecorator
def greet(name):
    return f"Hello {name}"

print(greet("Alice"))  # 先打印 "Calling greet"
```

###### 上下文管理器（with语句）

用于实现 `with` 语句，常用于**自动关闭资源（如文件、数据库连接）**。

- **`__enter__(self)`**: 进入 `with` 代码块时的准备工作。
- **`__exit__(self, exc_type, exc_val, exc_tb)`**: 退出代码块时的善后工作。

```python
class ManagedFile:
    def __init__(self, filename, mode='r'):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    # __enter__: with 块开始时调用
    def __enter__(self):
        print(f"Opening {self.filename}")
        self.file = open(self.filename, self.mode)
        return self.file
    
    # __exit__: with 块结束时调用（无论是否异常）
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"Closing {self.filename}")
        if self.file:
            self.file.close()
        if exc_type:
            print(f"Exception: {exc_val}")
        return False  # True 表示抑制异常

# 使用示例
with ManagedFile("test.txt", "w") as f:
    f.write("Hello World")
# Opening test.txt
# Closing test.txt
```

###### 数学运算相关

```python
class Complex:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
    
    def __repr__(self):
        return f"{self.real} + {self.imag}i"
    
    # 整数转二进制、进制转换
    def __int__(self):
        return int(self.real)
    
    def __float__(self):
        return float(self.real)
    
    def __complex__(self):
        return complex(self.real, self.imag)
    
    def __bool__(self):
        return self.real != 0 or self.imag != 0
    
    def __round__(self, n=0):
        return Complex(round(self.real, n), round(self.imag, n))

c = Complex(3.7, 4.2)
print(int(c))    # 3
print(bool(c))   # True
print(round(c))  # 4 + 4i
```

> 魔术方法是 Python **鸭子类型（Duck Typing）** 的核心。
>
> 通过实现特定的魔术方法，自定义对象可以“伪装”成任何数据类型，从而完美融入 Python 的生态系统。

#### 总结对比

| 特性             | 实例方法              | 类方法         | 静态方法        | 抽象方法          | 魔术方法               |
| :--------------- | :-------------------- | :------------- | :-------------- | :---------------- | :--------------------- |
| **装饰器**       | 无                    | `@classmethod` | `@staticmethod` | `@abstractmethod` | 无（`__xx__`命名约定） |
| **第一个参数**   | `self`                | `cls`          | 无              | `self` 或 `cls`   | `self`                 |
| **访问实例属性** | ✅                     | ❌              | ❌               | ✅（实例方法）     | ✅                      |
| **访问类属性**   | ✅（`self.__class__`） | ✅（`cls.xxx`） | ❌（需类名）     | ✅                 | ✅                      |
| **调用方式**     | 实例.方法()           | 类.方法()      | 类.方法()       | 不能直接调用      | 自动触发               |
| **可以被继承**   | ✅                     | ✅              | ✅               | ✅ 必须被实现      | ✅                      |
| **使用场景**     | 大多数操作            | 工厂方法、统计 | 工具函数        | 定义接口          | 语言集成               |

### 属性

> [!NOTE]
>
> Python 中的属性可以分为 3 大类：**实例属性**、**类属性**、**魔术属性**，再根据访问控制可以进一步细分：
>
> - **实例属性**
> - **类属性**
> - **`__xx__` 魔术属性**

#### 实例属性

实例属性是属于**每个实例对象独有的**数据，**存储在实例对象的 `__dict__` 魔术属性中**。

实例属性的定义分为两种形式：

- 初始化实例属性：

  ```python
  class Person:
      def __init__(self, name, age):
          self.name = name      # 实例属性
          self.age = age        # 实例属性
          self.email = None     # 实例属性（可先设为 None）
  ```

- 动态添加的实例属性：

  ```python
  person = Person()
  
  person.gender = True
  person.address = 'hunan'
  ```

注：**每个创建的实例都持有各自独立的私有属性数据，互不影响**。

```python
p1 = Person('jack', 18)
p1.gender = True
p1.address = 'hunan'

p2 = Person('tom', 20)
p2.email = 'tom@163.com'

print(p1.__dict__)  # {'name': 'jack', 'age': 18, 'gender': True, 'address': 'hunan'}
print(p2.__dict__)  # {'name': 'tom', 'age': 20, 'email': 'tom@163.com'}
```

##### 向上查找机制

实例对象访问实例属性的**向上查找**顺序：

- 优先级：**先看实例对象自身的 `__dict__` 有没有 > 通过 `__class__` 属性向上查找到实现类的 `__dict__`有没有 > 继续向上查找 > ... > 顶层 object 基类**

```python
class A:
    value = 'A'

    def __init__(self):
        self.value = 'a'


class B(A):
    value = 'B'


class C(B):
    value = 'C'


c = A()
print(c.value)  # a

# 查找顺序：
# 1. 实例 c 的 __dict__ → 找到 "a"，返回
# 如果实例没有：
# 2. C 类的 __dict__
# 3. B 类的 __dict__
# 4. A 类的 __dict__
# 5. object 类的 __dict__
# 6. 抛出 AttributeError
```

<img src="media/实例对象属性的向上查找机制.png" style="zoom:50%;" />

#### 类属性

类属性属于**类本身**，是所有实例**共享**的**公共数据**，**存储在类的 `__dict__` 魔术属性中**。

定义方式：

```python
class <类名>:
    <类属性名> = <值>
```

核心特点：

- **保存在类身上**，可以**通过 `<类名>.<类属性名>` 和 `<实例对象>.<类属性名>` 访问**
- **修改类属性会影响所有实例对象**

示例：

```python
class Book:
    name: str = 'Python'
    price: float = 34.6


b1 = Book()
b2 = Book()

print(Book.__dict__) # { 'name':'Python', 'price': 34.6,... }
print(b1.__dict__) # {} 
print(b2.__dict__) # {} 

# 可以通过类名访问
print(Book.name) # Python
# 也可以通过实例对象访问
print(b1.name) # Python
print(b2.price) # 34.6

# 修改类属性会影响其他实例对象
Book.price = 10

print(b1.price) # 10
print(b2.price) # 10
```

##### 关于实例对象访问类属性

- **优先级：同名实例属性 > 同名类属性**
- 虽然**实例对象可以访问类属性，但是并不能通过 `<实例对象>.<属性名> = <值>`的形式对类属性进行修改，因为这是给实例对象身上添加一个属性**。

```python
b1.name = 'JavaScript' # 这个其实是给 b1 实例对象自身添加一个 name 属性

print(Book.__dict__['name']) # Python
print(b1.__dict__) # { 'name': 'JavaSCript' }

# 之后再访问 b1.name 则是拿取的 b1 实例对象身上的 name 属性
print(b1.name)  # JavaScript

# 如果想通过 b1 找到它所属类的 name 属性，则需通过 __class__.__dict__ 来访问
print(b1.__class__.__dict__['name']) # Python
```

##### 实例属性 vs 类属性

```python
class Demo:
    class_attr = "我是类属性"      # 类属性
    
    def __init__(self):
        self.instance_attr = "我是实例属性"  # 实例属性

d1 = Demo()
d2 = Demo()

# 存储位置不同
print(d1.__dict__)      # {'instance_attr': '我是实例属性'}
print(Demo.__dict__)    # 包含 'class_attr' 但不包含 'instance_attr'

# 访问优先级：实例属性 > 类属性
d1.instance_attr = "修改实例属性"
print(d1.instance_attr)  # 修改实例属性（实例属性）
print(d2.instance_attr)  # 我是实例属性（不影响其他实例）

# 通过实例修改类属性？实际上是创建实例属性
d1.class_attr = "看起来像修改类属性"
print(d1.class_attr)      # 看起来像修改类属性（实例属性）
print(d2.class_attr)      # 我是类属性（类属性未变）
print(Demo.class_attr)    # 我是类属性（类属性未变）
```

##### 向上查找机制

###### 类查找

当一个子类继承了多个类时：

- 查找某个类属性：**优先查找自身有没有 > 通过 `__class__` 属性向上查找父类有没有 > 继续向上查找顶层基类有没有...**.，在这个过程中，**有则取，无则继续向上查找，直到顶层基类**为止。

```python
class GrandParent:
    value = 'grand_parent'

class Parent(GrandParent):
    value = 'parent'

class Child(Parent):
    pass

print(Child.value)  # parent
```

###### 实例对象查找

当实例对象要访问某个类属性时：

- 优先级：**自身的同名实例属性 > 通过 `__class__` 属性找到实现类，去查看实现类的类属性 > 父类中的类属性 > ... > 顶层基类**

```python
class GrandParent:
    value = 'grand_parent'


class Parent(GrandParent):
    # value = 'parent'
    pass


class Child(Parent):
    pass


child = Child()
# 先看 child 实例对象自身有没有，没有再看 Parent 类，再看 GrandParent 父类，最后 object 基类
print(child.value)  # grand_parent

# 往实例对象添加 value 属性
child.value = 'child'
print(child.value)  # child
```

<img src="media/实例对象访问类属性的向上查找机制.png" style="zoom:50%;" />

#### 魔术属性

> 如果说**魔术方法**是对象的“动作钩子”（行为），那么**魔术属性**（Magic Attributes）就是对象的“元数据标签”。

魔术属性是 Python 中**以双下划线开头和结尾**的特殊属性，它们由 Python 解释器**自动管理**，用于提供对象或类的元信息。

与魔术方法不同，魔术属性**不需要调用**，直接访问即可。

核心概念：魔术属性是**任何对象都内置的 “元数据标签” （类、函数、模块最终都是对象，万物皆对象）**；通常由 Python 自动维护，记录了关于**类、实例对象或模块**的**结构信息**。

根据不同功能应用，常用魔术属性分为 3 类：

| 魔术属性            | 所属对象     | 说明             |
| :------------------ | :----------- | :--------------- |
| `__dict__` (最常用) | 实例/类/模块 | 属性的字典       |
| `__class__`         | 实例         | 对象所属的类     |
| `__name__`          | 类/函数/模块 | 名称字符串       |
| `__module__`        | 类/函数      | 定义所在的模块名 |
| `__bases__`         | 类           | 直接父类元组     |
| `__mro__`           | 类           | 方法解析顺序元组 |
| `__doc__`           | 类/函数/模块 | 文档字符串       |
| `__slots__`         | 类           | 允许的属性名元组 |
| `__file__`          | 模块         | 模块文件路径     |
| `__package__`       | 模块         | 包名             |
| `__annotations__`   | 函数/类      | 类型注解         |
| `__defaults__`      | 函数         | 默认参数值       |
| `__code__`          | 函数         | 编译后的代码对象 |
| `__closure__`       | 函数         | 闭包单元格       |
| `__dict__`          | 模块         | 模块的全局字典   |

##### 类与实例

Python 为类与实例对象提供了如下魔术属性：

| **魔术属性**     | **含义与用途**                                               |
| ---------------- | ------------------------------------------------------------ |
| **`__dict__`**   | **最常用**。**存储对象属性的字典。修改它会直接影响实例**。   |
| **`__class__`**  | **指向对象所属的类**。通过它可以获取类名或再次实例化。       |
| **`__name__`**   | **类、函数或模块的名字**。在模块中常用作入口判断。           |
| **`__doc__`**    | 存储类或函数的**文档字符串**（Docstring）。                  |
| **`__module__`** | 该**对象所在的模块名**。                                     |
| **`__slots__`**  | 用于内存优化。如果你有数百万个小对象，可以用 `__slots__` 告诉 Python：“我只会有这几个属性，别给我准备字典了”。 |

###### \__dict__ 类与实例对象的自身属性

默认情况下，Python 会**为类与它的每个实例创建 `__dict__` 字典**，绝大多数**类的属性与它的实例对象自身属性都存在这个字典里**。

```python
class Animal:
    """这是一个动物类的文档"""

    species = "Unknown"  # 类属性

    def __init__(self, name, age):
        self.name = name      # 实例属性
        self.age = age

    def speak(self):
        pass


class Dog(Animal):
    def __init__(self, name, age, breed):
        super().__init__(name, age)
        self.breed = breed


dog = Dog('旺财', 3, '金毛')
print(dog.__dict__)   # {'name': '旺财', 'age': 3, 'breed': '金毛'}
print(dog.__dict__['name'])   # '旺财'

# 甚至可以直接操作它（虽然不推荐）
dog.__dict__['name'] = '小黑'
print(dog.name)       # 小黑
```

###### \__slots__ 选择性存储属性

默认情况下，Python 为每个实例创建 `__dict__`，且实例的所有属性都会存储，这很灵活但费内存。

描述：当**一个实例对象的属性过多时，可通过 `__slots__` 属性来限制实例对象所对外产出的字典，用一些极少的属性来替代 `__dict__`**，一次来节省内存。

```python
class Point:
    __slots__ = ('x', 'y')  # 固定属性，不再有 __dict__

p = Point()
p.x = 10
# p.z = 20  # 这会报错，因为 z 不在 slots 里
```

###### 其他

```python
# __class__: 对象所属的类
print(dog.__class__)          # <class '__main__.Dog'>
print(dog.__class__.__name__) # 'Dog'

# __name__: 类/函数/模块的名称（字符串）
print(Dog.__name__)    # 'Dog'

# __module__: 类/函数/实例所属的模块名
print(Dog.__module__)  # '__main__'

# __bases__: 类的直接父类（元组）
print(Dog.__bases__)   # (<class '__main__.Animal'>,)

# __base__: 类的第一个直接父类
print(Dog.__base__)    # <class '__main__.Animal'>

# __mro__: 方法解析顺序（Method Resolution Order）
print(Dog.__mro__)     # (<class '__main__.Dog'>, <class '__main__.Animal'>, <class 'object'>)

# __subclasses__(): 返回类的直接子类（方法，不是属性）
print(Animal.__subclasses__())  # [<class '__main__.Dog'>]

# __doc__: 文档字符串
print(Animal.__doc__)   # '这是一个动物类的文档'

# __annotations__: 类型注解（Python 3.10+）
def func(x: int, y: str) -> bool:
    pass
print(func.__annotations__)  # {'x': <class 'int'>, 'y': <class 'str'>, 'return': <class 'bool'>}
```

##### 函数

```python
def calculate(x, y=10, *args, **kwargs):
    """计算函数示例"""
    return x + y

# __code__: 编译后的代码对象
print(calculate.__code__)
print(calculate.__code__.co_varnames)  # 局部变量名：('x', 'y', 'args', 'kwargs')

# __defaults__: 默认参数值
print(calculate.__defaults__)  # (10,)

# __kwdefaults__: 关键字默认参数（Python 3）
print(calculate.__kwdefaults__)  # None

# __closure__: 闭包信息
def outer():
    msg = "hello"
    def inner():
        return msg
    return inner
closure_func = outer()
print(closure_func.__closure__)      # 闭包单元格对象
print(closure_func.__closure__[0].cell_contents)  # 'hello'

# 其他函数属性
print(calculate.__name__)      # 'calculate'
print(calculate.__module__)    # '__main__'
print(calculate.__doc__)       # '计算函数示例'
print(calculate.__annotations__)  # {}
```

##### 模块

```python
# 假设这个文件是 example.py

print(__name__)        # '__main__'（直接运行）或 'example'（被导入）
print(__file__)        # 当前文件的路径
print(__package__)     # 包名（如果有）
print(__doc__)         # 模块的文档字符串

# 示例：判断是否直接运行
if __name__ == "__main__":
    print("This script is run directly")
else:
    print("This script is imported as a module")
```

### 数据分层管理机制与方法复用

#### 数据分层管理机制

Python 将**数据存储在不同的命名空间（类变量 vs 实例变量）**中，以确保代码**既能共享资源，又能保持个体差异**。

- **类属性：** 定义在类中、方法之外。它们由所有实例**共享**。适用于**存储常量或所有实例通用的属性**
- **实例变量：** 存储在 `self` 中。它们是每个实例**独有**的。适用于**存储每个实例个体的独立特征**。

#### 为什么需要 self？

**核心原因**：同一个类会创建多个实例，**`self` 形参让方法知道「当前操作的是哪个实例的数据」**。

核心价值：`self` 是连接**抽象定义**与**具体数据**的**纽带关联**。

**如果没有 `self`：**

- 类不知道该去修改哪一个实例的数据。
- 方法将无法访问实例的属性，因为它没有指向特定内存地址的指针。

<img src="media/数据分层管理与self关联机制结构图.png" style="zoom:50%;" />

##### 为什么不能省略？

1. **显式优于隐式：** Python 的设计哲学要求明确指出变量的来源。如果不写 `self.name`，Python 会在局部作用域或全局作用域寻找 `name`，而不会去实例的口袋里翻找。

   不过，在代码编写中，`self` 会隐式传递给方法

   <img src="media/self隐式传参机制的理解.png" style="zoom:50%;" />

2. **区分属性与局部变量：** 在方法内部，`self.x` 代表实例属性，而 `x` 仅仅是一个临时变量。

| 没有 self 的问题是…        | self 的解决方案                            |
| :------------------------- | :----------------------------------------- |
| 方法无法区分操作哪个实例   | `self` 指向当前实例                        |
| 每个实例的数据需要独立存储 | `self.attr` 绑定到特定实例                 |
| 方法代码只需一份           | 所有实例共享方法，通过 `self` 访问各自数据 |

#### 方法复用

> 为什么实例方法都存在类中，而不是每个实例存一份？
>
> 核心原因：**逻辑共享，数据隔离**，一切为了代码复用。

Python 为了节省内存性能，设计了一套存储方案，即**方法复用**。

方法复用是 OOP 的精髓：**逻辑只写一遍，但可以处理无数个不同的数据个体。**

在内存中，方法（函数代码）只在类中存储一份，并不会为每个实例都复制一遍。这就是**方法复用**。

核心价值：

- **逻辑统一：** 所有的实例都共享类中定义的方法代码，节省内存。

- **行为一致：** 只要是同一个类的实例，它们处理数据的方式就是相同的，确保了系统的可预测性。

#### 三者串联逻辑

核心定义：**类 (逻辑中心)** + **实例 (数据载体)** + **`self` (连接器)** = **高效复用**

1. **类**定义了模板（包含**复用的方法**和**类变量**）
2. **实例**根据模板创建，并通过 `__init__` 开辟独立的内存空间存放**实例变量**
3. **`self`** 在调用方法时，将**实例的引用**传递给方法，使得一套公共的代码能够精准地操作不同实例的私有数据

这种机制完美平衡了“代码冗余的最小化”（方法复用）与“数据操作的精确化”（`self` + 实例变量）。

#### 属性查找链机制

在 Python 中，属性查找链（Attribute Lookup Chain）是面向对象的核心。当你访问一个对象的属性时，Python 会按照一个严谨的顺序**“从下往上”搜索，直到找到为止**。

##### 查找链

1. **实例查找 (Instance)**

   - 首先检查**对象自己的 `__dict__`（实例属性）**。

   - 例如：`r1.name`，如果实例里存了 "Gort"，查找结束。


2. **类查找 (Class)**

   - 如果在实例中没找到，**顺着 `__class__` 指针去所属类的 `__dict__` 里找**。

   - 例如：`r1.say_hi`，通常方法都存在这里。


3. **父类查找 (MRO/Superclass)**

   - 如果在当前类也没找到，就顺着 **MRO (方法解析顺序)** 往上爬，去查找所有的父类。

   - 这就是“继承”的本质。


4. **终点站 (Object)**
   - 如果一路上到 `object` 类（所有类的祖先）都还没找到，Python 就会抛出 `AttributeError` 报错。


其中，查找链中最核心的就是 **`__class__` 魔术属性**，它用来**指向当前实例对象的所属类**。

```python
class Person:
    person_id = "person_id"

    def __init__(self, name, age):
        self.name = name
        self.age = age


p1 = Person("张三", 18)
# 1. 实例对象身上查找，通过 __dict__ 属性集
print(p1.__dict__)  # {'name': '张三', 'age': 18}

# 2. 通过 实例对象.__class__ 找到它的所属类，并在所属类对象身上查找，通过 __dict__ 属性集
print(p1.__class__)  # <class '__main__.Person'>
print(Person.__dict__)  # {'person_id': 'person_id'} 找到了！
```

##### \__class__ 实例所属类指针

在 Python 的底层设计中，`__class__` 是连接“实例”与“类”的**灵魂纽带**。

在 Python 中，**实例本身其实是非常“贫瘠”的**。它通常只存储数据（如 `name`, `age`），而不存储方法（函数）。

所以在属性查找链机制中，实例向上查找的过程，就是通过 **`实例对象.__class__` 指针来索引它的所属类**。

- **它的角色**：它是一个**隐藏的属性**，存储在每个实例中。
- **它的指向**：它永远指向创建该实例的那个 **类对象**。
- **它的意义**：它是 Python 实现“属性查找链”的物理基础。

```python
class Robot:
    
	def say_hi(self):
        pass
    
r1 = Robot()
r1.say_hi()
```

当你输入 `r1.say_hi()` 时：

1. **实例碰壁**：Python 查看 `r1` 的内存空间，发现只有 `name` 数据，没有 `say_hi`。
2. **顺藤摸瓜**：Python 自动读取 `r1.__class__`。
3. **定位目标**：顺着这个指针，Python 瞬间瞬移到了 **Robot 类**。
4. **抓取成功**：在 Robot 类中找到了 `say_hi` 函数。
5. **反向注入**：将 `r1` 实例反手传给函数的 `self` 参数。

<img src="media/属性查找链机制.png" style="zoom:50%;" />

### 继承

> [!NOTE]
>
> OOP 面向对象编程的四个特性是**继承性、多态性、封装性、抽象性**，这四大特性组合起来构建了强大、易扩展的面向对象程序。
>
> | 概念     | 解释                                 | Python 实现方式                    |
> | :------- | :----------------------------------- | :--------------------------------- |
> | **封装** | 隐藏内部实现细节，**只暴露必要接口** | **私有属性 `__attr`、`@property`** |
> | **继承** | **子类复用父类代码，建立层级关系**   | **`class Child(Parent):`**         |
> | **多态** | **同一接口，不同实现**               | 方法重写、鸭子类型                 |
> | **抽象** | **定义接口规范，不关心具体实现**     | **`ABC` 模块、`@abstractmethod`**  |
>
> 为什么用 OOP？
>
> 1. **代码复用** - 继承减少重复代码
> 2. **易于维护** - 封装让修改影响范围小
> 3. **扩展性好** - 多态让新增类型不影响旧代码
> 4. **模块化** - 对象独立，便于团队协作

继承是面向对象编程的四大特性之一，即**允许一个类（子类）继承另一个类（父类）的属性和方法**，实现**代码复用、建立层次关系**。

#### 继承方式

继承又分为 **单继承** 和 **多继承** 两种形式：

##### 单继承

- **一个类想要继承某个类**需要通过 **`class <子类名>(父类名):`**的语法来实现

```python
class Father:
    pass

class Child(Father):
    pass
```

核心作用：此时 **`Child` 子类就自动拥有了 `Father` 父类的所有属性、方法**。

##### 多继承

多继承指的是**允许一个子类可以同时继承多个父类的属性、方法**，实现**多态性**。

语法：

```python
class Person:
    pass

class Worker:
    pass

class Student(Person, Worker):
    pass
```

描述：此时，**`Student` 子类同时继承了 `Person` 和 `Worker` 两个父类，同时具备了它们的属性、方法**。

###### 继承顺序

在多继承中，**`<子类名>(父类1，父类2..)` 中的 `(父类1，父类2..)` 定义的顺序**会极大**影响多个父类之间的构造方法执行顺序、属性查找顺序**。

核心概念： **`(父类1，父类2..)` 的书写顺序决定了子类的 `__mro__` 继承链查找顺序**。

```python
class Person:
    pass

class Worker:
    pass

# --------------------------------------
class Student(Person, Worker):
    pass

print(Student.__mro__)
# (<class '__main__.Student'>, <class '__main__.Person'>, <class '__main__.Worker'>, <class 'object'>)

# 换一下
class Student2(Worker,Person):
    pass

print(Student2.__mro__)
# (<class '__main__.Student'>, <class '__main__.Worker'>, <class '__main__.Person'>, <class 'object'>)
```

可以看出，在`(A,B,C...)`多继承语法中，**始终以传入的第一个父类作为直接实现父类**，其余的父类都是在此实现类的基础上扩展；所以在**后续的属性查找、构造方法执行**，传入的第一个父类**始终优先于**其他父类执行。

###### 多父类构造方法执行顺序

前面提到，**多继承语法中 `(A,B,C...)` 书写的顺序决定了子类的 “继承链”**。

所以有如下注意事项：

- 当子类**没有**定义 `__init__` 构造方法**，**实现的父类都定义了它们的构造方法，那么**只会**执行多继承语法**`(A,B,C...)`** 中传入**第一个父类 `A` 的构造方法**，其他 `B`、`C` 父类的构造方法则不会执行。

```python
class Person:
    def __init__(self):
        print('Person 类的构造方法')

class Worker:
    def __init__(self):
        print('Worker 类的构造方法')

# -----------------------------
class Student(Person, Worker):
    pass

student = Student()
# 输出：Person 类的构造方法 => Worker 类并没有执行它的构造方法

# 换一下
class Student2(Worker, Person):
    pass

student = Student2()
# 输出：Worker 类的构造方法 => Person 类并没有执行它的构造方法
```

- 当子类**定义了`__init__`构造方法**，需要**在子类空间中管理其他多个父类的共有属性**时。

  **不能通过 `super()` 来访问父类了**。因为 **`super()` 始终指向 多继承语法（A,B,C...）中传入的第一个父类 `A`**；

  解决办法：**通过 `<父类名>.__init__()`直接进行硬编码**。

  ```python
  class Person:
      def __init__(self, name, age, gender):
          self.name = name
          self.age = age
          self.gender = gender
  
      def speak(self):
          print(f"我是{self.name},今年{self.age}岁,性别{self.gender}")
  
  class Worker:
      def __init__(self, company):
          self.company = company
  
      def do_word(self):
          print(f"我在{self.company}做兼职")
  
  class Student(Person, Worker):
      def __init__(self, name, age, gender, company, stu_id, grade):
          # 多继承中，super() 始终指向第一个父类
          # 如果需要调用其他父类的构造方法，需要通过 <类名>.__init__ 手动调用
          # 注意要显式传入第一个 self 参数
          Person.__init__(self, name, age, gender)  # Person 父类的公共属性
          Worker.__init__(self, company)  # Wroker 父类的公共属性
  
          # Student 子类自己的属性
          self.stu_id = stu_id
          self.grade = grade
  
      def study(self):
          print(f"我在学习，我的学号是{self.stu_id},我的成绩是{self.grade}")
  
          
  student = Student('student', 18, '男', '肯德基', '20100507', '初三')
  print(student.__dict__)
  # {'name': 'student', 'age': 18, 'gender': '男', 'company': '肯德基', 'stu_id': '20100507', 'grade': '初三'}
  
  student.speak()  # 我是student,今年18岁,性别男
  student.do_word()  # 我在肯德基做兼职
  student.study()  # 我在学习，我的学号是20100507,我的成绩是初三
  ```

###### 多父类属性查找顺序

核心概念：

1. 当子类继承多个父类时，如果**多个父类中存在相同名称的属性**，那么在**子类中访问该属性**时：

2. 会**优先访问**多继承语法（A,B,C...）中**第一个父类 `A`**中的属性，如果第一个父类中不存在该属性，则会继续查找第二个父类`B`、`C`，以此类推，直到 object 顶层父类。

3. 如果所有父类中都不存在该属性，则会抛出 `AttributeError` 异常。

简单理解：**子类自身 > (A,B,C...) 传入的第一个父类 `A` 查找 >  (A,B,C...) 传入的第二个父类 `B` 查找 > ... > `object` 顶层父类**。

```python
class Person:
    name = 'person'

class Worker:
    name = 'worker'
    
class Student(Person, Worker):
    pass

student = Student()
print(student.name)  # person
print(Student.__mro__)
# (<class '__main__.Student'>, <class '__main__.Person'>, <class '__main__.Worker'>, <class 'object'>)

# 换一下父类传入顺序
class Student2(Worker, Person):
    pass

student2 = Student2()
print(student2.name)  # worker
print(Student2.__mro__)
# (<class '__main__.Student2'>, <class '__main__.Worker'>, <class '__main__.Person'>, <class 'object'>)
```

可以看出，Python 是**严格按照 `<子类>.__mro__` 属性中存储的 “继承链” 顺序进行属性、方法查找的**。

##### \__mro__ 继承链

Python 严格按照 **`<类名>.__mro__`** 魔术属性中定义的顺序来**执行某个类的继承链**，它**记录了子类实例对象向上查找属性、方法的顺序**。

```python
class GrandFather:
    name = 'grand_father'

class Father:
    name = 'father'

class Child(Father):
    pass

# 从左往右 依次是 当前类 > 父类 > ... > 顶层基类
print(Child.__mro__) 
# (<class '__main__.Child'>, <class '__main__.Father'>, <class '__main__.GrandFather'>, <class 'object'>)
```

##### dir() 查看实例可用的属性集

`dir()` 是 Python 的一个内置函数，它的核心作用是：**列出对象的所有“属性”和“方法”**。

它有 2 个写法：

- **不传参数**：返回**当前本地作用域内已定义的变量、函数、类和导入的模块**

  > 每个新建的 `.py` 模块，都会默认导入以下模块：
  >
  > - ['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__']

  ```python
  name = 'jack'
  age = 10
  
  def my_func():
      pass
  
  class Person:
      pass
  
  person = Person()
  print(dir())
  
  # 默认的：['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__',... ]
  # 自定义的：['Person', 'age', 'my_func', 'name', 'person',...]
  ```

- **传入实例对象**：可以**查看任何对象（字符串、列表、类实例等）可用的属性和方法（包括继承其他类的属性、方法）**

  ```python
  class Person:
      name = 'person'
      
      def run(self):
          pass
  
  class Student(Person):
      stu_id = id('20251368')
      
      def hello(self):
          pass
  
  
  student = Student()
  
  # ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__firstlineno__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__static_attributes__', '__str__', '__subclasshook__', '__weakref__', ]
  # 以上这些属性、方法都是继承自 object 顶层基类的
  
  print(dir(student))
  # [...,'hello', 'name', 'run', 'stu_id']
  
  print(dir(Student))
  # [...,'hello', 'name', 'run', 'stu_id']
  
  # [...,'name', 'run']
  print(dir(Person))
  ```

###### 查找机制

1. **实例对象本身**（`__dict__` 中的私有属性）
2. **它所属的类**（类属性和方法）
3. **继承链上的所有父类**（一直追溯到 `object` 基类）

#### 子类实例初始化决策流程

##### 核心概念

1. 当**子类继承了某个父类**，会**自动调用父类的 `__init__` 构造方法完成父类的属性、方法初始化**，此时**子类就自动拥有了父类的所有属性、方法**。

```python
class Father:
    name = 'father'

    def __init__(self):
        print('父类的 __init__ 构造方法执行')

    def say(self):
        print('父类的 say 实例方法')
        pass

class Child(Father):
    pass

child = Child()

print(child.name)
child.say()
# 输出：
# 父类的 __init__ 构造方法执行
# father
# 父类的 say 实例方法
```

2. 与之同理，**由于 Python 会为所有类预设一个 `__init__` 无参构造方法，即使父类没有显式声明 `__init__` 构造方法，也会执行父类的无参构造方法，完成父类的所有属性、方法初始化**。

```python
class Father:
    name = 'father'

    def say(self):
        print('父类的 say 实例方法')

class Child(Father):
    pass

child = Child()

print(child.name)
child.say()
# 输出：
# father
# 父类的 say 实例方法
```

3. 如果**父类显式定义了 `__init__` 构造方法，且带有参数**，但**子类没有显式定义 `__init__` 构造方法**时。

   在**子类创建实例**时，**会选择取用父类的 `__init__` 构造方法为准进行传参来初始化**，如果**强行使用子类构造方法初始化则会报错**。

> 相当于**在子类空间里，按照父类的规则对子类实例对象的属性进行初始化**
>
> > 当执行 `child = Child('Jack')` 时：
> >
> > - 内存中只产生了一个对象，它的类型（`__class__`）始终是 `Child`。
> > - Python 发现 `Child` 类里没写 `__init__`，于是顺着**继承链（MRO）**向上找到了 `Parent` 的 `__init__`。
> > - 这个 `__init__(self, name)` 被执行时，传入的 `self` 依然是那个 `Child` 类型的实例。

```python
class Father:
    def __init__(self, name, age):
        print(f"父类初始化执行，self 的类型是: {type(self)}") # <class '__main__.Child'>
        self.name = name
        self.age = age

class Child(Father):
    pass

# child = Child() 
# 由于子类没有定义 __init__ 构造方法，则会去查找父类 __init__ 构造方法，但父类的构造方法需要传递参数
# 报错：TypeError: Father.__init__() missing 2 required positional arguments: 'name' and 'age'

child = Child('child', 18) # 调用父类的构造方法，必须传父类要求的参数，在子类空间中按照父类规则进行初始化
print(child.__dict__)  # {'name': 'child', 'age': 18}
print(Father.__dict__) # {'name': 'child', 'age': 18,...}
```

4. 当子类显式定义了 `__init__` 构造方法，就需要用到 `super()` 内置函数了。 


##### super() 访问父类

###### 核心概念

如果**子类显式定义了 `__init__` 构造方法**，则**以子类自己的规则进行属性初始化**，且**会覆盖并阻断父类的 `__init__` 构造方法执行**。

那么父类定义的那些实例属性将**不会出现在子类对象中**。子类虽然继承了方法，但**失去了父类预设的初始数据状态**。

```python
class Father:
    def __init__(self):
        print('父类的 __init__ 构造方法执行')

class Child(Father):
    def __init__(self):
        print('子类的 __init__ 构造方法执行')

child = Child()
# 输出：
# 子类的 __init__ 构造方法执行
```

此时，需要**在子类的构造方法中通过 `super()` 显式调用父类的构造方法**，用于**在子类中访问父类，执行父类的初始化逻辑**。

```python
class Father:
    def __init__(self):
        print('父类的 __init__ 构造方法执行')

class Child(Father):
    def __init__(self):
        super().__init__() # 执行父类的初始化
        print('子类的 __init__ 构造方法执行')

child = Child()
# 输出：
# 父类的 __init__ 构造方法执行
# 子类的 __init__ 构造方法执行
```

> 注：如果在子类写了 `__init__` 但没有写 `super()`，父类的 `__init__` 初始化逻辑**永远不会执行**。只有当子类**完全没写** `__init__` 时，Python 才会因为**在子类找不到构造器，而向上“搜索”**并执行父类的构造器。

###### 核心目的（空间管理、属性隔离）

> [!NOTE]
>
> 这种设计的优势
>
> - **代码复用**：不需要在每个子类里重复写一遍 `self.name = name`。
> - **易于维护**：如果父类修改了基础属性的逻辑（比如给名字加个前缀），所有子类都会通过 `super()` 自动同步，而不需要一个个去改。

​	为了**区分父子类的层级关系**，**保证父类与子类的属性数据隔离【共有（父子类共享）属性与私有属性（子类特有）隔离】**；

​	通过 **`super()`** 内置函数**可以在父类空间专门保存父子类共有的共享属性**，而**让子类在自己空间中专门保存子类自己的额外属性**；等后续**子类访问某个父子类共有的共享属性**时，直接**通过 “继承链” 向上查找去父类空间里检索取用即可**。

<img src="media/super()父子类属性隔离机制.png" style="zoom:50%;" />

###### 实现方式

如果**需要调用父类的构造方法，委托父类的逻辑去处理那些所有父子类体系都有的特征**，有 2 种方式：

1. **手动调用 `super()` 内置函数访问父类的 `__init__` 构造方法并进行传参**
2. 直接**通过 `<父类名>.__init__` 访问父类的构造方法**，但**需要额外传递 `self` 参数**，**表明当前父类初始化的是哪个子类实例**

核心意义：**把父子类共有的属性交给父类去构造初始化管理**，此时，**子类只需要负责初始化只属于子类的专有属性即可**。

```python
class Father:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Child(Father):
    def __init__(self, name, age, gender):
        # 把父子类共有的属性交给父类去构造初始化
        # 方式一：super() 访问父类，调用父类的 __init__() 构造方法
        super().__init__(name, age)  # 调用父类的构造方法，并传参，在父类空间中完成属性的初始化
        
        # 方式二：直接调用父类的构造方法，将当前初始化的实例对象传递给 self
        Father.__init__(self, name, age)
        
        # 子类只需要负责初始化子类自己的一些额外属性信息
        self.gender = gender

child = Child('child', 18, 'male')
print(child.name)
print(child.age)
print(child.gender)
# 输出：
# child
# 18
# male
```

###### super(X,x) 类外部使用

`super()` 是一个内置函数，理论上除了可以在类中使用，还可以在类外部使用。

语法：

```python
super(<子类名>, <实例对象>) # 返回 子类的父类
```

作用：根据**传入的 `<子类名>` **，**找到并返回它的父类**。

```python
class A:
    def hello(self):
        print('A')


class B(A):
    @classmethod
    def say(self):
        print('B')


b = B()

super(B, b).hello() # super(B, b) = A 类
print(super(B, b))  # <super: <class 'B'>, <B object>>
```

##### 总结

```
创建子类实例时的完整流程：

1. 调用子类的 __new__(cls) → 创建实例对象
2. 调用子类的 __init__(self)：
   ├─ 如果子类没有定义 __init__：
   │   └─ 自动调用父类 __init__（按 MRO 向上）
   ├─ 如果子类定义了 __init__：
   │   ├─ 执行子类自己的初始化代码
   │   └─ 如果调用了 super().__init__()：
   │       └─ 继续沿 MRO 向上调用父类 __init__
   │   └─ 如果没有调用 super()：
   │       └─ 父类不会自动初始化（属性隔离实现）
   └─ 属性访问时：
       └─ 优先子类空间，然后沿继承链向上查找
```

<img src="media/子类实例初始化查找决策流程.png" style="zoom:50%;" />

#### 属性查找机制

在 Python 中，实例访问属性的机制可以总结为一条**“由近及远、向上溯源”**的路径。

##### 核心查找链（MRO）

当子类实例对象访问一个属性时，Python 会按照以下顺序 “爬楼梯”：

1. **实例自己的 `__dict__`**：先看该实例在 `__init__` 中是否绑定了该属性（私有/特有数据）
2. **实现类的 `__dict__`**：如果实例没有，就去它的类定义里找（类属性/方法）
3. **父类的 `__dict__`**：如果类里也没有，就顺着继承链向上找父类
4. **基类 `object`**：最后找到所有类的祖先 `object`，如果还没找到，抛出 `AttributeError`

| **阶段**   | **查找位置** | **说明**                                     |
| ---------- | ------------ | -------------------------------------------- |
| **第一步** | **实例空间** | 找 `self.xxx` 绑定的变量（最快，优先级最高） |
| **第二步** | **当前类**   | 找类定义的属性或方法                         |
| **第三步** | **继承链**   | 顺着父类、爷爷类...一路向上查                |
| **第四步** | **终点**     | 查到 `object` 类，无果则报错                 |

##### 空间隔离

- **共有属性（存放在父类/上层）**： 通过 **`super().__init__()` 触发父类构造，将共性数据初始化**

  虽然数据最终写在子类实例的内存里，但逻辑是由父类管理的。

- **特有属性（存放在子类/底层）**： 在**子类 `__init__` 中直接赋值**

  这些属性在查找链的最底端，**优先级最高**，会**覆盖**上层的同名属性。

这个设计机制的妙处：子类只需要存储自己特有的那部分“差异（Delta）”，而把沉重的公共逻辑和数据交给父类。当实例对象访问属性时，Python 的动态查找机制会自动把这些碎片拼接成一个完整的对象。

<img src="media/实例对象属性查找机制.png" style="zoom:50%;" />

#### 方法重写

**Python 不支持传统的方法重载（相同方法名不同参数）**，但支持方法重写。

核心概念：当**子类中定义了一个与父类相同的方法**时，子类的方法会**覆盖**父类的同名方法。即允许子类**重新定义**父类中已有的方法，实现**多态**行为。

```python
class Parent:
    def say(self):
        print('父类的 say 方法')


class Child(Parent):
    def say(self):
        print('子类的 say 方法')


child = Child()
child.say()
# 输出：
# 子类的 say 方法
```

##### 多层继承的方法重写

```python
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        print("B")
        super().method()  # 正确：调用父类

class C(A):
    def method(self):
        print("C")
        A.method(self)    # 硬编码父类名（多重继承时有问题）

class D(B, C):
    def method(self):
        print("D")
        super().method()

d = D()
d.method()
# D
# B
# C
# A（注意：C 也调用了 A，但通过 super() 只调用一次）

print(D.__mro__)  # (D, B, C, A, object)
```

##### super() 扩展方法重写

在子类方法中可以**使用 `super()`内置函数调用父类的其他方法实现扩展**。

```python
class Parent:
    def say(self):
        print('父类的 say 方法')

    def run(self):
        print('父类的 run 方法')


class Child(Parent):
    def say(self):
        super().run()
        print('子类的 say 方法')


child = Child()
child.say()
# 输出：
# 父类的 run 方法
# 子类的 say 方法
```

##### 重写类方法

```python
class Animal:
    species_count = 0
    
    @classmethod
    def create(cls, name):
        cls.species_count += 1
        return cls(name)
    
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    breed_count = 0
    
    @classmethod
    def create(cls, name, breed="Mixed"):
        # 重写类方法
        cls.breed_count += 1
        instance = super().create(name)
        instance.breed = breed
        return instance

dog1 = Dog.create("Buddy", "Golden Retriever")
dog2 = Dog.create("Max")
print(Dog.breed_count)  # 2
print(dog1.breed)       # Golden Retriever
print(dog2.breed)       # Mixed
```

##### 重写静态方法

```python
class MathUtils:
    @staticmethod
    def calculate(numbers):
        return sum(numbers)

class StatsUtils(MathUtils):
    @staticmethod
    def calculate(numbers):
        # 重写静态方法
        return {
            'sum': sum(numbers),
            'avg': sum(numbers) / len(numbers),
            'count': len(numbers)
        }

print(MathUtils.calculate([1, 2, 3]))     # 6
print(StatsUtils.calculate([1, 2, 3]))    # {'sum': 6, 'avg': 2.0, 'count': 3}
```

##### 重写魔术方法

```python
class Playlist:
    def __init__(self, name):
        self.name = name
        self._songs = []
    
    def __len__(self):
        """重写 len() 函数"""
        return len(self._songs)
    
    def __getitem__(self, index):
        """重写索引访问"""
        return self._songs[index]
    
    def __setitem__(self, index, value):
        """重写索引赋值"""
        self._songs[index] = value
    
    def __contains__(self, item):
        """重写 in 运算符"""
        return item in self._songs
    
    def __iter__(self):
        """重写迭代器"""
        return iter(self._songs)

playlist = Playlist("My Favorites")
playlist._songs = ["Song A", "Song B", "Song C"]
print(len(playlist))     # 3
print(playlist[0])       # Song A
print("Song B" in playlist)  # True
for song in playlist:
    print(song)
```

### 封装

封装是面向对象编程的四大特性之一，它将**数据**和**操作数据的方法**绑定在一起，通过**访问权限控制**和**属性机制**隐藏内部实现细节，**只对外暴露必要的接口**，使代码更安全、更易维护。

核心目的：

- 数据隐藏：保护数据不被随意修改
- 接口简化：只暴露必要的操作
- 实现隔离：内部实现变化不影响外部代码

#### 封装颗粒度

在 Python 中，根据**封装的可控制颗粒度（外部访问行为处理）**，分为以下 3 个层级：

- **初级封装**：属性、方法**命名约定**（`_attr`, `__attr`）【**单一属性访问权限控制**】
- **中级封装**：使用 **`@property` 装饰器**（将方法伪装成属性）【**单一属性访问拦截控制**】
- **高级封装**：
  - 定义 **`__slots__`** 魔术属性【**限制类中可访问属性集**】
  - 描述符协议：重写 **`__get__`  与 `__set__` ** 魔术方法【**底层对类属性的访问拦截器，通过一套校验规则验证类属性值的检查**】

#### 核心概念

编程语言通过一定的规则**将数据和操作绑定在一起**，根据**数据的隐私级别**划分**不同的外部访问权限等级**，通常为：

- **公开的（public）**：数据对外**全部可见**；外部可**直接访问修改**。使用者只需要知道公开接口，不需要了解内部实现
- **受保护的（protected）**：数据对外**部分可见**；类对外提供 “授权” 约定，外部需实现并提供一定的身份验证才可以访问
- **隐私的（private）**：数据对外**不可见**；外部**需通过类中对外暴露的接口来访问隐私数据**，若不暴露访问接口，则永远无法访问

这 3 个权限等级既保证了类中数据的可扩展性，也保证了类中数据的安全与隐私性。

#### 属性访问权限控制机制

Python 通过**约定命名规范（软约束）**的形式来实现对类中属性、方法的初级**访问权限控制**。

##### 公开属性、方法

所有**正常命名（不加任何前缀）的属性、方法**都是类中**公开**的成员，外部可直接访问。

- 访问权限：**当前类、子类中、类外部**都可以直接访问

- 核心原则：**直接看 `__dict__ ` 属性集中是否有这个属性、方法**

```python
class Person:
    # 公开类属性
    person_id = 'person'

    def __init__(self, name, age):
        self.name = name  # 公开实例属性
        self.age = age  # 公开实例属性

    # 公开类方法
    @classmethod
    def hello(cls):
        print('你好')

    # 公开实例方法
    def eat(self):
        print('吃饭')

    # 公开静态工具方法
    @staticmethod
    def sleep():
        print('睡觉')

    # ------- 当前类中访问 -------
    # 实例方法访问实例属性、实例方法、静态工具方法、类属性、类方法
    def run(self):
        print(f"{self.name}, {self.age}")
        self.eat()
        print(self.__class__.person_id)
        self.__class__.hello()
        self.__class__.sleep()

    # 类方法中访问类属性、类方法、静态工具方法
    @classmethod
    def init(cls):
        print(cls.person_id)
        cls.hello()
        cls.sleep()


class Student(Person):
    pass


# -------类外部访问公开属性、方法-------
# 当前类访问 ---> 公开类属性/方法、静态工具方法、实例方法，直接通过 <类名>.__dict__ 查看
print(Person.__dict__)
# {'person_id':'person','init':...,'hello':...,'eat':...,'sleep':...,'run':...,...}

print(Person.person_id)  # person
Person.hello()  # 你好
Person.sleep()  # 睡觉

# 当前类实例访问 ---> 公开实例属性，直接通过 <实例对象>.__dict__ 查看
person = Person('jack', 18)
print(person.__dict__)  # {'name': 'jack', 'age': 18}
print(person.name)  # jack
print(person.age)  # jack
person.run()  # 输出：jack, 18 你好 吃饭 睡觉

# ----> 子类访问父类的公开属性/方法
student = Student('tom', 20)
print(student.__dict__)  # {'name': 'tom', 'age': 20}
print(student.name)  # tom
student.run()  # tom, 20 你好 吃饭 睡觉
```

##### _xx 受保护属性、方法

所有**约定以 `_` 单下划线开头命名的属性、方法**都是类中**受保护**的成员，外部**需持有一定身份**才可以访问。

- 访问权限：**当前类、子类中**才可以访问；**外部虽然也可以直接访问，但是不应该（约定）**

- 核心原则：**直接看 `__dict__ ` 属性集中是否有这个属性、方法**；

  - 会看到 **类与实例对象的 `__dict__` 都包含了`_` 单下划线开头的受保护属性、方法**；这意味着外部其实也可以直接强制访问，但是不推荐。

    > Python 提倡约定看到以`_` 单下划线开头的属性、方法，开发者就应该意识到这是一个受保护的属性、方法，不应该直接访问，而是按照约定规范来操作（**约定大于约束** 准则），这是因为 Python 为了展现极致灵活性的做法。
    >
    > 解释：**Python 中的受保护成员（单下划线）是一种"君子协定"：它告诉使用者"这个成员是内部的，需要身份验证（理解风险）才能访问"，但并不强制执行。这种设计平衡了灵活性和封装性，让专家可以绕过限制，同时警告新手不应依赖。**

```python
class Person:
    # 受保护的类属性
    _person_id = '_person'

    def __init__(self, name, age, stu_id, grade):
        self.name = name  # 公开实例属性
        self.age = age  # 公开实例属性
        self._stu_id = stu_id  # 受保护的实例属性
        self._grade = grade  # 受保护的实例属性

        # 当前类中访问受保护的属性、方法
        self._run()
        print(self.__class__._person_id)
        self.__class__._eat()
        self.__class__._sleep()

    # 受保护的类方法
    @classmethod
    def _eat(cls):
        print('吃饭')

    # 受保护的静态工具方法
    @staticmethod
    def _sleep():
        print('睡觉')

    # 受保护的实例方法
    def _run(self):
        print(f"{self._stu_id}, {self._grade}")
        self._eat()


class Student(Person):

    def __init__(self, name, age, stu_id, grade, gender):
        super().__init__(name, age, stu_id, grade)

        # 子类中访问父类的受保护属性、方法
        print(self._stu_id)  # 002
        print(self._grade)  # 初三
        print(self._person_id)  # _person
        self._eat()  # 输出：吃饭

        # 子类扩展自己的受保护属性
        self._gender = gender  # 受保护的实例属性


# -------类外部访问公开属性、方法-------
print(Person.__dict__)
# {'_person_id':'person','_eat':...,'_sleep':...,'_run':...,...}

person = Person('jack', 18, '001', '初三')
print(person.name)  # ✅ 公开
print(person.age)  # ✅ 公开
print(person._stu_id)  # ⚠️ 可以访问，但违反约定
print(person._grade)  # ⚠️ 可以访问，但违反约定

print(person.__dict__)
# {'name': 'jack', 'age': 18, '_stu_id': '001', '_grade': '初三'}
Person._sleep()  # # ⚠️ 可以访问，但违反约定

# 子类访问父类受保护的属性、方法
student = Student('tom', 20, '002', '初三', '男')
print(student.__dict__)
# {'name': 'tom', 'age': 20, '_stu_id': '002', '_grade': '初三', '_gender': '男'}
print(student._stu_id)  # ⚠️ 可以访问，但违反约定
print(student._grade)  # ⚠️ 可以访问，但违反约定
print(student._gender)  # ⚠️ 可以访问，但违反约定
```

##### __xx 隐私属性、方法

所有**约定以 `__` 双下划线开头命名的属性、方法**都是类中**隐私**的成员。

- 访问权限：仅在**当前类**才可以访问；外部不可以访问，会直接报错

- 核心原则：**直接看 `__dict__ ` 属性集中是否有这个属性、方法**；

  - 会**看到 `__` 双下划线开头的隐私属性、方法都被自动添加了 `_类名` 前缀（`_类名__属性名`）**

    这是 Python 内部的机制（**软约束**）：

    - Python 会将类中所有 `__` 双下划线开头的隐私属性、方法自动添加一个 `__类名` 前缀，以**防止外部通过 `__属性名` 的形式直接访问**
    - Python 通过直接改变属性、方法的命名，让外部无法知道真实的属性、方法名，从而无法直接访问

    - 但真的要强制访问的话，还是**可以通过 `__dict__` 查看隐私属性、方法真实的命名，从而直接通过 `__dict__[__类名_属性名]`来强制访问隐私数据**。只是这样的做法**不推荐**。

```python
class Person:
    # 隐私类属性
    __person_id = '__person'

    def __init__(self, name, age, stu_id, grade):
        self.name = name  # 公开实例属性
        self.age = age  # 公开实例属性
        self.__stu_id = stu_id  # 隐私实例属性
        self.__grade = grade  # 隐私实例属性

        # 当前类中访问隐私属性、方法
        self.__run()
        print(self.__class__.__person_id)
        self.__class__.__eat()
        self.__class__.__sleep()

    # 隐私类方法
    @classmethod
    def __eat(cls):
        print('吃饭')

    # 隐私静态工具方法
    @staticmethod
    def __sleep():
        print('睡觉')

    # 隐私实例方法
    def __run(self):
        print(f"{self.__stu_id}, {self.__grade}")
        self.__eat()


class Student(Person):
    pass


# -------类外部访问公开属性、方法-------
print(Person.__dict__)
# {'_Person__person_id':'__person','_Person__eat':...,'_Person__sleep':...,'_Person__run':...,...}

person = Person('jack', 18, '001', '初三')

print(person.__dict__)
# {'name': 'jack', 'age': 18, '_Person__stu_id': '001', '_Person__grade': '初三'}
print(person.name)  # ✅ 公开
print(person.age)  # ✅ 公开
# print(person.__stu_id)  # ⚠️ 报错，外部不可以访问
# print(person.__grade)  # ⚠️ 报错，外部不可以访问

# 子类访问父类的隐私数据
student = Student('tom', 20, '002', '初三')
print(student.__dict__)
# {'name': 'tom', 'age': 20, '_Person__stu_id': '002', '_Person__grade': '初三'}
# print(student.__stu_id)  # ⚠️ 报错，外部不可以访问

# ---- 通过 __dict__ 魔术属性查看并强制访问 ---- 【不推荐】
print(person.__dict__)
# {'name': 'jack', 'age': 18, '_Person__stu_id': '001', '_Person__grade': '初三'}
print(person._Person__stu_id)  # ✅ 可以访问，但违反约定
print(person._Person__grade)  # ✅ 可以访问，但违反约定
print(person._Person__person_id)  # ✅ 可以访问，但违反约定
```

##### 总结

核心对比：

| 特性             | **公开（Public）** | **受保护（Protected）** | **私有（Private）**    |
| :--------------- | :----------------- | :---------------------- | :--------------------- |
| **命名约定**     | `name`             | `_name`                 | `__name`               |
| **语法形式**     | 无下划线开头       | 单下划线开头            | 双下划线开头           |
| **访问限制**     | ❌ 无限制           | ⚠️ 约定限制（非强制）    | ✅ 名称修饰限制         |
| **外部直接访问** | ✅ 允许             | ⚠️ 允许（但违反约定）    | ❌ 不允许（名称修饰）   |
| **子类访问**     | ✅ 允许             | ✅ 允许（设计意图）      | ❌ 不允许（需名称修饰） |
| **名称修饰**     | ❌ 不修饰           | ❌ 不修饰                | ✅ 修饰为 `_类名__name` |
| **IDE 警告**     | ❌ 无警告           | ⚠️ 有警告                | ❌ 无警告（无法访问）   |
| **文档生成**     | ✅ 包含             | ⚠️ 通常排除              | ❌ 排除                 |

详细对比：

| 对比维度              | 公开（Public） | 受保护（Protected）  | 私有（Private）            |
| :-------------------- | :------------- | :------------------- | :------------------------- |
| **设计目的**          | 对外稳定接口   | 内部实现，子类可扩展 | 绝对隐藏，防止意外覆盖     |
| **访问方式**          | `obj.name`     | `obj._name`          | `obj._ClassName__name`     |
| **外部访问后果**      | 正常使用       | 违反约定，可能不稳定 | 需要知道修饰名，强烈不建议 |
| **子类重写**          | ✅ 允许         | ✅ 允许（设计意图）   | ⚠️ 可以但创建新属性         |
| **API 稳定性**        | ✅ 稳定         | ⚠️ 不稳定，可能变化   | ❌ 不应依赖                 |
| **文档化**            | ✅ 应该文档化   | ⚠️ 可选（内部使用）   | ❌ 不文档化                 |
| **单元测试访问**      | ✅ 直接测试     | ⚠️ 可访问但需注意     | ❌ 需通过公共接口           |
| **序列化默认行为**    | ✅ 包含         | ⚠️ 通常不包含         | ❌ 不包含                   |
| **`dir()` 显示**      | ✅ 显示         | ✅ 显示               | ⚠️ 显示修饰名               |
| **`__dict__` 中键名** | `name`         | `_name`              | `_ClassName__name`         |

#### getter/setter （实例属性访问劫持）

描述：如果想要**让受保护的属性、隐私属性能安全的被外部访问、修改**，可通过**为属性注入 `getter/setter` 能力**；经过`getter/setter` 能力的修饰后，当外部想要访问、修改受保护的属性时则**需要经过一定的逻辑运算与校验处理才能安全通过**。

核心逻辑：定义**与受保护的属性相同名称的方法**（方法名也可以随意，但为了突出一致性与像属性一样的使用体验，方法名通常要与属性名相同），并**添加对应的装饰器**，**使该方法修饰成像属性一样使用**。*类似于 Vue 的 computed 计算属性*

> 也可以像下面这样写，但是书写并不方便，容易造成方法名歧义混杂，且每次都要调用方法，比较麻烦，为了代码简洁，还是建议选择装饰器写法。
>
> ```python
> class Person:
>  def __init__(self, name, age):
>      self.__name = name
>      self.__age = age
> 
>  def get_name(self):
>      return self.__name
> 
>  def set_name(self, name):
>      self.__name = name
> 
>  def get_age(self):
>      return self.__age
> 
>  def set_age(self, age):
>      self.__age = age
> 
> person = Person('jack', 18)
> 
> person.get_name()
> person.set_name('tom')
> person.set_age(20)
> person.get_age()
> ```

`getter/setter` 能力的注入分为三部分：

- **`getter` 能力**：通过**为方法添加 `@property` 装饰器**修饰；当外部**通过 `<对象名>.<方法名>` 时会自动触发**该方法
- **`setter` 能力**：通过**为方法添加 `@<方法名>.setter` 装饰器**修饰；当外部**通过 `<对象名>.<方法名> = xxx` 时会自动触发**该方法
- **`deleter` 能力**：通过**为方法添加 `@<方法名>.deleter` 装饰器**修饰；当外部**通过 `del <对象名>.<方法名>` 时会自动触发**该方法

> 注：`setter` 和 `deleter` 装饰器都是可选的，分别对应不同的扩展能力。若只有 `getter` 装饰器，则该方法就是**只读**的计算属性。

##### @property => getter

作用：**将类中的方法修饰成像属性一样使用**。外部**通过 `<对象名>.<方法名>` 访问时会自动执行该方法**，并返回某个具体属性值。

描述：注入了 `@property` 装饰器的方法，内部逻辑**通常是返回某个具体属性值（属性的 `getter`方法）**，相当于是**在获取某个受保护的属性值之前添加了一层校验逻辑**。

```python
class Person:
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    @property
    def name(self):
        return self.__name

    @property
    def age(self):
        return self.__age


person = Person('jack', 13)

# 方法像属性一样使用
print(person.name)  # jack
print(person.age)  # 13
```

##### @<方法名>.setter

作用：为注入了`@property` (getter) 能力的方法扩展一个 **`setter` 能力**方法。（**`setter` 方法必须与 `@property` 方法同名**）

- 外部**通过 `<对象名>.<方法名> = xxx` 修改值时会自动执行该方法**。

描述：注入了 `@<方法名>.setter` 装饰器的方法，内部逻辑**通常是修改某个具体属性值（属性的 `setter`方法）**，相当于是**在修改某个受保护的属性值之前添加了一层校验逻辑**。

```python
class Person:
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, name):
        self.__name = name

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, age):
        if (age >= 110):
            return False
        else:
            self.__age = age


person = Person('jack', 13)

# 将方法变得像属性一样使用
print(person.name)  # jack
print(person.age)  # 13

# 外部通过 对象.方法名 = xxx 修改赋值时自动触发对应的 setter 方法
person.name = 'tom'
person.age = 112
print(person.name)  # tom
print(person.age)  # 13
```

##### @<方法名>.deleter

作用：为注入了`@property` (getter) 能力的方法扩展一个 **`deleter` 能力**。（**`deleter` 方法必须与 `@property` 方法同名**）

- 外部**通过 `del <对象名>.<方法名>` 删除属性时会自动执行该方法**。

描述：注入了 `@<方法名>.deleter` 装饰器的方法，内部逻辑**通常是删除某个具体属性（属性的 `deleter`方法）**，相当于是**在删除某个受保护的属性之前添加了一层校验逻辑**。

```python
class Person:
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    @property
    def name(self):
        print(2)
        return self.__name

    @name.setter
    def name(self, name):
        self.__name = name

    @name.deleter
    def name(self):
        print('del __name')
        del self.__name

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, age):
        if (age >= 110):
            return False
        else:
            self.__age = age

    @age.deleter
    def age(self):
        print('del age')
        del self.__age


person = Person('jack', 13)

# 将方法变得像属性一样使用，获取值
print(person.name)  # jack
print(person.age)  # 13

# 外部通过 对象.方法名 = xxx 修改赋值时自动触发对应的 setter 方法
person.name = 'tom'
person.age = 112
print(person.name)  # tom
print(person.age)  # 13

# 外部通过 del 对象.方法名 删除某个属性值自动触发对应的 deleter 方法
del person.name  # 输出 del __name
del person.age  # 输出 del age
```

##### Property vs Getter/Setter

Python 对类中属性、方法的 `getter/setter`能力封装可以有 2 种写法：

```python
# Java 风格（Python 中不推荐）
class JavaStyle:
    def __init__(self, value):
        self._value = value
    
    def get_value(self):
        return self._value
    
    def set_value(self, value):
        self._value = value

# Pythonic 风格（推荐）
class PythonicStyle:
    def __init__(self, value):
        self._value = value
    
    @property
    def value(self):
        return self._value
    
    @value.setter
    def value(self, value):
        self._value = value

# 使用对比
js = JavaStyle(10)
print(js.get_value())  # 需要调用方法
js.set_value(20)

ps = PythonicStyle(10)
print(ps.value)        # 像属性一样访问
ps.value = 20          # 像属性一样赋值
```

#### 描述符协议（类属性访问劫持）

##### 基本概念

核心逻辑：**描述符协议**就是**通过一个独立的代理类去劫持其他类的类属性访问过程**，针对性的**使用同一套逻辑对类属性进行校验**。

核心功能：编写通用的验证类、缓存装饰器，或在开发大型框架/库时处理复杂的属性绑定。

要想实现一个描述符协议，通常需要具备如下 3 个角色的参与：

- **描述符类**：一个**独立的代理类**；

  ​	它内部**封装了一套用于拦截类属性赋值的可复用校验逻辑模板（`__get__` 和 `__set__`）**，用于**给宿主类的类属性取值赋值处理添加一层安全检测**。

- **宿主类**：**产出实例的实体类模板**，可定义若干个类属性。**宿主类的类属性赋值**需要**由 “描述符类” 进行代理拦截**

  ​	通过**引用 “描述符类” 实例作为类属性值**，把**外部传入的类属性值交给 “描述符类” 去拦截处理**，进而执行它内部封装的校验逻辑处理，最后**将处理之后的类属性值还给宿主类实例的 `__dict__` 去独立管理**。

- **宿主类实例对象**：宿主类创建的实例对象，**真正的数据实体用户**；

  ​	当**宿主类创建实例产出数据实体后**，会经过一系列的类属性初始化，而类属性初始化会**经过 “描述符类” 的代理拦截**，从而**把处理之后的类属性键值对交给宿主类实例对象的 `__dict__` 去独立管理**。也就是说**类管理属性值校验逻辑模板，实例对象管理各自独立的类属性值**。

> 以上三者之间是**“代理-宿主-实例”**的三位一体关系
>
> > **类对象 (Class Object)**
> >
> > - **存储内容**：描述符类的**实例**（即定义了 `__get__` / `__set__` 的对象）。
> > - **地位**：它是描述符的“家”。描述符必须定义为类属性，否则协议不会生效。
> >
> > **实例对象 (Instance Object)**
> >
> > - **存储内容**：具体的**属性数据**。
> > - **地位**：它是数据的“仓库”。当通过实例访问属性时，Python 会去“类”里找描述符。
> >
> > **描述符协议 (Protocol)**
> >
> > - **作用**：连接类与实例的**纽带**。
> > - **逻辑**：它像一个拦截器，把原本对实例 `__dict__` 的直接访问，重定向到描述符的方法上。

简单理解：**类属性赋值拦截逻辑**在**“描述符类**”里，**配置**在**“宿主类”**的属性位上，但**真实的数据**是存在**“宿主类实例对象”**的 `__dict__` 里。

<img src="media/描述符协议核心流程.png" style="zoom:50%;" />

<img src="media/描述符协议核心逻辑流程.png" style="zoom:50%;" />

##### 什么是描述符类？

描述符类本质上就是一个**普通的 Python 类**。这个类里只有 2 个方法：**`__get__`** 和 **`__set__`** 魔术方法（非 `object` 类默认内置）。

核心功能：通过**为每个类属性赋值为 “描述符类” 实例**，用于**劫持外部对类中绑定了 “描述符”实例的类属性的访问、修改动作，并添加一些自定义逻辑**。

*类似于 JavaScript 的 Object.defineProperty() 拦截器*

核心价值：当**外部想要访问、修改一个类属性**时，不再是简单的从 `__dict__` 内存字典中取值，而是要**经过 “描述符类”中的一段自定义逻辑处理**，校验通过后，才真实赋值给类属性。

###### 与 `@property` 装饰器的对比

> - **`@property` 装饰器**：**可选地针对类中某个实例属性**的访问拦截机制；若不添加，则没有拦截效果**（单一实例属性的拦截）**
>   - 它是描述符的最常见包装
> - **`__get__`、`__set__` 拦截器**：**可选的针对类中某个类属性**添加的访问拦截机制**（单一类属性的拦截）**
>   - **`@classmethod` 和 `@staticmethod`**：它们底层都是通过 `__get__` 改变了函数的调用方式

###### \__get__(self,instance,owner) 类属性取值拦截器

触发时机：当**外部通过 `<实例对象名>.<类属性名>` 或 `<类名>.<类属性名>` **进行**取值**时自动触发**拦截**，并执行对类属性的**自定义取值处理逻辑**。

参数：

- **`self`**：描述符类实例对象本身【**代理类实例**】
- **`instance`**：访问类属性的具体实例对象【**数据实体**】（如果是通过类访问，则为 `None`）
- **`owner`**：类属性所属的类【**宿主类模板**】

```python
# 代理类
class ValueDescriptioner:

    def __get__(self, instance, owner):
        if instance is None:
            print('get owner', owner)
            # 类实例 访问类属性
            return 'get owner'
        else:
            # 实例对象 访问类属性
            print('get instance', instance)
            return 'get instance'

class Person:
    name = ValueDescriptioner() # 每个类属性都是一个 “描述符类” 实例


person = Person()

print('实例访问', person.name) 
# -- 当访问 person.name 时，会执行 ValueDescriptioner 代理类的 __get__ 方法，执行对类属性的自定义取值逻辑。
# 输出：
# get instance <__main__.Person object at 0x000001EE45D18D70>
# 实例访问 get instance

print('类访问', Person.name)
# -- 当访问 Person.name 时，会执行 ValueDescriptioner 代理类的 __get__ 方法，执行对类属性的自定义取值逻辑。
# 输出：
# get owner <class '__main__.Person'>
# 类访问 get owner
```

###### \__set__(self,instance,value) 类属性赋值拦截器

触发时机：当**外部通过 `<实例对象名>.<类属性名>`** 进行**赋值**时自动触发**拦截**，并执行对类属性的**自定义赋值处理逻辑**。

参数：

- **`value`：想要赋给类属性的新值**

```python
# 代理类
class ValueDescriptioner:
    def __set__(self, instance: object, value):
        if instance is None:
            # 类实例 访问类属性
            print('set owner', instance.__class__)
            instance.__class__.__dict__['name'] = f"值：{value}"
        else:
            # 实例对象 访问类属性
            print('set instance', instance)
            instance.__dict__['name'] = f"值：{value}"

class Person:
    name = ValueDescriptioner() # 每个类属性都是一个 “描述符类” 实例


person = Person()
print(person.__dict__)  # 未赋值前 {}

person.name = 100
# 当执行 person.name = 100 时，会触发 ValueDescriptioner 类的 __set__ 方法，执行对类属性的自定义赋值逻辑
# 输出：
# set instance <__main__.Person object at 0x000001EE45D18D70>

print(person.__dict__)  # 经过 __set__ 处理之后的新值 {'name': '值：100'}
print(person.name)  # 值：100
```

##### 类属性访问描述符

通过描述符协议，可以实现对一个类中所有类属性的规则校验模型，建立一个强壮的类。

###### 实现逻辑

1. 将类属性赋值给一个特定的 “描述符类”，为 “描述符类” 定义一个构造方法，该构造方法接收以下参数：
   - `name`：初始化的类属性名，用于作为 `key` 键后续存入到实例对象中的 `__dict__` 字典
   - `validator`：属性值校验函数
   - `error_msg`：校验不通过时抛出的错误信息
2. 类属性本身是每个实例共享的，通过描述符协议可以使得类本身存储校验逻辑，实例独立存储真实的类属性数据
   1. 每次 `类实例.类属性名` 获取值时，访问的是 类实例的 `__dict__` 中存储的对应数据
   2. 每次 `类实例.类属性名 = xx`，改变的是 类实例的 `__dict__` 中中存储的对应数据
3. 以此管理宿主类中每个类属性的值

核心理解：**每个类属性的值都是一个 “描述符类”，当外部访问、修改类属性值，会自动触发 “描述符类” 定义的 `__get__` 和 `__set__` 魔术方法，并执行对应的自定义取值、赋值校验逻辑，最后将类属性键值对分别存储到各个类实例中的 `__dict__` 独立管理**

###### 数据存储位置分离

通过在宿主类中定义 “顶层” 视图的类属性，经过 “描述符类” 的校验逻辑处理之后，真实的数据存储在类实例对象的 `__dict__` 字典中；

层次关系如下：

- **验证逻辑（描述符）：存在类中（共享）**
- **实际数据：存储在各个实例中（独立）**

```
┌─────────────────────────────────────────────────────────────┐
│                      类 Person                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ age = Validator对象（描述符）                         │   │
│  │ name = Validator对象（描述符）                        │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                    │                    │
                    │ 实例化              │ 实例化
                    ▼                    ▼
    ┌──────────────────────┐    ┌──────────────────────┐
    │ 实例 p1               │    │ 实例 p2               │
    │ ┌──────────────────┐ │    │ ┌──────────────────┐ │
    │ │ __dict__:        │ │    │ │ __dict__:        │ │
    │ │   age: 25        │ │    │ │   age: 30        │ │
    │ │   name: "Alice"  │ │    │ │   name: "Bob"    │ │
    │ └──────────────────┘ │    │ └──────────────────┘ │
    └──────────────────────┘    └──────────────────────┘
```

###### 代码实现

- 描述符通用校验模板：

```python
# 代理类
class Validator:
    """万能校验基类"""

    # 构造方法；
    # 创建一个“描述符类”实例赋值给宿主类的类属性，在类属性初始化时，传递该属性对应的名称，数据校验函数以及错误信息
    def __init__(self, attr_name: str, validator: Any, error_msg: str):
        self.__attr_name = attr_name
        self.__validator = validator
        self.__error_msg = error_msg

    # 自定义 __get__ 类属性取值逻辑
    def __get__(self, instance: object, owner):
        if instance is None:
            return self
        else:
            # ---实例对象对类属性取值---
            print(instance.__dict__, self.__attr_name)
            # {'name': 'jack', 'age': 18} name

            # 真正取值时，去实例的私有字典里拿
            return instance.__dict__.get(f"{self.__attr_name}", None)

    # 自定义 __set__ 类属性赋值逻辑
    def __set__(self, instance: object, value: Any):
        if instance is None:
            # 如果是通过类访问（User.age），返回描述符本身，不返回具体类属性值
            # 因为类属性值是独立存储在各个类实例的 __dict__ 属性中，而不是存储在类本身的 __dict__ 属性集字典中
            return self
        else:
            # ---实例对象对类属性赋值---
            # 赋值时，传入的值不符合校验规则，抛出错误信息
            if not self.__validator(value):
                raise ValueError(self.__error_msg)

            print(f"✅ 校验通过：将 {self.__attr_name} 设置为 {value}")
            # 重点：符合规则的，则存储在每个实例对象各自的 __dict__ 属性集字典中，确保实例之间数据隔离
            instance.__dict__[f"{self.__attr_name}"] = value


# 宿主类
class Person:
    name = Validator(
        'name',
        lambda x: not len(x) <= 2,
        "姓名长度不能小于 2 位"
    )

    age = Validator(
        'age',
        lambda x: 0 < x < 120,
        "年龄必须在 0 ~ 120 之间"
    )

    def __init__(self, name, age):
        pass


# 使用
person = Person('jack', 13)
# person.name = '2'  # 抛出错误：ValueError: 姓名长度不能小于 2 位
# person.age = 159  # 抛出错误：ValueError: 年龄必须在 0 ~ 120 之间

person.name = 'jack'
person.age = 18

print(person.name)  # jack
print(person.age)  # 18
```

- 描述符扩展校验模板：

```python
# 实现一个可复用的属性访问描述符类
class Validator:
    """通用校验基类"""

    def __init__(self, attr_name: str):
        self.attr_name = attr_name

    def __get__(self, instance: object, owner):
        if instance is None:
            return self
        else:
            return instance.__dict__.get(f"{self.attr_name}", None)

    def __set__(self, instance: object, value: Any):
        if instance is None:
            return self
        else:
            self.validate(value)
            instance.__dict__[f"{self.attr_name}"] = value

    def validate(self, value: Any):
        """留给子类实现的 validate 校验函数"""
        raise NotImplementedError


class String(Validator):
    """字符串校验描述符类"""

    def validate(self, value: Any):
        if not isinstance(value, str):
            raise TypeError(f"{self.attr_name} 必须是字符串类型")
        if len(value) < 2:
            raise ValueError(f"{self.attr_name} 长度不能小于 2 位")


class Integer(Validator):
    """数字校验描述符类"""

    def validate(self, value: Any):
        if not isinstance(value, int):
            raise TypeError(f"{self.attr_name} 必须是整数类型")
        if value <= 0 or value >= 120:
            raise ValueError(f"{self.attr_name} 必须在 0 ~ 120 之间")


# 宿主类
class Person:
    name = String('name')
    age = Integer('age')

    def __init__(self, name, age):
        self.name = name
        self.age = age


# 使用
person = Person('jack', 13)
# person.name = '2'  # 抛出错误：ValueError: name 长度不能小于 2 位
# person.age = 156  # 抛出错误：ValueError: age 必须在 0 ~ 120 之间

print(person.__dict__)  # {'name': '22', 'age': 100}
```

#### 属性动态代理（所有属性访问劫持）

在 Python 中，`object` 基类为所有类都提供了一套关于类中所有属性的访问控制魔术方法。如下：

- **`__setattr__(self,name,value)`**：**设置任何属性时**触发
- **`__getattr__(self,name)`**：**访问实例对象中不存在的属性时**触发
- **`__getattribute__(self,name)`**：**访问任何属性时触发**
- **`__delattr__(self,name)`**：**删除属性时**触发

通过重写以上方法，可以**自定义控制外部操作实例对象中的实例属性、类属性时的行为**。

> 注意：直接通过类操作类属性**不会**拦截，以上方法**只关心实例对象**的操作。

##### \__setattr__(self,name,value) 属性赋值拦截器

作用：当外部**通过 `<实例对象>.<属性名> = xxx` 为实例、类属性赋值**时触发。

**参数：**

- **`name`：要赋值的属性名**
- **`value`：要给属性赋的新值**

```python
class Person:
    person_id = 'person_id'

    # 设置任何属性值时触发
    def __setattr__(self, name: str, value: Any, /) -> None:
        print(f"设置属性 {name} 的值为 {value}")
        self.__dict__[name] = value

person = Person()

# 设置属性值，会触发 __set__ 魔术方法
person.name = 'jack'  # 设置实例属性 name 的值为 jack
person.person_id = '__person_id__'  # 设置类属性 name 的值为 jack

Person.person_id = 'dw'  # 直接通过类操作类属性不会被拦截
```

##### \__getattr__(self,name) 缺省代理

作用：当外部**通过 `<实例对象>.<属性名>` 访问实例对象中不存在的属性**时触发。

特点：这是最常用、也最温和的代理方式。它只在 **“属性找不着”** 的时候才会被触发。它是一个“兜底”机制。如果实例字典、类字典、父类里都没有这个属性，Python 才会来问它。

**适用场景**：封装一个 SDK，或者把一个字典伪装成对象（比如 `obj.key` 代替 `obj['key']`）

**参数：**

- **`name`：要访问的属性名**

```python
class DynamicProxy:
    def __init__(self, data):
        self._data = data

    def __getattr__(self, name):
        # 当访问不存在的属性时，去字典里找
        print(f"--- 触发缺省代理: 正在查找 {name} ---")
        if name in self._data:
            return self._data[name]
        raise AttributeError(f"找不到属性 {name}")

proxy = DynamicProxy({'name': 'Jack', 'age': 27})
print(proxy.name)  # 触发代理
```

##### \__getattribute__(self,name) 全量代理

作用：外部**通过`<实例对象名>.<属性名>` 获取任何属性**时触发。

特点：无论属性存不存在都会经过它，权利极大。它是所有属性访问的“必经之路”。

核心功能：**全量代理**，定义一个**代理类**，将**某个类的实例传入给该代理类，可以拦截外部对该类的任何属性操作，从而执行自定义的属性访问处理逻辑**。*类似于 JavaScript 中的 Proxy*

注意：在它内部访问 `self.xxx` 时，**必须使用 `super().__getattribute__(name)`，否则会陷入无限递归（因为它会再次拦截自己），从而抛出栈溢出异常**。

**参数：**

- **`name`：要访问的属性名**

```python
# 全量代理，定义一个代理类，将某个类的实例传入给该类，可以拦截外部对该类的任何属性操作，从而执行自定义的属性访问处理逻辑
# 原始类
class Database:
    def connect(self):
        return "数据库连接成功"

    def execute(self, query):
        return f"正在执行: {query}"


# 代理类（代理原始类实例的操作）
class MonitorProxy:
    def __init__(self, target):
        # target 是被代理的真实对象（Database 实例）
        self._target = target

    def __getattr__(self, name):
        # 1. 从被代理对象中获取属性或方法
        attr = getattr(self._target, name)

        # 2. 如果是方法，我们动态包装它
        if callable(attr):
            def wrapper(*args, **kwargs):
                start_time = time.time()
                # 执行真实的方法
                result = attr(*args, **kwargs)
                end_time = time.time()

                print(
                    f"--- [监控日志] 方法 '{name}' 执行耗时: {end_time - start_time:.4f}s ---")
                return result
            return wrapper

        # 3. 如果是普通属性，直接返回
        return attr


# 原始对象
real_db = Database()
# 穿上“代理外套”
db = MonitorProxy(real_db)

# 使用方式和原来一模一样，但多了日志功能
print(db.connect())
print(db.execute("SELECT * FROM users"))
```

> - **描述符**是“点对点”的拦截（针对具体某个插槽）。
>
> - **动态代理**（`__getattr__` 等）是“面对面”的拦截（针对整个对象）。

##### \__delattr__(self,name) 属性删除拦截器

作用：当外部**通过 `del <实例对象>.<属性名>` 删除实例属性**时触发。

**参数：**

- **`name`：要删除的属性名**

```python
class Person:
    
    # 删除属性时触发
    def __delattr__(self, name: str, /) -> None:
        print(f"删除属性 {name}")
        del self.__dict__[name]

person = Person()
person.name = 'jack'

# 删除实例属性，会触发 __del__ 魔术方法
del person.name  # 删除属性 name
```



#### \__slots__ 限制可访问属性集

描述：正常情况下，`__dict__` 属性会将类以及类实例对象的所有属性（包括改名后的隐私属性）都暴露给外部访问，这不利于类中数据的隐私性，而且如果一个类具有大量属性时，`__dict__` 属性会消耗大量内存来存储这些属性。

作用：Python 为所有类提供了 `__slots__` 魔术属性，可以**重写**它，用于**规定当前类只能有多少个属性、限制暴露给外部的属性集**。

核心价值：**`__slots__` 属性会取代类的 `__dict__` 属性**，使得**外部只能访问 `__slots__`中暴露的属性，而不是访问全部属性**。这对于一个拥有大量属性的类而言，十分节省内存，封装性也是最高的。

```python
class Person:
    __slots__ = ['name', 'age']
    # 表示 Person 实例对象只能有 name 和 age 属性，外部也无法添加额外属性
    # 并且，它会取代 __dict__ 属性，用来限制外部访问的属性集

    def __init__(self, name, age):
        self.name = name
        self.age = age


person = Person('jack', 18)
person.name = 'tom'
person.age = 20

# print(person.__dict__)  # 报错，__dict__ 不存在

# 不能再为 person 实例对象添加额外属性了
# person.gender = '男'  # 报错，gender 属性不存在
```

#### 总结

| 机制             | 语法                | 访问性         | 用途                 |
| :--------------- | :------------------ | :------------- | :------------------- |
| **公开**         | `self.name`         | 完全公开       | 正常使用的接口       |
| **约定私有**     | `self._name`        | 可访问（警告） | 内部实现，子类可使用 |
| **名称修饰私有** | `self.__name`       | 名称修饰       | 避免子类意外覆盖     |
| **Property**     | `@property`         | 像属性一样     | 验证、计算属性       |
| **描述符**       | `__get__/__set__`   | 自定义         | 可复用的验证逻辑     |
| **`__slots__`**  | `__slots__ = [...]` | 限制属性       | 节省内存、限制属性   |

### 多态

在编程中，多态是指**同一个接口（方法名）可以作用于多种类型的对象，并表现出不同的行为**。

核心理念：

- **统一入口**：调用者只需要记住一个方法名（如 `run()`）
- **不同实现**：不同的对象（狗、车、程序）各自实现自己的 `run()`

根据不同的行为，Python 的多态分为 3 个层次：

- **标准多态**：重点关注继承的血缘关系
- **鸭子多态**：
- **运算符多态**：同一个运算符作用于不同对象有不同含义

#### 为什么需要多态？

多态其实是封装的**解耦利器**：

1. **解耦**：调用者不需要知道对象的具体类，只需要知道它有某个方法。
2. **可扩展性**：如果你以后想增加一个 `Bird` 类，你只需要写好 `speak()` 方法。原来的 `make_it_speak` 函数**一行代码都不用改**。

#### 标准多态

Python 中的标准多态是基于继承的多态（经典模式）。

核心概念：多个子类重写父类的同一个方法。**不同子类对象调用同一个方法时，呈现出不同的行为**。

核心要点：

- 必须要有**继承关系**，即**限制类型范围：必须是某个类的子类，它会自动进行类型转换**
- 子类必须**重写**父类的同一个方法

```python
# 标准多态：重点关注继承血缘关系。子类重写父类同一个方法，不同子类的调用呈现不同的效果
class Animal:
    def eat(self):
        print("动物吃食物")

class Dog(Animal):
    def eat(self):
        print("狗吃骨头")

class Cat(Animal):
    def eat(self):
        print("猫吃鱼")

def func(animal):
    animal.eat()

dog = Dog()
cat = Cat()
func(dog)  # 狗吃骨头
func(cat)  # 猫吃鱼
```

#### 鸭子多态

核心理念：“如果它走起路来像鸭子，叫起来也像鸭子，那么它就是鸭子。”

核心要点：

- **不关注**继承关系，**不限制**类型范围，**不会进行**类型转换，**不需要**重写方法
- **只关注**这些类中**有没有同一个具体方法**，有就代表**它们是同一种形态**
- 即完全不看血缘关系，**只看功能**

> 在 Java 中，多态依赖于**继承**（子类必须继承父类）。但在 Python 中，**只要对象有那个方法，它就能参与多态**，管你有没有继承关系。
>
> **场景**：Python 的 `len()` 函数。只要你的类实现了 `__len__` 魔术方法，不管是列表、字典还是你自己定义的类，都能用 `len()`，这就代表自定义类参与了一种形态。

```python
class Dog:
    def eat(self):
        print("狗吃骨头")

class Cat:
    def eat(self):
        print("猫吃鱼")

class Trigger:
    def eat(self):
        print("老虎吃肉")

def func(animal):
    animal.eat()

dog = Dog()
cat = Cat()
trigger = Trigger()

func(dog)  # 狗吃骨头
func(cat)  # 猫吃鱼
func(trigger)  # 老虎吃肉
```

以上代码可以看出，`Dog`、`Cat`、`Trigger` 这 3 个类都是独立的，但它们都有同一个方法 `eat`，所以它们在某种程度上就是多态。

#### 运算符多态

同一个运算符作用于不同对象有不同含义。

- **场景**：`+` 作用于整数是加法，作用于字符串是拼接，作用于列表是合并。

### 抽象

抽象是面向对象编程的四大核心特性之一，它关注**隐藏实现细节，只保留必要接口留给子类去实现（“只定义规则，不实现细节”）**。

核心理念：抽象的本质是定义一个“模具”**或者**“契约”。它告诉子类：“如果你想成为我的一员，你必须实现这些方法，但我不管你具体怎么实现。”

核心价值：

- **规范化**：确保所有子类都有一致的方法名

- **安全性**：防止由于忘记实现某个关键方法而导致的运行时错误

- **设计感**：让代码结构从“解决具体问题”提升到“定义通用框架”

抽象有 2 种实现方式：

- **手动抛出 `NotImplementedError` 异常**：提醒子类去实现这个方法
- **继承抽象类，重写抽象方法**

#### NotImplementedError 占位符

这是一个**抽象方法占位符**，用于**提醒子类必须重写某个方法**；这是一个**弱约束**，**只在调用该方法时发生子类未重写就抛出异常**。

```python
class Animal:
    def __init__(self, name):
        self.name = name

    # 这是一个抽象方法，具体实现细节由子类去完成
    def speak(self):
        raise NotImplementedError("子类必须实现speak方法")

class Dog(Animal):
    pass

dog = Dog('狗')
# dog.speak()  # 抛出异常：NotImplementedError: 子类必须实现speak方法

class Pig(Animal):

    def speak(self):
        print('哼哼哼')

pig = Pig('猪')
pig.speak()  # 哼哼哼
```

#### 抽象模块 abc

Python 默认提供了一个 **`abc` 抽象类模块**，它对外提供了 **`ABC` 抽象类** 与 **`@abstractmethod` 抽象方法装饰器**。

```python
from abc import ABC, abstractmethod
```

##### 抽象类 ABC

`ABC` 类是 Python 官方提供的 **抽象基类（Abstract Base Classes）** 机制。

核心理解：抽象类就是一个“半成品”**或者是**“带有强制性约束的蓝图”。

核心功能：要想定义一个**抽象类**，必须让**抽象类继承 `ABC` 抽象基类**，并**在抽象类中定义至少一个 `@abstractmethod` 抽象方法 或 抽象属性**。

> 注意：它是一个**强约束**，一旦抽象类继承了 `ABC` 抽象基类，**其抽象类的子类必须实现抽象方法或抽象属性**，否则会报错。

核心特性：

- 抽象类**不能被实例化**

- 抽象类中**可以包含**抽象方法，**也可以包含**普通方法（类方法、实例方法、静态方法）但**必须实现具体功能细节**

  > 注意：抽象类中的普通方法**不应该涉及到具体子类实例的属性**，应该**只用于表示一个通用逻辑**

- 抽象类**只能被继承**，表示**部分未实现的功能（抽象方法、抽象属性）由子类去实现具体细节**；

  - 继承抽象类的**子类必须实现抽象类中的所有 抽象方法 和 抽象属性**，否则该子类也是抽象类

<img src="media/抽象类的核心理念.png" style="zoom:50%;" />

```python
from abc import ABC, abstractmethod

# 抽象类（定义规则）
class Animal(ABC):

    # 抽象实例方法
    @abstractmethod
    def speak(self):
        pass

    # 抽象属性
    @property
    @abstractmethod
    def name(self):
        pass

    # 可以有具体方法（非抽象）
    def description(self):
        print("这是一个动物")

# animal = Animal() # 报错！不能实例化抽象类

# 子类继承抽象类（实现规则）
class Dog(Animal):

    def speak(self):
        print('汪汪汪')

    def run(self):
        print('跑跑跑')

    @property
    def name(self):
        if self.__dict__['_name'] is None:
            return None
        return self.__dict__['_name']

    @name.setter
    def name(self, name):
        self.__dict__['_name'] = name


dog = Dog()
dog.speak()  # 汪汪汪
dog.run()  # 跑跑跑
dog.description()  # 这是一个动物

# print(dog.name)  # 报错：KeyError: 'name'
dog.name = '狗'  # 设置属性值
print(dog.name)  # 狗
```

##### @abstractmethod 抽象方法装饰器

核心功能：**修饰一个方法为抽象方法**。

核心特点：

- 抽象方法**没有方法体，必须使用 `pass`关键字占位**，**不能在抽象类中实现具体细节**，否则它就是一个普通方法了

- 抽象方法**必须由继承抽象类的子类去实现抽象方法的具体细节**

  > 如果子类未实现抽象方法，则会报错 `TypeError: Can't instantiate abstract class Dog without an implementation for abstract method 'hello'`

- `@abstractmethod`抽象方法装饰器**可以修饰普通方法、静态方法、类方法**

  > 注意：定义抽象类方法与抽象静态方法时， **`@classmethod` 和 `@staticmethod` 必须写在 `@abstractmethod` 上面**！

<img src="media/抽象的概念理解.png" style="zoom:50%;" />

```python
from abc import ABC, abstractmethod

# 抽象类（定义规则）
class Animal(ABC):

    # 抽象实例方法
    @abstractmethod
    def speak(self):
        pass

    # 抽象类方法
    @classmethod
    @abstractmethod
    def hello(cls):
        pass

    # 抽象静态工具方法
    @staticmethod
    @abstractmethod
    def tool():
        pass
    
# 子类继承抽象类（实现规则）
class Dog(Animal):

    def speak(self):
        print('汪汪汪')

    def hello(cls):
        print('你好。我是狗')

    def tool():
        print('工具方法')
        
        
dog = Dog()
dog.speak()  # 汪汪汪
dog.hello()  # 你好。我是狗
Dog.tool()  # 工具方法
```

##### @property + @abstractmethod 抽象属性

抽象属性指的是抽象类中通过**`@property` + `@abstractmethod`** 两个装饰器结合在一起，**共同修饰**定义的**抽象属性（方法）**。

核心理念：

1. 先**将普通方法通过 `@property` 装饰为属性**，让**方法像属性一样使用**
2. 再通过**`@abstractmethod` 装饰器将该属性（方法）修饰为抽象方法**
3. 最后**让子类去实现这个抽象属性（方法）**的具体细节。

```python
from abc import ABC, abstractmethod

# 抽象类（定义规则）
class Animal(ABC):

    # 抽象属性（getter）
    @property
    @abstractmethod
    def name(self):
        pass
    
    # 抽象属性（setter）
    @name.setter
    @abstractmethod
    def name(self, value):
        pass
    
# 子类继承抽象类（实现规则）
class Dog(Animal):
    @property
    def name(self):
        if not self.__dict__['name']:
            return None
        return self.__dict__['name']

    @name.setter
    def name(self, value):
        self.__dict__['name'] = value
       
dog = Dog()

# print(dog.name)  # 报错：KeyError: 'name'
dog.name = '狗'  # 设置属性值
print(dog.name)  # 狗
```

### 顶层类型

在 Python 中，有 2 个顶层类型 **`object` 基类** 和 **`type` 元类**。

核心概念：`object` 基类 与 `type` 元类是**两个互相依赖的顶层类型**。*就像 JavaScript 的 Function 类 与 Object 类*

#### type 元类

元类是 Python 中**最深奥**但也最强大的特性之一。简单来说：**类也是对象，而元类就是创建类的“类”**。

核心概念：

- 一切皆对象。**类也是对象。所有类（包括 `object` 类、`function` 函数）的类型都是 `<class 'type'>`**

```python
# 类的类型是 type
print(type(int))  # <class 'type'>
print(type(float))  # <class 'type'>
print(type(str))  # <class 'type'>
print(type(bool))  # <class 'type'>
print(type(bytes))  # <class 'type'>
print(type(list))  # <class 'type'>
print(type(tuple))  # <class 'type'>
print(type(set))  # <class 'type'>
print(type(dict))  # <class 'type'>
print(type(object))  # <class 'type'>

class Student:
    pass

student = Student()
print(type(Student))  # <class 'type'>



# 实例的类型是类本身
print(type(student))        # <class '__main__.Student'>
print(type(123))        # <class 'int'>
print(type(1.26))        # <class 'float'>
print(type('str'))        # <class 'str'>
print(type(True))        # <class 'bool'>
print(type(None))        # <class 'NoneType'>
print(type(b"abc"))        # <class 'bytes'>
print(type([1, 2, 3]))        # <class 'list'>
print(type((0, 1)))        # <class 'tuple'>
print(type({'A', 'B'}))        # <class 'set'>
print(type({'name': 'abc'}))        # <class 'dict'>

# 注意：函数也是一种类型！
def my_func():
    pass
print(type(my_func))  # <class 'function'>
```

**核心理解：**

- **`type` 是所有类**的**元类**（metaclass）
- **类 = `type` 的实例**
- **对象 = 类的实例**

##### 基本定义

**元类是创建类的“类”。** 就像**类定义了实例的行为，元类定义了类的行为**。

```python
# 普通关系：
# 元类 (type) → 创建 → 类 (Dog) → 创建 → 实例 (dog)

# 类比理解：
# - 蛋糕模具（元类）→ 制作 → 蛋糕模具（类）→ 制作 → 蛋糕（实例）
# - 但这里的"蛋糕模具"本身也是用更大的"模具制作机"制作的
```

关系链：

```python
class Animal:
    pass

dog = Animal()

# 关系链
print(type(dog))        # <class '__main__.Animal'>  (dog 的类是 Animal)
print(type(Animal))     # <class 'type'>            (Animal 的类是 type)
print(type(type))       # <class 'type'>            (type 的类是自己)
```

###### 层次关系

```
┌─────────────────────────────────────────┐
│  元类（type 的子类）                      │
│  - 控制类的创建                          │
│  - 修改类的行为                          │
└──────────────┬──────────────────────────┘
               │ 创建
               ▼
┌─────────────────────────────────────────┐
│  类（type 的实例）                       │
│  - 定义属性和方法                        │
│  - 作为实例的模板                        │
└──────────────┬──────────────────────────┘
               │ 实例化
               ▼
┌─────────────────────────────────────────┐
│  实例（类的实例）                        │
│  - 实际数据                              │
│  - 执行业务逻辑                          │
└─────────────────────────────────────────┘
```

##### 创建类的两种方式

###### class 关键字-静态创建

这是普通应用开发者最常用的方式：

```python
class Dog:
    species = "Canine"
    
    def bark(self):
        return "Woof!"

dog = Dog()
print(dog.bark())  # Woof!
```

###### type() 函数-动态创建

核心概念：`type()` 函数除了可以返回实例、类的类型之外，还可以用来**根据一个模板来动态创建一个类**。

语法：

```python
type(class_name, bases, dict)
```

参数：

- **`class_name`：类名**
- **`bases`：要继承的类元组（可以有多个）**
- **`dict`：属性、方法集**

```python
# 父类
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "Some sound"

dog = type('Dog', (Animal,), {'name': 'dog', 'woof': lambda x: print('woof', x)})

print(dog.__dict__['name'])  # dog
print(dog.__dict__['woof']('dog'))  # woof dog

print(type(dog))  # <class 'type'>
print(issubclass(dog, object))  # True，dog 是 object 的子类
print(isinstance(dog, type))  # True，dog 是 type 的实例
print(dog.__bases__)  # (<class 'object'>,) dog 的直接父类是 object
```

###### 自定义元类

在定义类时，**显式继承 `type` 来创建自定义元类**：

```python
class MyMeta(type):
    """自定义元类"""
    
    def __new__(cls, name, bases, dct):
        """创建类时调用"""
        print(f"创建类: {name}")
        print(f"父类: {bases}")
        print(f"属性: {list(dct.keys())}")
        
        # 自动添加属性
        dct['created_at'] = "2024-01-01"
        
        # 检查是否必须有文档字符串
        if '__doc__' not in dct or not dct['__doc__']:
            raise TypeError(f"类 {name} 必须有文档字符串")
        
        # 创建类
        return super().__new__(cls, name, bases, dct)
    
    def __init__(cls, name, bases, dct):
        """初始化类"""
        print(f"初始化类: {name}")
        super().__init__(name, bases, dct)
    
    def __call__(cls, *args, **kwargs):
        """创建实例时调用"""
        print(f"创建 {cls.__name__} 的实例")
        return super().__call__(*args, **kwargs)

# 使用元类
class MyClass(metaclass=MyMeta):
    """这是有文档的类"""
    pass

# 输出：
# 创建类: MyClass
# 父类: ()
# 属性: ['__module__', '__qualname__', '__doc__']
# 初始化类: MyClass

obj = MyClass()  # 创建 MyClass 的实例
```

###### 元类的调用链

```python
class TraceMeta(type):
    def __new__(cls, name, bases, dct):
        print(f"1. __new__: 创建类 {name}")
        return super().__new__(cls, name, bases, dct)
    
    def __init__(cls, name, bases, dct):
        print(f"2. __init__: 初始化类 {name}")
        super().__init__(name, bases, dct)
    
    def __call__(cls, *args, **kwargs):
        print(f"3. __call__: 创建实例")
        instance = super().__call__(*args, **kwargs)
        print(f"4. 实例创建完成")
        return instance

class MyClass(metaclass=TraceMeta):
    def __init__(self, value):
        print(f"5. __init__: 初始化实例")
        self.value = value

# 创建类时输出 1,2
# 创建实例时输出 3,4,5
obj = MyClass(10)
```

##### 总结

描述：**元类是 Python 中用于创建和定制类的“类”，它允许在类被创建时拦截并修改类的行为，是框架级开发的核心工具。**

#### object 基类

##### 核心概念

1. 在 Python 中，万物皆对象，都由类来构造。而**所有的类（自定义类、内置类、模块类）都继承自 `object`类**。也就是说`object` 是 Python 中**所有类的基类**，是类层次结构的**根**
2. 因为 `object` 类是所有类的祖先类，所以 **Python 中的所有对象，都间接是 `object` 类的实例**
3. **`object` 的类型是 `<class 'type'>`**

核心价值：**`object` 类自身提供的各种属性、方法保证了 Python 中所有对象都具备了统一的基本能力（属性、方法）**。

```python
# object 是所有类的终极父类
class MyClass: # 等价于 class MyClass(object): 这是隐式的，Python 内部会对所有类都实现继承 object 类的编码
    pass

print(issubclass(MyClass, object))  # True
print(issubclass(int, object))      # True
print(issubclass(str, object))      # True
print(issubclass(list, object))     # True
print(issubclass(dict, object))     # True

# 即使是自定义类，也隐式继承 object
print(MyClass.__bases__)  # (<class 'object'>,)
```

###### \__bases__ 查看直接父类

Python 为所有**类**都提供了 **`__bases__` 魔术属性**可以用于**查看类的直接父类元组**。

```python
class Person:
    pass

class Student(Person):
    pass

print(Student.__bases__) # (<class '__main__.Person'>,)
print(Person.__bases__) # (<class 'object'>,)
print(object.__bases__) # ()
```

###### 提供的核心方法

`object` 类提供了如下核心方法，使得所有对象都具备这些基本能力：

```python
# 查看 object 的所有方法
print(dir(object))

# 输出（Python 3.11）：
# ['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', 
#  '__format__', '__ge__', '__getattribute__', '__getstate__', 
#  '__gt__', '__hash__', '__init__', '__init_subclass__', 
#  '__le__', '__lt__', '__ne__', '__new__', '__reduce__', 
#  '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', 
#  '__str__', '__subclasshook__']

# 每个方法的说明：
class ObjectMethodExplainer:
    """
    主要方法说明：
    - __new__(cls, *args, **kwargs): 创建实例（类方法）
    - __init__(self, *args, **kwargs): 初始化实例
    - __del__(self): 析构函数
    - __repr__(self): 开发者的字符串表示
    - __str__(self): 用户的字符串表示
    - __eq__(self, other): 相等比较 (==)
    - __ne__(self, other): 不等比较 (!=)
    - __lt__(self, other): 小于比较 (<)
    - __le__(self, other): 小于等于比较 (<=)
    - __gt__(self, other): 大于比较 (>)
    - __ge__(self, other): 大于等于比较 (>=)
    - __hash__(self): 哈希值（用于字典/集合）
    - __getattribute__(self, name): 属性访问
    - __setattr__(self, name, value): 属性赋值
    - __delattr__(self, name): 属性删除
    - __getattr__(self, name): 访问不存在的属性
    - __call__(self, *args, **kwargs): 实例作为函数调用
    - __len__(self): len() 函数
    - __getitem__(self, key): 索引访问
    - __setitem__(self, key, value): 索引赋值
    - __delitem__(self, key): 删除索引
    - __iter__(self): 迭代器
    - __next__(self): 下一个元素
    - __contains__(self, item): in 运算符
    - __enter__(self): 上下文管理器进入
    - __exit__(self, exc_type, exc_val, exc_tb): 上下文管理器退出
    """
    pass
```

##### object 的类型

- **`object` 本身也是一个类，它是 `type` 元类的实例**。

```python
# object 本身是一个类
print(type(object))  # <class 'type'>

# object 是 type 的实例
print(isinstance(object, type))  # True

# object 的基类为空（它是根类）
print(object.__bases__)  # ()
```

##### object() 构造函数

Python 提供了 **`object()` 内置函数**，可用于**创建一个空对象**，但是该对象**不能添加属性**。

使用场景：因为 `object()` 是创建一个实例对象，会**返回一个唯一的内存地址**，所以它常用于**作为哨兵值（唯一标识）**。

```python
empty = object()
print(empty)  # <class 'object' at 0x0000020A7E5F5E80>
print(type(empty))  # <object 'object'>

# 但 object 实例不能添加属性
try:
    empty.name = "test"
except AttributeError as e:
    print(e)  # 'object' object has no attribute 'name'
```

想要创建一个能添加属性的空对象，直接用 `class` 定义一个 `pass` 待实现的类即可：

```python
# 创建可添加属性的空对象
class Namespace:
    pass

ns = Namespace()
ns.name = "test"    # ✅ 可以添加属性
ns.value = 100
print(ns.name, ns.value)  # test 100
```

##### 总结

| 方面               | 说明                                                       |
| :----------------- | :--------------------------------------------------------- |
| **本质**           | Python 所有类的根基类                                      |
| **类型**           | `type` 的实例，本身是类                                    |
| **位置**           | 类层次结构的顶层                                           |
| **提供的方法**     | `__init__`, `__str__`, `__repr__`, `__eq__`, `__hash__` 等 |
| **继承关系**       | 所有类都直接或间接继承 `object`                            |
| **与 `type` 关系** | `type` 继承 `object`，`object` 是 `type` 的实例            |
| **实例特点**       | `object()` 创建的对象不能添加属性                          |

描述：**`object` 是 Python 中所有类的终极父类，它为所有对象提供了最基础的默认行为，是类层次结构的根，但它本身不是元类——它的类型是 `type`。**

#### object 基类 vs type 元类

在 Python 中：

- **一切类都是由 `type` 元类构造的（所有类都是 `type` 元类的实例）**
- **一切类都是继承自 `object` 基类（所有类都是 `object` 基类的子类，所有类的实例也是 `object` 基类的实例）**

核心区别：

- **`object` 是类（所有类的基类，定义了类的属性、方法等行为）**
- **`type` 是元类（所有类的类型，定义了类的结构）**

##### 对象模型

核心概念：**`object` 基类 与 `type` 元类是相互循环的依赖关系、互相绑定**。它们之间既有**继承关系**，又有**实例化关系**，形成了一个环状逻辑【**“闭环的自省系统”**】。*类似于 JavaScript 的 Function 类与 Object 类*

在 Python 中，**事先预设**了两个根源上的顶层类型： **`type` 元类** 和 **`object` 基类**。

- **`object` 基类：**
  - **定义了所有对象的基础魔术属性、魔术方法** 
  - 同时它又是**由 `type` 类创建出来的**
  - 它是体系结构的**水平**终点 (所有东西都是 object)
  - 想查**“这个类有什么功能”**，顺着 `__bases__` 往上找，终点是 `object`
- **`type` 元类**：
  - **定义了类的结构**；负责创建“类对象”，控制类的创建行为
  - 同时**它又继承自 `object` 基类**，所以它**也具备对象的特性**
  - 它是体系结构的**垂直**起点 (所有东西都由 type 创建)
  - 想查**“这个类是怎么造出来的”**，顺着 `__class__` 往上找，终点是 `type`

###### 具体过程

在 Python 解释器启动时，`type` 和 `object` 是**同时硬编码**产生的，没有先后。

在 Python 中，**`type`** 作为**元类**，拥有生产一切类的能力，`object` 基类专门用于定义所有对象的属性、方法。

1、首先，**`type` 元类为自己产生了一个 `type` 实例 （自己生自己）：**

```python
print(type(type)) 
# 输出：<class 'type'>
```

> **`type` 这个关键字（也就是元类本身），它的类型还是 `type`**。它是自己的实例。

2、`type` 元类产生的 **`type` 元类实例有一个构造方法，也就是常用的 `type()`内置函数**；该构造方法除了可以检查实例类型之外，还可以**根据类模板规则来创建一个类**，所以 **`type` 元类实例通过它的构造方法 `type()` 创建了 `object` 类的实例（`object` 是 `type` 的实例）**

```python
print(isinstance(object, type)) # True
```

> 我们一直使用的 **`object` 这个关键字其实是 `type` 元类实例的构造方法 `type()` 构建的实例对象**。
>
> ```python
> object = type('object', (), {...})
> 
> class MyClass(object): # 隐式的
>  pass
> ```

3、再**由 `object` 实例继承自 `object` 基类，使它作为所有类的共同祖先类，当创建所有新类时，会默认继承这个 `object` 实例**，并持有 `object` 类的所有属性、方法等统一基本能力。

> 在 Python 的语境下，**“object 基类”** 与 **“object 类” **其实指的都是**同一个实体**：即**内置的 `object` 对象**。

4、与此同时，`type` 元类**为了让自己也具备‘对象’的基本属性**，它选择**继承**了它自己构建的 `object`类。（`type` 是 `object` 的子类）

```python
class type(object): # 继承自 object ，拥有对象的特性
    __class__ = type  # 自己生自己
    pass

print(issubclass(type, object))  # True type 又继承了 object 类
print(isinstance(object, type))  # True object 又是 type 的实例

print(type.__mro__)  # (<class 'type'>, <class 'object'>)
print(type.__bases__)  # (<class 'object'>,) type 的直接父类是 object 类
print(type.__class__)  # <class 'type'> type 的类型是 type
```

5、这样一来，当我们定义任何新类时，这个类既会**继承 `object`**（获得基础方法），又是由 **`type` 生产**出来的（获得创建实例的能力）。这两者相辅相成，共同构成了‘万物皆对象’的闭环体系。”

##### 总结

​	**`type` 元类自己产生了一个 `type` 实例对象（我们常用的 `type` 关键字）**，该实例对象**通过 `type()` 方法构造了 `object` 类，同时设定相关的属性、方法（我们常用的 `object` 关键字）**；

​	并且 Python 会在每次**创建新类**时，**默认添加 `class MyClass(object)` 去继承 `object` 类**，使得所有类都具有统一的属性、方法等基本能力；

​	之后，`type` 元类自身为了持有对象的特性，也**继承了它自己创建的 `object` 类**，这样一来，形成了一个闭环流程。

​	**`type` 元类用来让类拥有实例化能力，`object` 基类让类持有 “继承链” 查找机制**。





<img src="media/type元类与object基类的关系.png" style="zoom:50%;" />

```python
		  继承 (Inheritance)
    type  ---------------->  object
     ^ |                      ^ |
     | |      实例化 (Instance) | |
     +-+ <--------------------+ +-+
   (自己生自己)              (由 type 生)
```

这种“鸡生蛋、蛋生鸡”的闭环设计，保证了语言体系内部的自洽性：**既保证了万物皆有基类可寻，又保证了万物皆有元对象可造。**