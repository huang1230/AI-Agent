# Pydantic 数据模型

> 官网中文文档：https://pydantic.com.cn/

FastAPI 充分利用了 Pydantic 的**数据模型（`BaseModel` 类）**来处理 API 的 **数据字段的值校验、请求/响应数据的 `JSON` 格式转换（序列化/反序列化）** 以及 **Swagger UI 自动化 API 文档生成**。

核心要点：Pydantic 的**数据模型（`BaseModel`）**负责定义**数据的 “形状”（Schema 结构）**，而 FastAPI 负责把这个 **数据模型（`BaseModel`） 应用到 HTTP 请求 和 响应 数据中**（以 **`@ 路由装饰器` + `async def` API 接口函数的参数列表** 组合形式）。

## 核心特性

- **类型验证**：只需要使用标准的 Python 语法 `name: str` 或 `age: int` 来**定义数据模型**，Pydantic 就能**自动解析这些注解并生成验证逻辑，无需额外编写校验代码**
- **错误提示**：当**数据验证失败**时，Pydantic 会**抛出 422 `ValidationError` 异常**，提供结构化的错误详情，精准定位问题字段和原因，便于快速调试
- **灵活的序列化**：
  - 可以轻松地**将模型实例转换为字典 (`model_dump()`) 或 JSON 字符串 (`model_dump_json()`)**
  - 反之亦可**将外部数据源（如 API 响应、JSON 文件）加载为模型实例**
- **数据解析**：将**输入数据（如 JSON）转换为 Python 对象**
- **默认值与可选字段**：支持**为字段设置默认值**或**标记为可选**

## BaseModel 类

在 Pydantic（以及 FastAPI）中，**`BaseModel`** 是定义**所有数据模型的基类**。可以把它看作是一个升级版的 Python 原生 `dataclass`，但它自带了极其强大的**数据校验（Validation）**和 **类型转换（Parsing）**功能。

### 基础定义

核心机制：

1. 创建一个**继承自 `BaseModel` 的派生类**，表示 **API 接口**的**请求体、查询参数、响应体**所**接收**的**数据模型（Data Model）**

2. 当**类属性（数据字段）**使用 Python 原生的 **类型提示（Type Hints）来声明类型**时，便会触发 Pydantic 的**自动类型转换**

| **编程概念 (Python 类)**            | **对应 Web 概念 (HTTP 传输)** | **实际生活比喻**                         |
| ----------------------------------- | ----------------------------- | ---------------------------------------- |
| **`BaseModel` 派生类**              | **数据模型 (Model)**          | **表单/契约的模板**（规定好必须填什么）  |
| **类属性 (Class Attributes)**       | **数据字段 (Fields)**         | **表单上的具体格子**（姓名、年龄、邮箱） |
| **类属性的类型提示 (`str`, `int`)** | **数据类型验证**              | **格子的硬性要求**（年龄必须填数字）     |

```python
from pydantic import BaseModel
from typing import Optional


# 定义一个数据模型类
class User(BaseModel):
    # 数据字段: 数据类型 = 默认值

    # 必填字段（未指定默认值）
    id: int
    name: str

    # 选填字段（带有默认值）
    address: Optional[str] = None
    is_active: bool = True
```

#### 传值-类型转换与校验

在类实例化时，Python 支持 2 种方式传入类的实例属性：

- **位置参数 | 关键字参数**：

  ```python
  user = User(id='1', name='Tom', address='Beijing', is_active=False)
  print(user)  # id=1 name='Tom' address='Beijing' is_active=False
  # Pydantic 会自动将 id 转换为 int 类型
  ```

- **`**` 解包传值**：

  ```python
  external_data = {'id': 1, 'name': 'Tom', 'address': 'Beijing', 'is_active': False}
  
  user2 = User(**external_data)
  print(user2)  # id=1 name='Tom' address='Beijing' is_active=False
  ```

#### **自动类型转换**

Pydantic 不仅会检查类型，还会**尽最大努力将数据转换为正确的类型**。

- 当传入 `id='1'` 时，Pydantic 会自动将 `'1'` 转为 `1`（int 类型）
- 当传入 `is_active="false"` 或 `is_active=0`，Pydantic 会自动转换为布尔值 `False`

### 基础使用

#### 序列化输出数据

- **`module_dump()`**：将**数据模型的内容**以 **`{}` 字典**形式**输出**

  ```python
  user4 = User(id=1, name='Tom', address='Beijing', is_active=False)
  
  print(user4.model_dump())  # 获取当前数据模型的字典形式
  # {'id': 1, 'name': 'Tom', 'address': 'Beijing', 'is_active': False}
  ```

- **`module_dump_json()`**：将**数据模型的内容**以 **JSON 字符串** 格式**输出**

  ```python
  user4 = User(id=1, name='Tom', address='Beijing', is_active=False)
  
  print(user4.model_dump_json())  # 获取当前数据模型的 JSON 字符串形式
  # {"id":1,"name":"Tom","address":"Beijing","is_active":false}
  ```

#### 异常处理 e.json()

当传入了错误的数据，Pydantic 会直接抛出 **`pydantic.ValidationError`** 异常，而不会继续带着脏数据继续执行代码。

> 可以通过 **`try-except` 异常处理**，接收到具体的**异常对象 `e`**，可通过 **`e.json()`** 输出具体的**异常数据字段、异常原因**...

```python
from pydantic import BaseModel, ValidationError


# 异常处理
try:
    # 传入的错误的数据 id='a'，不是 int 类型
    user3 = User(id='a', name='Tom', address='Beijing', is_active=False)
except ValidationError as e:
    print(e.json(indent=2))  # 打印格式化后的异常信息

    # [
    #     {
    #         "type": "int_parsing", # 错误类型
    #         "loc": [  # 错误的数据字段
    #         	"id"
    #         ],
    #         "msg": "Input should be a valid integer, unable to parse string as an integer", # 错误信息
    #         "input": "a", # 输入的错误数据
    #         "url": "https://errors.pydantic.dev/2.13/v/int_parsing"
    #     }
    # ]
```

错误信息：

- **`type`：错误类型**
- **`loc`：错误的数据字段**
- **`msg`：错误信息**
- **`input`：输入的错误数据**

#### model_copy() 拷贝数据模型

作用：将当前 Pydantic 类实例的数据模型拷贝一份副本并返回。

```python
user4 = User(id=1, name='Tom', address='Beijing', is_active=False)

# 浅拷贝复制一个全新的 User 类的实例，与原实例无任何关联，可以使用 deep=True，转为深拷贝
user_copy = user4.model_copy(deep=True)
print(user_copy)  # id=1 name='Tom' address='Beijing' is_active=False
```

#### model_json_schema() 返回数据模型的 JSON Schema 说明

> JSON Schema：本身也是一个 JSON 文件，用来**定义和校验**其他 **JSON 数据的结构、内容和规则**。
>
> ```json
> {
> "$schema": "https://json-schema.org/draft/2020-12/schema",
> "title": "User",
> "type": "object",
> "properties": {
>  "name": {
>    "type": "string",
>    "description": "用户的名字"
>  },
>  "age": {
>    "type": "integer",
>    "minimum": 0,
>    "description": "用户的年龄，必须大于或等于 0"
>  }
> },
> "required": ["name"]
> }
> ```

```python
user4 = User(id=1, name='Tom', address='Beijing', is_active=False)

user_json_schema = user4.model_json_schema()  # 获取当前数据模型的 JSON Schema 模型
print(user_json_schema)


# {'properties': {'id': {'title': 'Id', 'type': 'integer'}, 'name': {'title': 'Name', 'type': 'string'}, 'address': {'anyOf': [{'type': 'string'}, {'type': 'null'}], 'default': None, 'title': 'Address'}, 'is_active': {'default': True, 'title': 'Is Active', 'type': 'boolean'}}, 'required': ['id', 'name'], 'title': 'User', 'type': 'object'}
```

### 数据模型嵌套

继承自`BaseModel` 的派生类可以作为另一个 `BaseModel` 类的字段模型，用来处理复杂的层级结构。

```python
from pydantic import BaseModel
from typing import Optional
from enum import Enum

# 嵌套模型
class Address(BaseModel):
    city: str
    street: str


# 枚举类型
class SexEnum(str, Enum):
    man = 'man'
    woman = 'woman'


class Student(BaseModel):
    id: int
    name: str
    age: int = 0
    sex: SexEnum = SexEnum.man

    address: List[Address] = [] # 嵌套 Address 模型列表


data = {
    'id': 1,
    'name': 'Tom',
    'age': 18,
    'sex': 'man',
    'address': [
        {
            'city': 'Beijing',
            'street': 'Chaoyang Road'
        }
    ]

}

student = Student(**data)

print(student.model_dump_json())
# {"id":1,"name":"Tom","age":18,"sex":"man","address":[{"city":"Beijing","street":"Chaoyang Road"}]}
```

### model_config 模型配置

可以通过 **`model_config` 属性**来**调整模型的行为**；例如不允许未知字段传入，或者支持 ORM 模型：

```python
class Item(BaseModel):
    name: str
    price: float

    model_config = {
        "extra": "forbid",          # 如果传入模型中没有定义的字段，直接报错
        "str_strip_whitespace": True # 自动去除字符串前后的空格
    }
```

#### model_config 输出模型配置

作用：可以输出**当前数据模型实例**内的**配置项**。

```python
print(student.model_config)
# {'extra': 'forbid', 'str_strip_whitespace': True}
```

#### 总结：快速记忆

- **定义**：继承 `BaseModel`，写好类型提示，用 `=` 赋默认值，用 `Field()` 加校验规则
- **输入**：`Model(dict_data)` 实例化，不合规直接报错 `ValidationError`
- **输出**：`.model_dump()` 转字典，`.model_dump_json()` 转 JSON

## 数据校验与元数据

在 Pydantic 中，数据校验（Validator）是其最核心的功能。

Pydantic 提供了以下 6 种灵活的校验方式来验证数据的输入/输出，从最基础的**类型提示**到强大的**自定义校验器**。

| **校验方式**           | **适用场景**                                          |
| ---------------------- | ----------------------------------------------------- |
| **类型提示**           | 基础的数据类型强制与转换（如 `str` -> `int`）         |
| **`Field()`**          | 简单的范围限制（大小、长度、正则匹配）                |
| **`@field_validator`** | 单个字段的复杂业务逻辑校验                            |
| **`@model_validator`** | 涉及多个字段联动、依赖关系的校验                      |
| **`TypeAdapter`**      | 校验非类对象（如标准库中的 `list`, `dict`, `set` 等） |
| **`@validate_call`**   | 规范函数入参，防止脏数据进入函数体                    |

### 类型提示

这是 Pydantic 最基础、最常用的校验方式。只需要声明 Python 标准的类型提示，Pydantic 会在初始化时自动强制执行**类型检查**和**数据转换（Coercion）**。

```python
class Admin(BaseModel):
    id: int          # 必须能转换为整数
    name: str        # 必须能转换为字符串
    is_active: bool  # 必须能转换为布尔值


# 自动类型转换：字符串 "123" 会被转为整数 123，"true" 会被转为 True
admin = Admin(id="123", name="Alice", is_active="true")
print(admin)  # id=123 name='Alice' is_active=True
```

### Field 类

Pydantic 提供了一个 **`Field` 类**（数据字段），可以针对 **`BaseModel` 数据模型类**中的**单个或多个数据字段**进行更复杂的**业务约束**（如数值范围、字符串长度、正则匹配等）。

#### 基础语法

```python
Field(
	default: Any = Undefined, # 静态默认值
    default_factory: (() -> Any) | None = _Unset, # 动态默认值（工厂）

    # 长度限制，适用于 str、list\dict\set\tuple 等序列容器数据类型
    min_length: int | None = None, # 最小长度
    max_length: int | None = None, # 最大长度
    
    # 数值范围验证，适用于 int、float 数值类型
	gt: float | None = None, # 大于 ＞
    ge: float | None = None, # 大于等于 ≥
    lt: float | None = None, # 小于 ＜
    le: float | None = None, # 小于等于 ≤
    
    # 正则表达式校验
    pattern: str | None = None,
    regex: str | None = None,
    
    # 其他（Swagger UI 文档标注）
    alias: str | None = None, # 别名
    description: str | None = None, # 参数描述信息
    deprecated: bool | None = None, # 是否已被废弃
    #...
)
```

#### 数据校验

当外部给 `BaseModel` 类实例传递实例属性值时，会自动触发以下参数的校验处理。

##### default 静态默认值

```python
# 定义一个Product类，继承自BaseModel
# 这是一个数据模型类的定义，通常用于数据验证和序列化
class Product(BaseModel):
    name: str = Field(...)  # ... 代表没有默认值，必填字段
    price: float = Field(default=8.99)  # 默认值为8.99，选填字段


product = Product(name='Apple')
print(product.model_dump())  # {'name': 'Apple', 'price': 8.99}
```

- **`default`：**
  - **`...`：一个占位符；表示没有默认值；必传字段**

注：当要**使用 `Field()` 为 `BaseModel`  的某个数据字段进行添加额外验证处理**时，必须**传入 `...` 或一个默认值** 给 **`Field()`  第一个位置参数**，否则会报错 `pydantic_core._pydantic_core.ValidationError` 异常。

##### default_factory 动态默认值

*defualt 和 default_factory 是互斥的，必须二者选其一*

核心作用：`default_factory` 参数支持**传入一个 `def` 函数、`lambda` 匿名函数**，用于**为数据字段动态生成一个默认值（工厂模式）**。

> 注意：`default_factory` 参数**所传入的函数返回的值必须与 `<数据类型>` 一致**，否则会抛出 `ValidationError` 异常。

```python
class Product(BaseModel):
    desc: str = Field(default_factory=lambda: 'Default Value')
```

##### 长度限制

作用：适用于 **`str` 字符串、`list\dict\set\tuple` 等序列容器数据类型**的**长度限制**。

```python
class Product(BaseModel):
    # 字段长度在3到10之间，超出或小于范围会报错 pydantic_core._pydantic_core.ValidationError
    desc: str = Field(..., min_length=3, max_length=10)
```

##### pattern 正则表达式验证

作用：适用于 **`str` 、`int` 等基本类型**数据，仅接受**完全匹配**的输入。

```python
class Product(BaseModel):
    # 正则表达式校验，不符合正则表达式会报错
    code: str = Field(..., pattern="^[A-Z]{2}-[0-9]{3}$")  # 例如：AB-123

product = Product(code='AB-123')
```

规则：当输入 `/pattern_item/a` 时校验通过，若 `a` 为 `A` 大写字母则校验失败，抛出 422 异常。

##### gt\lt ge\le 数值范围验证

作用：适用于 **`int`、`float` 等数值类型**的数据，用于**限制传入的查询参数值的数值范围**。

```python
class Product(BaseModel):
    size: int = Field(default=10, ge=0, le=100)  # size 必须在 0 到 100 之间

    rating: int = Field(0, gt=5)  # rating 必须大于 5
    # gt: greater than 大于
    # ge: greater than or equal 大于等于
    # lt: less than 小于
    # le: less than or equal 小于等于


product = Product(size=5, rating=6)
```

规则：当 `/user?age=xx` 的 `age` 值低于 18 或 超过 60 时，抛出 422 异常。

##### Enum 枚举类校验

核心定义：为 **可变 URL 路径参数** 定义**多个可选枚举项**，必须**选择其一**，否则会抛出 422 异常。

- 需事先提前定义**一个 `枚举类` （继承自 `enum.Enum`）**，并将**某个 可变 URL 路径参数 的数据类型**设为**该 `枚举类`**。

```python
from enum import Enum

class GoodPlace(str, Enum):
    beijing = "beijing"
    shanghai = "shanghai"
    
class Product(BaseModel):
    # good 必须为 beijing 或 shanghai，否则报错
    good: GoodPlace = Field(default=GoodPlace.beijing)


product = Product(good=GoodPlace.shanghai)

print(product.model_dump(mode='python'))
# python: {'good': <GoodPlace.beijing: 'beijing'>}

print(product.model_dump(mode='json'))
# json: {'good': 'beijing'}
```

#### 元数据

Pydantic 的 Field 类可以为 **`BaseModel` 数据模型**中的任意**数据字段**添加**元数据描述信息**，用于**生成文档、代码提示以及 API 架构描述（JSON Schema / OpenAPI）**。

当在 FastAPI 或其他支持 JSON Schema 的框架中使用 Pydantic 时，这些属性会直接决定 API 文档（Swagger UI）有多清晰、多好用。

##### title 标题

**作用**：为字段提供一个更具人类可读性的**名称**。

**默认行为**：如果不设置，Pydantic 会默认将字段名转换为首字母大写的形式（例如：`user_name` 变为 `User Name`）。

```python
from pydantic import BaseModel, Field

class Product2(BaseModel):
    sku_id: str = Field(
        ...,
        title="Stock Keeping Unit ID",
        pattern='^PROD-[0-9]{5}$',
        description="商品的库存最小存货单位编码，必须以 PROD- 开头",
        examples=["PROD-10293", "PROD-88472"]
    )

    price: float = Field(
        ...,
        gt=0,
        title="商品单价",
        description="人民币结算的商品销售单价，最多保留两位小数",
        examples=[99.00, 1999.50]
    )


product = Product2(sku_id="PROD-10293", price=99.00)
```

##### description 字段描述

**作用**：对字段的用途、业务规则、注意事项进行详细的**文本说明**。

```python
from pydantic import BaseModel, Field

class Product2(BaseModel):
    sku_id: str = Field(
        ...,
        title="Stock Keeping Unit ID",
        pattern='^PROD-[0-9]{5}$',
        description="商品的库存最小存货单位编码，必须以 PROD- 开头",
        examples=["PROD-10293", "PROD-88472"]
    )

    price: float = Field(
        ...,
        gt=0,
        title="商品单价",
        description="人民币结算的商品销售单价，最多保留两位小数",
        examples=[99.00, 1999.50]
    )


product = Product2(sku_id="PROD-10293", price=99.00)
```

##### examples 示例

**作用**：提供一个或多个符合该字段要求的**示例数据**。

**使用场景**：在交互式文档中，这些示例会自动填充到请求体（Request Body）中，方便调用者一键点击 "Try it out" 进行测试，而不需要自己手打模拟数据。

```python
from pydantic import BaseModel, Field

class Product2(BaseModel):
    sku_id: str = Field(
        ...,
        title="Stock Keeping Unit ID",
        pattern='^PROD-[0-9]{5}$',
        description="商品的库存最小存货单位编码，必须以 PROD- 开头",
        examples=["PROD-10293", "PROD-88472"]
    )

    price: float = Field(
        ...,
        gt=0,
        title="商品单价",
        description="人民币结算的商品销售单价，最多保留两位小数",
        examples=[99.00, 1999.50]
    )


product = Product2(sku_id="PROD-10293", price=99.00)
```

### 校验方法

Pydantic 提供了 **`@field_validator`（针对单个或多个特定字段）** 和 **`@model_validator`（针对整个对象/多字段联动）** 两个装饰器。

核心价值：为了避免**单个数据字段（Field）使用过多的 `Field()` 参数**而导致代码**臃肿**。

核心机制：

- **`Field()` 专门定义 `BaseModel` 模型字段的数据类型、简单的数据校验**
- **更复杂的校验逻辑**都**抽离到使用 `@field_validator` 和 `@model_validator`  独立方法**中，实现更**简洁**的代码结构。

> 注意点：若 **`Field()` 的校验参数**与 **`@field_validator()` 校验方法内**的**校验逻辑一致**，则**以 `Field()` 的校验参数 为准**。

#### @field_validator 装饰器（单个字段）

核心作用：针对 `BaseModel` 数据模型类中**单个或多个特定的数据模型字段（Field）**，使用 **`@field_validator` 字段级校验器**，实现**更复杂的校验逻辑**。

核心机制：**`BaseModel` 类中的校验方法**必须设定为一个 **`@classmethod` 类方法**，且 **`@field_validator` 必须写在 `@classmethod` 前面**。

##### 基本语法

```python
form pydantic import BaseModel, field_validator

class <数据模型类>(BaseModel):
    <数据字段>: <数据类型> = <默认值 | Field() 校验>
    
    @field_validator(<数据字段1>,<数据字段2>,...)
    @classmethod
    def <字段数据校验函数>(cls, <数据字段的值>: <数据类型>) -> <返回值类型>:
        # 校验逻辑...
        
        return <返回值> # 一旦校验逻辑通过，则 return 的返回值会赋值给 <数据字段>
```

核心价值：

- **职责分离**：属性定义归属性定义，校验逻辑归校验逻辑
- **复用方便**：如果有多个字段需要相同的校验规则（比如 `username` 和 `nickname`），只需要在装饰器里加上即可：`@field_validator("username", "nickname")`

##### 示例

```python
# @field_validator 统一的校验函数（针对单个或多个数据字段）
class UserForm(BaseModel):
    account: str = Field(..., title="用户名")  # , max_length=5
    password: str = Field(..., title="密码", min_length=8, max_length=15)

    @field_validator('account')  # 针对单个字段
    @classmethod
    def validator_account(cls, value: str) -> str:
        if len(value) < 5:
            # 这里与 Field(...,max_length=5) 冲突，则以 Field() 校验参数为准
            raise ValueError('用户名长度不能小于5')

        return value

    @field_validator('account', 'password')  # 针对多个字段
    @classmethod
    def validator_(cls, value: str) -> str:
        if not value.isalnum():
            raise ValueError('用户名/密码只能包含字母和数字')
        return value


user_form = UserForm(account='hello1234', password='1234567890')
```

#### @model_validator 装饰器（多个字段）

核心作用：当涉及到 **`BaseModel` 数据模型类**中的**多个数据字段**进行**联合校验处理**时，需要使用 **`@model_validator` 模型级校验器**。

核心机制：在 `BaseModel` 类中，**`@model_validator` 装饰器修饰的校验方法**根据 Pydantic 解析数据的阶段分别为一个**`@classmethod` 类方法**、一个**普通方法**。

##### 基本语法

```python
form pydantic import BaseModel, model_validator

class <数据模型类>(BaseModel):
    <数据字段1>: <数据类型> = <默认值 | Field() 校验>
    <数据字段2>: <数据类型> = <默认值 | Field() 校验>
    
    
    @model_validator(mode=<after | before>) # 多字段联合校验方法
    def <字段数据校验函数>(self) -> <返回值类型>:
        # 涉及到的多个联合字段校验逻辑...
        
        return <返回值>  # 一旦校验逻辑通过，则 return 的返回值会赋值给 <数据字段>
```

##### 参数

- **`mode='before'`**：在 **Pydantic 解析数据之前运行**，此时**输入的数据**还是一个**原始的 `dict` 字典**，可以进行**数据清洗**与**预处理**

  > 注：**`@model_validator` 修饰的方法**必须是一个 **`@classmethod` 类方法**
  >
  > 因为**原始数据还未被 Pydantic 解析到 `BaseModel` 类实例中，无法正常访问实例属性**。

- **`mode='after'`**（默认）：在 **Pydantic 自身的基础校验完成后运行**，此时**可以通过 `self` 访问 `BaseModel` 类中的实例属性字段**

  > 注：**`@model_validator` 修饰的方法**必须是一个**普通方法**
  >
  > 此时**数据已经校验、解析完成**，并**注入到 `BaseModel` 类实例中**，**需通过 `self` 来正常访问**

##### Pydantic 数据解析过程

> [!NOTE]
>
> 🏭 **阶段一：原始数据刚进厂（`mode='before'`）**
>
> 当外部数据（比如从前端传来的 JSON、字典或字符串）刚刚输入进来，Pydantic 还没来得及对它做任何处理（比如还没把字符串 `"123"` 转成数字 `123`）。
>
> - **数据状态**：原始的、未经处理的（通常是一个 **`dict`** 或原始值）。
> - **你的任务**：如果你想在 Pydantic 报错之前，**提前修改数据结构**、清洗脏数据、或者设置默认值，就用 `before`。
>
> 🏭 **阶段二：Pydantic 官方标准加工**
>
> Pydantic 开始干活：检查类型、把字符串转成数字、校验 `Field(max_length=5)` 等基础规则。如果类型不对（比如把 `"hello"` 转成 `int`），在这个阶段就会直接报错。
>
> 🏭 **阶段三：标准加工完成（`mode='after'`）**
>
> 此时，Pydantic 的基础校验和类型转换已经**全部安全通过**了。
>
> - **数据状态**：已经变成了一个干净的、类型正确的 **Pydantic 对象实例（`self`）**。
> - **你的任务**：如果你想基于这些已经确保准确的数据，做进一步的**业务逻辑联动校验**（例如：确保 `end_time` > `start_time`），就用 `after`。
>
> | **维度**                      | **mode='before'**                                         | **mode='after' (默认)**                                  |
> | ----------------------------- | --------------------------------------------------------- | -------------------------------------------------------- |
> | **你拿到的输入是什么**        | 原始的 `dict`（或原始输入值）                             | 已经校验通过的 `self` 对象                               |
> | **Pydantic 基础校验跑完了吗** | ❌ 还没跑，正准备跑                                        | 已经全部跑完且成功了                                     |
> | **核心目的**                  | **数据清洗/预处理**（防止 Pydantic 因为格式问题直接报错） | **业务逻辑联检**（多字段组合校验，确保数据符合业务常理） |
> | **参数形式**                  | `@classmethod`，第一个参数是 `cls`                        | 普通方法，第一个参数是 `self`                            |

##### 示例

```python
# model_validator（多个数据字段联合校验处理）
class Emplyee(BaseModel):
    name: str = Field(..., title="员工姓名")
    age: int = Field(..., title="员工年龄", ge=18, le=65)

    # --数据清洗--
    # 前端传来的数据格式不太对，传了一个：{"name": "张三", "age": " 25岁 "}。
    # 如果直接让 Pydantic 校验，" 25岁 " 没法转成整数 int，程序会直接报错崩掉。
    # 这时候我们需要在标准校验之前把数据洗干净：
    @model_validator(mode='before')
    @classmethod
    def validator(cls, data: dict) -> dict:
        # 注意：此时 data 是一个原始 dict
        if isinstance(data, dict) and "age" in data:
            if isinstance(data["age"], str):
                # 把 " 25岁 " 提取并清洗为 "25"
                data["age"] = data["age"].replace("岁", "").strip()
        return data  # 返回清洗后的字典，交由 Pydantic 继续处理

    # --数据校验--
    # 前端传来的数据格式很规范：{"name": "李四", "age": 17}。
    # Pydantic 成功把它转成了整型 17。但我们的公司不招未成年人，这属于业务逻辑逻辑错误，需要后续校验：
    @model_validator(mode='after')
    def check_business_rules(self) -> 'Emplyee':
        # 注意：此时 Pydantic 对象已经生成，你可以通过 self 访问属性
        if self.age < 18:
            raise ValueError("未满 18 岁，无法录入系统")
        return self


emplyee = Emplyee(name='张三', age=' 25岁 ')
print(emplyee.model_dump(mode='json'))  # {'name': 'Tom', 'age': 25}
```

