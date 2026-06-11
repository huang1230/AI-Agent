# HTTP 请求参数与校验

## 前置概念

### 核心概念

在 FastAPI 中，根据一个 HTTP 请求的结构拆解，`fastapi` 模块提供了 **`Path()`、`Query()`、`Body()`** 3 个类，它们都具备**包装、数据校验**和**元数据（字段标题、字段描述...）声明**的功能，但是核心设计与作用域不同：

| **函数**      | 支持的 HTTP 方法  | **它负责校验/声明什么？**      | **绑定的位置**      | **对应的 HTTP 概念**                |
| ------------- | ----------------- | ------------------------------ | ------------------- | ----------------------------------- |
| **`Path()`**  | *不限制*          | 路径参数（URL 路径的一部分）   | 函数形参            | `/items/{item_id}` 中的 `item_id`   |
| **`Query()`** | *不限制*          | 查询参数（URL 问号后面的参数） | 函数形参            | `/items?limit=10` 中的 `limit`      |
| **`Body()`**  | *POST/PUT/DELETE* | 直接从请求体中提取的参数       | 函数形参            | HTTP Request Body                   |
| **`Field()`** | ...               | Pydantic 模型内部的字段        | **Pydantic 类成员** | 嵌套在请求体/响应体 JSON 内部的属性 |

它们之间的关系：

```
			[ Pydantic: FieldInfo ]  (最底层的元信息基类)
                       │
         ┌─────────────┴─────────────┐
         ▼                           ▼
  [ Pydantic: Field ]        [ FastAPI: Param ] (内部基类)
                                     │
                    ┌────────────────┼────────────────┐
                    ▼                ▼                ▼
               [ Path ]          [ Query ]         [ Body ]
```

示例：

```python
from fastapi import Path,Query,Body

# API 接口函数使用（传统写法）
@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>/{<形参变量A>}'):
async def <API 接口函数>(
    # Path 路径参数
	<形参变量A>: <数据类型> = <默认值-包装/校验 Path(...)>
    
    # Query 查询参数
	<形参变量B>: <数据类型> = <默认值-包装/校验 Query(...)>
	<形参变量C>: <数据类型> = <默认值-包装/校验 Query(...)>
    
    # Body 请求体
   	<形参变量D>: <数据类型> = <默认值-包装/校验 Body(...)>
	<形参变量E>: <数据类型> = <默认值-包装/校验 Body(...)>
    #...
):
    # ...
    
    return <响应数据>

# API 接口函数使用（现代写法）
@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>'):
async def <API 接口函数>(<形参>: Annotated[<数据类型>, <Path()\Query()\Body()...>] = <默认值>):
    # ...
    
    return <响应数据>
```

同时，为了解决当**一个 API 接口存在多个请求参数**时会导致 API 接口函数的**形参列表过于冗余**的问题，FastAPI 支持**结合 Pydantic 的数据模型（`BaseModel`）**将**多个同类型的请求参数提取出来定义在一个 “类结构” 中定义声明**，并**统一分配到 API 接口函数的 `形参变量`  进行复用**。

> 注：**`Path()`、`Query()`、`Body()`** 是 FastAPI 提供的类，主要是用于**在 API 接口函数中的「形参列表」的单字段（`形参变量`）校验**，**校验规则完全等价于 Pydantic 的 `Field()` 类**（用于**多字段校验**）。

### Pydantic 数据模型的引入使用

​	在 FastAPI 中，Pydantic 的数据模型（`BaseModel` 派生类）规定以 **类结构** 的形式，常用来**定义 API 接口的Query 查询参数（`?k=v`）、请求体（Request Body）、响应体（Response Body）**的数据模型，底层通过 **JSON** 进行**序列化**与**反序列化**进行**自动转换处理**。

#### 核心流程

​	当一个 API 接口需要**接收多个数据字段**时，为**防止 API 接口函数的形参列表因为定义了太多的形参而导致代码臃肿**；

​	可以将**多个数据字段收集并提取出来**，额外在外部通过**使用 `pydantic.BaseModel` 提取出一个 `class` 类结构**来声明**`Query` 查询参数、请求体（Request Body）、响应体**的数据结构（Data Schema）；

​	并且**在 `BaseModel` 类**中使用 **`pydantic.Field` 类** 和 **`@field_validator` / `@model_validator` 校验方法**来**对数据字段进行参数值的校验处理**；

​	之后在 **API 接口函数的「形参列表」**中**定义一个 `形参变量`** 来**声明为一个 `BaseModel` 派生类类型**，表示**该 `形参变量` 会有多个数据字段**；

​	最后再**使用 `=` 赋予 `Query\Body\Depends...` 等类的默认值** 或者 **`:Annotated[]`** 的**现代类型声明写法**，用于**将该 `形参变量` 包装为具体的实体（查询参数、请求体...）**即可。

```python
from fastapi import Path,Query,Body
from pydantic import BaseModel,Field,Depends
from typing import Annotated

# 仅限于 Query 查询参数、请求体（Request Body）、响应体 的 JSON 数据模型，Path 可变路径参数不可用 BaseModel！
class <数据模型A>(BaseModel):
    <字段名>: <数据类型> = <默认值约束 Field()> # 数据字段
    <字段名>: <数据类型> = <默认值约束 Field()> # 数据字段
    <字段名>: <数据类型> = <默认值约束 Field()> # 数据字段
    # ...
    
    @field_validator(<字段名>)
    @classmethod
    async <校验函数名>(cls, data: <数据类型>):
        # 字段的值校验逻辑...
    
    
# API 接口函数使用（传统写法）
@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>'):
async def <API 接口函数>(<形参>: <数据模型A> = <默认值-包装/校验 Query()\Body()\Depends()...>):
    # ...
    
    return <响应数据>

# API 接口函数使用（现代写法）
@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>'):
async def <API 接口函数>(<形参>: Annotated[<数据模型A>, <Query()\Body()\Depends()...>] = <默认值>):
    # ...
    
    return <响应数据>
```

> 注：在启动 FastAPI 应用后，FastAPI 会**根据 Pydantic 的 `BaseModel` 数据模型自动验证输入数据**，并**在 Swagger UI 中自动生成交互式 API 文档**。

##### Path 路径参数不可用

核心原因：**Pydantic 数据模型（`BaseModel`）**本质上是一个 **JSON 格式的复杂类结构体（请求体）**，而 **Path 可变路径参数的值只能是基本类型**，所以**不能**被 Pydantic 的数据模型（`BaseModel`） **提取收集**，只能是**一个个定义在 API 接口函数的「形参列表」中**。

- ❌️ 错误示例

  ```python
  class PathModel(BaseModel):
      id: int = Field()
      name: str = Field()
      version_id: int = Field()
      
  
  @app.get('/user/{id}/{name}/{version_id}')
  async def get_user(id: Annotated[BaseModel, Path()]): pass 
  # ❌️ 这会抛出 AssertionError: Path params must be of one of the supported types 异常
  ```

- ✅️ 正确示例：

  ```python
  @app.get('/user/{id}/{name}/{version_id}')
  async def get_user(
  	id: Annotated[int, Path()], # 或者旧写法：id: int = Path()
      name: Annotated[str, Path()], # 或者旧写法：name: str = Path()
      version_id: Annotated[int, Path()], # 或者旧写法：version_id: int = Path()
  ): pass 
  ```

### 核心设计原则

#### 原则一、单一事实来源

FastAPI 的设计核心就是：把**请求参数**和**数据字段的校验规则**全部声明在 **API 接口函数的「形参列表」**里，这**唯一的一处代码**，同时作为了**数据接收器**、**类型转换器**、**安全校验器**和**文档生成器**。

```python
# 传统写法
@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>'):
async def <API 接口函数>(<形参>: <数据类型 | 数据模型> = <默认值约束/包装[Path()\Query()\Body()\Depends()...]>):
    # ...
    
    return <响应数据>

# 现代写法
@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>'):
async def <API 接口函数>(
    <形参>: <Annotated[<数据类型 | 数据模型>, <包装 Path()\Query()\Body()\Depends()>]> = <默认值>):
    # ...
    
    return <响应数据>
```

##### 运行机制

在 **API 接口函数的「形参列表」**中写下的那一行类型注解和规则（**`<形参变量名>: <数据类型> = <默认值约束/包装>`**），会同时被注入到 3 个不同的底层引擎中：

1. **数据提取引擎**：去 HTTP 数据包的指定物理位置（Path/Query/Body）强行把数据抠出来
2. **数据校验引擎（Pydantic）**：自动进行类型转换（如 `"123"` → `123`），并卡死值边界（如 `ge=1`）。验证失败立即抛出结构化的 `ValidationError`，**绝不允许不合法的数据污染业务逻辑代码**
3. **文档渲染引擎（OpenAPI/Swagger）**：自动把参数的名称、类型、是否必填、校验规则全部收集起来，实时渲染成在线交互式文档

#### 原则二、类型驱动与约定优于配置

FastAPI 极大地压榨了 Python 3 **静态类型提示（Type Hints）**的红利，建立了一套隐式的**参数自动推导算法**。

核心机制：框架通过检查 **API 接口函数中的 `形参变量`** 的 **“变量名”、“数据类型”** 以及 **“给定的默认值/包装函数”**，通过**级联的决策树**，**自动决定数据的物理来源**。

##### 级联判定链条

1. **特权判定**：是否为框架内置特殊对象（`Request`/`Response`/`BackgroundTasks`）？ → 是则直接注入到 `形参变量` 中，无视其他规则
2. **空间判定（Path 优先）**：变量名是否命中 URL 路径中的 `{可变参数}`？ → 命中则强制圈定为路径参数
3. **类型与标记判定（Query vs Body）**：
   - **基础类型**：默认判定为 “查询参数（Query）”。除非右侧强制声明包装为 `= Body()` 或 `:Annotated[<数据类型>, Query()]` 的请求体字段
   - **BaseModel 模型**：默认判定为 “请求体（Body）”（JSON 对象）。除非右侧强制声明包装为 `= Depends()` 或 `= Query()` 或 `:Annotated[<数据类型>, Query()]` 的 查询参数（Query）

#### 原则三、宏观路由检测与微观字段业务校验

FastAPI 将请求参数与业务流程拆分成了 2 个互补的场景：

##### **API 路由（函数形参列表）-- 宏观物理流向**

一个 HTTP 请求数据包的结构：

```python
POST /users/2026?source=search&version=2.0 HTTP/1.1  <-- 【请求行 (Path & Query)】
Host: api.example.com
Content-Type: application/json                      <-- 【请求头 (Header & Cookie)】

{                                                   
  "username": "Alex",                               <-- 【请求体 (Body)】
  "age": 25
}
```

在**接口函数的形参处**（或通过 `Annotated`），利用 **`Path()` / `Query()` / `Body()` / `Depends()`**，**明确告诉框架数据从哪里来**、**在 HTTP 数据包的哪个位置拦截**，并定义这个参数在接口层面的宏观属性（如是否必填、别名 `alias`、接口文档中的 `description`）。

```python
# 直接写在 API 接口函数的形参列表中，会导致代码臃肿
@app.post('/user/{id}')
async def up_user(
    # Path 路径参数 → /2026
	id: int = Path(...),
    
    # Query 查询参数
    source: str = Query(...),
    version: float = Query(...),
    
    # Body 请求体
    username: str = Body(...),
    age: int = Body(...)
    
    
    header: Header(),  # Header 请求头
    cookie: Cookie(),  # Cookie 数据
    request: Request,  # request 请求对象
    response: Response, # response 响应对象
):
    
    return ...
```

- **`Path()` 路径参数（Path Parameters）**：直接镶嵌在 URL 的**骨架路径**里（如上面的 `/2026`）。它属于资源定位的一部分，通常用来代表某个具体资源的唯一标识（ID）
- **`Query()` 查询参数（Query Parameters）**：挂在 URL 问号（`?`）后面的**键值对**（如 `?source=search&version=2.0`）
- **`Body()` 请求体（Request Body）**：最底部的 **Payload（载荷数据段）**。它通常是一个复杂的 JSON 字符串，用来携带要创建或修改的完整业务数据

##### **数据模型（Pydantic BaseModel） -- 微观业务细节**

当**发送的 HTTP 请求参数的字段变多**时，可以选择将微观的、精细的业务校验规则全盘移步到 Pydantic 模型内部进行“打包封装”。

机制：

- 利用 **`Field()`** 限制数值边界、字符串长度、正则匹配
- 利用 **`@field_validator`** 做单字段的深度业务清洗
- 利用 **`@model_validator`** 做多字段的联合逻辑校验（如：当密码重置时，二次输入的密码必须一致）

```python
# 将Query 查询参数、Body 请求体，提取出来放在 BaseModel 数据模型中，并采用更现代的 Depends()/Annotated() 进行包装声明

# 抽取数据结构、定义数据模型
# Query 查询参数-数据模型类
class QueryModel(BaseModel):
    source: str = Field(..., title="资源名"),
    version: float = Field(..., title="版本")

# Body 请求体-数据模型类
class BodyModel(BaseModel):
    username: str = Field(..., title="用户名"),
    age: int = Field(..., title="年龄")
    
    @field_validator('username')
    @classmethod
    def check_username(cls, data: str): 
        ...
    
    
@app.post('/user/{id}')
async def up_user(
    # Path 路径参数 → /2026
	id: int = Path(...),
    
    # Query 查询参数
	query: QueryModel = Depends() # 或者 Annotated([QueryModel, Query()])
    
    # Body 请求体
	body: BodyModel # 或者 Annotated([BodyModel, Body()])
    
    
    header: Header(),  # Header 请求头
    cookie: Cookie(),  # Cookie 数据
    request: Request,  # request 请求对象
    response: Response, # response 响应对象
):
    
    return ...
```

#### 对比传统框架

在传统框架里，为了完成一个接口，通常要做三件**重复**的事：

1. 在 API 接口函数的形参定义 `request`、`response` 来接收请求、响应对象
2. 在代码里从 `request.args` 拿数据，使用 `response` 设置响应体
3. 用校验库（如 Marshmallow）写一套校验规则
4. 在 Markdown 或 Swagger 注释里写一套 API 文档

在这个过程中，为了防止前端传错参数导致后端报 `500 Internal Server Error`，在每个 API 接口函数中，我们不得不写大量的 `if not param:` 或者 `try...except ValueError`。FastAPI 把这些脏活累活全在底层拦截了，只要进到 API 接口函数内部的数据，**百分之百是合法的、类型正确的**。这极大地保护了核心业务逻辑的纯粹性

##### 优缺点

- 优点：
  - FastAPI 将所有参数都一股脑的怼进 API 接口函数的形参列表中，可以做到针对单个或多个数据字段进行自动类型转换与基础校验，还可以采用 `Path()`、`Query`、`Body` 以及 `pydantic.BaseModel` 和 `pydantic.Field` 或 `@field_validator` \ `@model_validator` 进行进一步的数据字段的值校验处理
  - 不需要额外对接收到的请求体/响应体内的参数字段进行异常处理，FastAPI 内部会自动处理好它（验证不通过自动抛出 `ValidationError` 异常，并具有友好的异常信息提示（`e.json`））
  - 并且，FastAPI 会自动解析 API 接口函数的形参列表中，每个形参的形参名、数据类型、验证校验规则，并收集到一起自动生成一个 API 接口文档（Swagger UI），而不需要开发人员额外手动编写 API 文档。
- 缺点：
  - 将所有参数一股脑的放入 API 接口函数的形参列表中，会造成理解混乱，代码臃肿，难以阅读，虽然后期可以通过 Annotated 来有所改善，但是还是存在有一定的阅读困难；其次，对于初学者来说，难以理解，会造成一定的心智模型负担
  - 当一个接口需要路径参数、分页参数、请求头认证、Cookie、复杂 Body 时，函数的括号可能长达十多行

## 请求模型与校验

### Path 可变 URL 路径参数 `/{x}`

FastAPI 支持在 `URL` 请求路径中声明一个或多个 **可变参数** 关键字，作为**一段/多段 URL 可变短路径**，并在 **API 接口视图函数**中使用**同名形参**进行**匹配接收**。

#### 基本语法

- 使用 **`{x}`** 关键字来定义一个 **URL 可变路径参数**。

> 注：**不要**在整个 URL 路径字符串**前面加上 `f`**，否则它被视为字符串模板。

```python
from fastapi import FastAPI

<FastAPI 应用实例> = FastAPI()

# 定义一个路由函数，并设定一个或多个可变路径参数x、y...，在 API 视图函数中使用同名形参接收
@<FastAPI 应用实例>.<HTTP 请求方式>('<URL 路径>/{x}/{y}...')
async def <API 路由函数>(<x>: <数据类型>, <y>: <数据类型>,...):
    # <变量1> = <x>
    return <响应数据>
```

**命名匹配机制**：

​	如果 **API 接口函数的「形参名字」**刚好**匹配**了路由 **URL 路径中的 `{变量名}`**，FastAPI 会自动将其识别为**可变 URL 路径参数**。

##### 自动类型转换

- **自动类型转换**：FastAPI 会**根据类型注解（Type Hints）自动转换 Path 可变路径参数的数据类型**，**失败则返回 422 异常**。

```python
@app.get("/read_items/{id}")
async def read_items(id: int):
    print('id:', id)
    return {}
```

规则：

- 若传入 `/read_items/123` ，FastAPI 会将其 `123` **自动处理为字符串**
- 若**类型不匹配（如 `int` 类型，传入非数字）**，则返回 **422 异常**

##### 注意点

- Path 可变路径参数**适用**于任何 HTTP 方法**（GET、POST、PUT、DELETE...）**，`/{x}` 只是代表**一个资源路径的定位**

- API 接口函数的**参数名必须和路径参数名一致**，否则无法匹配并识别到

  ```python
  @app.get('/get_article/{id}')  # 可变路径参数为 id
  async def get_article(id: int):  # API 接口函数的形参名必须也为 id
  ```

- 如果**多个接口的 URL 路径一致**，则按**由上往下**的**顺序匹配**

  ```python
  @app.get('/get_article/{id}')  # ✅️优先匹配
  async def get_article(id: int):  pass
  
  
  @app.get('/get_article/{id}')  # ❌️被覆盖
  async def get_article(id: int):  pass
  ```

- `Pydantic` 的类型自动转换：默认情况下，在为 **`<可变路径参数>` 显式声明了某个数据类型（`int`、`bool`、`list`...）**之后，FastAPI 内部会**自动转换 `<可变路径参数>` 的数据类型**；如果**没有设定**，则**默认为 `str` 字符串类型**

  ```python
  @app.get('/get_article/{id}')
  async def get_article(id):  # 默认情况下，id 的类型为 str
      print(type(id)) # <class 'str'>
      
  @app.get('/get_article/{id}')
  async def get_article(id):  # 显式声明后，Pydantic 会自动转换 id 形成的类型为 int
      print(type(id)) # <class 'int'>
  ```

##### 示例

请求地址：

- http://127.0.0.1:8000/get_article/1
- http://127.0.0.1:8000/update_article/1/xxx

```python
# 引入 FastAPI 模块
from fastapi import FastAPI, Response, status
# 数据模型基类
from pydantic import BaseModel


# 定义一个 Article 数据模型类
class Article(BaseModel):
    id: int
    title: str
    content: str


# 创建 FastAPI 实例
app = FastAPI()


# 1. 获取文章列表（GET）
@app.get('/get_article/{id}')  # 实际路径：/update_article/1
async def get_article(id: int):  # 参数名必须和路径参数名一致，且类型会自动转换
    print('id:', id)  # 路径参数：1
    print(type(id))  # <class 'int'>

    article = Article(id=id, title='title', content='content')  # 模拟数据库查询

    return article  # 返回文章数据


# 2. 修改文章信息（POST）
@app.post('/update_article/{id}/{title}')  # 实际路径：/update_article/1/xxx
async def get_article_list(id, title: str):  # 参数名必须和路径参数名一致，且类型会自动转换
    print(f'id: {id}, title: {title}')  # 路径参数：1 title: xxx
    
    print(type(id), type(title)) #  <class 'str'> <class 'str'>

    article = Article(id=id, title=title, content='content')  # 模拟数据库查询

    return article  # 返回文章数据


# 启动应用
if __name__ == "__main__":
    import uvicorn

    uvicorn.run('server3:app', host="127.0.0.1", port=8000, reload=True)
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-可变 URL 路径参数.png)

#### `Path` 类（包装、校验）

FastAPI 内置提供了一个 **`fastapi.Path` 类**，它可以让 **URL** 中的 **可变路径参数** 具有**更复杂**的**值校验规则、元数据描述**等能力。

> 注：如果**只是在 API 接口函数的形参列表中定义一个同名形参变量**，**仅持有自动类型转换校验**的功能。

核心机制：

​	为 **API 接口函数**中的一个 **`形参变量` 赋值为一个 `Path()` 默认值**，可以将该 `形参变量` **强制包装为一个 URL 中的可变路径参数（前提是该 `形参变量` 与 URL 中的 `{x}` 关键字同名）**，并且可以实现对 **URL 路径中可变参数 `{x}`** 的**值校验**、**元数据描述（Swagger UI 文档）**。

##### 基本语法

```python
from fastapi import FastAPI, Path

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.<HTTP 请求方式>('<URL 路径>/{x}/...')
async def <API 路由函数>(<形参变量x>: <数据类型> = Path()):

    return <响应数据>
```

核心作用：`<形参变量x> = Path()` 的主要目的是告诉 FastAPI 框架：**“这个 <函数形参变量x> 的值应该从 URL 路径中提取。”**

核心要点：`Path()` 是一个集**位置声明（包装）、安全把关（参数值校验）**和**代码即文档（元数据描述）**于一体的生产力工具。

##### 包装

核心定义：核心定义：在 FastAPI 中，**`Path()` 函数用于显式声明一个形参应该从 URL 路径中提取解析**。它除了继承自 Pydantic 的基础校验参数（如 `default`, `description`）之外，还包含了一些专门用于控制**请求体解析行为**和 **OpenAPI/Swagger 文档生成**的高级参数。

`Path` 类提供了如下配置参数：

```python
Path(
	default: Any = Undefined, # 默认值

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
    title: ste | None = None, # 参数标题（简要）
    description: str | None = None, # 参数描述信息
    deprecated: bool | None = None, # 是否已被废弃
    #...
)
```

###### default 默认值处理

核心定义：路径参数在 URL 路径 中默认是**必传**的（如果 URL 是 `/items/{item_id}`，不可能不传 `item_id` 访问），但由于 `Path()` 中需通过**传入参数配置的形式**来**定义更多的校验规则**；因此，**`Path()` 的第一个参数通常是 `...`（Ellipsis，表示必填），或者干脆不写默认值**。

```python
Path(
	default: Any = Undefined, # 默认值
)
```

- **`default`：**

  - **不传**：表示**没有默认值（必传字段）**，后面**没有其他的参数配置**
  - **`...`**：一个**占位符**；表示**没有默认值（必传字段）**，后面**有其他的参数配置**

  **`Path()` / `Path(...)`**：两者是等价的

> 注：**`Path()` 不能显式传入一个默认值（只能不传或传入 `...` 占位符）**，因为 URL 路径参数是 前端发送请求时定义的，不能额外由后端来限制，否则会报错 `AssertionError: Path parameters cannot have a default value`。

示例：

```python
from fastapi import FastAPI, Path

app = FastAPI()

# 无参数配置，仅包装
@app.get("/items/{item_id}")
async def read_item(item_id: int = Path()): # ❌️错误示例：Path(default=xxx)
    return {"item_id": item_id}


# 有参数配置，需传入一个 ... 占位符，其他参数配置写在 ... 占位符之后，表示定义更多的业务校验规则
@app.get("/items/{item_id}")
async def read_item(item_id: int = Path(..., <其他参数配置...>)):
    return {"item_id": item_id}
```

在这个例子中，`Path()` 明确指定 `item_id` 必须来自 URL 的 `{item_id}` 位置。

##### 参数值校验

这是 `Path()` 最核心的功能，它允许对**传入的路径参数（`形参变量`）**进行**类型之外的值校验处理（颗粒度控制）**。

- 如果**参数值校验失败（前端发送了一个不符合的值）**，FastAPI 会**自动返回 `422 ValidationError` 错误，无需手动写 `if` 判断**

常用的参数值校验如下：

###### 长度限制

```python
Path(
    # 长度限制，适用于 str、list\dict\set\tuple 等序列容器数据类型
    min_length: int | None = None, # 最小长度
    max_length: int | None = None, # 最大长度
)
```

作用：适用于 **`str` 字符串、`list\dict\set\tuple` 等序列容器数据类型**的**长度限制**。

```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/read_str/{title}")
async def read_str(title: str = Path(..., min_length=3, max_length=10)):  # title 长度必须在3到10之间
    print('title:', title)
    return {}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run('server6:app', host="127.0.0.1", port=8000, reload=True)
```

规则：若传入 `/read_str/a`，则抛出 422 异常（`min_length` = 3, `max_length` = 10）。

###### 正则表达式验证

```python
Path(
    # 正则表达式校验
    pattern: str | None = None, # 新写法
    regex: str | None = None, # 旧写法
)
```

作用：适用于 **`str` 、`int` 等基本类型**数据，仅接受**完全匹配**的输入。

```python
@app.get("/pattern_item/{title}")
async def pattern_item(title: str = Path(..., pattern=r"^[a-z]+$")): # ^[a-z]+$ 仅输入小写字母
    print('title:', title)
    return {}
```

规则：当输入 `/pattern_item/a` 时校验通过，若 `a` 为 `A` 大写字母则校验失败，抛出 422 异常。

###### 数值范围验证

```python
Path(
    # 数值范围验证，适用于 int、float 数值类型
	gt: float | None = None, # 大于 ＞
    lt: float | None = None, # 小于 ＜
    ge: float | None = None, # 大于等于 ≥
    le: float | None = None, # 小于等于 ≤
)
```

作用：适用于 **`int`、`float` 等数值类型**的数据，用于**限制传入的查询参数值的数值范围**。

```python
@app.get("/user/{age}")
async def user(age: int = Path(..., ge=18, le=60)):  # age 必须在18到60之间
    print('age:', age)
    return {}
```

规则：当 `/user?age=xx` 的 `age` 值低于 18 或 超过 60 时，抛出 422 异常。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-ge-le 数值范围限制.png)

###### Enum 枚举值限制

核心定义：为 **可变 URL 路径参数** 定义**多个可选枚举项**，必须**选择其一**，否则会抛出 422 异常。

- 需事先提前定义**一个 `枚举类` （继承自 `enum.Enum`）**，并将**某个 可变 URL 路径参数 的数据类型**设为**该 `枚举类`**。

```python
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"


@app.get("/model/{model_name}")
async def get_model(model_name: ModelName):  # model_name 必须选择 alexnet 或 resnet
    if model_name == ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "resnet":
        return {"model_name": model_name, "message": "Image Recognition FTW!"}

    return {"model_name": model_name}
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Path 可变路径参数-Enum 枚举类选择验证.png)

规则：只能访问 `/model/alexnet` 或 `/model/resnet`，否则抛出 422 异常。

##### 元数据描述

核心定义：`Path()` 允许为参数（`形参变量`）添加人类可读的元数据。额外添加的元数据配置对程序的运行没有影响，但它们会被 FastAPI 自动提取，并**渲染到自动生成的 Swagger UI (`/docs`) 或 ReDoc 交互式 API 文档中**。

常用的元数据参数如下：

###### 别名与元数据

```python
Path(
    # 其他（Swagger UI 文档标注）
    alias: str | None = None, # 别名
    title: str | None = None, # 参数描述信息
    description: str | None = None, # 参数描述信息
)
```

- **`alias`** ：**避免 可变 URL 路径参数名命名冲突**，可使用 `alias` 为其**创建一个别名**。仅用于 Swagger UI 文档中显示
- **`title`：可变 URL 路径参数的描述信息**
- **`description`：参数的详细描述（支持 Markdown）**

```python
@app.get("/max_items/{id}")
async def max_items(id: int = Path(..., alias='item_id', title='根据 item_id 获取某个元素')):
    # 在 发起 API 请求时，id 可以被 item_id 代替；但是后台代码中，仍需使用 id
    print('id:', id)
    return {}
```

规则：可以使用 `/max_items/01` 进行访问。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-alias别名与元数据.png)

###### deprecated 弃用标记

```python
Path(
    # 其他（Swagger UI 文档标注）
    deprecated: bool | None = None, # 是否已被废弃
)
```

作用：通过 **`deprecated=True` 参数** 在 **Swagger UI API 接口文档**中**标记**该接口**已经被弃用**。（实际上还是能用的）

```python
@app.get("/old_api")
async def old_api(desc: int = Path(...,  description='已被弃用，请使用新的接口', deprecated=True)):
    return {}
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-deprecated 标记是否被弃用.png)

### Query 查询参数 `?x=y&...`

FastAPI 支持通过在 API 接口函数中的**形参列表**中**定义一个或多个形参**来**表示 Query 查询参数**。

- **Query 查询参数**：是**一个 `key=value` 键值对集合，位于 URL 路径的 `?` 之后，使用 `&` 分隔**。

#### 基本语法

```python
from fastapi import FastAPI

<FastAPI 应用实例> = FastAPI()

# 定义一个路由函数，并为 API 路由函数的形参列表中定义多个形参， 以表示 GET Queuy 查询参数
@<FastAPI 应用实例>.get('<URL 路径>')
async def <API 路由函数>(<形参变量x>: <数据类型>, <形参变量y>: <数据类型>,...):

    return <响应数据>
```

- 实际路径：**`/<URL 路径>?<形参变量x>=xxx&<形参变量y>=yyy`**

##### 参数识别机制

核心机制：如果 **API 接口函数的形参**是**单一的基本类型**（如 `str`, `int`, `float`, `bool` 等），且**没有**与 URL 路径中的 `{x}` 关键字**同名**；若同名则被识别为 Path 路径参数，否则 FastAPI 会**自动将其识别**为 **URL 的 Query 查询参数（即 `?key=value`）**。

```python
@app.get("/read_items/{id}") 
async def read_items(id: int, name: str): 
    # id 与 {id} 同名，识别为 Path 路径参数
    # name 没有出现在 URL 路径中，被识别为 Query 查询参数
    
    # 实际 URL 路径：/read_items/123?name=xxx
    return {}
```

##### 命名匹配机制

FastAPI 规定，在 **API 接口函数**中定义的 **`<形参变量x>`** 必须**与 HTTP 请求中 URL 路径 `?` 后的 Query 查询参数字段同名**：

- **✅️同名**：FastAPI **正确匹配到对应的 Query 查询参数字段，解析注入**
- **❌️不同名**：FastAPI **无法匹配到对应的 Query 查询参数字段**，抛出 **`422 Field required` 异常**

```python
[ HTTP 请求 URL ] ----->  /items?query_name=apple
                                     |
                             (直接寻找同名字段)
                                     |
                                     v
[ FastAPI 接口函数 ] ---> def read_items(query_name: str)  --> ✅ 成功注入 'apple'
                    ---> def read_items(x: str)           --> ❌ 找不到 'x'，抛出 422
```

###### alias 别名

解决方案：可以使用 **`Query()` 类的 `alias` 参数设定别名**，让 **`alias` 别名去匹配 Query 查询参数字段**，而 **API 接口函数中的 `形参变量` 则使用其他名称**。

```python
[ HTTP 请求 URL ] ----->  /items?query_name=apple
                                     |
                             (匹配 alias 的值)
                                     |
                                     v
[ FastAPI 接口函数 ] ---> def read_items(x: str = Query(alias="query_name"))
                                         |
                                         +------------------> ✅ x 成功接收到 'apple'
```

##### 自动类型转换

- **自动类型转换**：FastAPI 会**根据类型注解（Type Hints）自动转换 Query 查询参数的数据类型**，**失败则返回 422 异常**。

```python
@app.get("/read_items")
async def read_items(id: int):
    print('id:', id)
    return {}
```

规则：

- 若传入 `/read_items?id123` ，FastAPI 会将其 `123` **自动处理为字符串**
- 若**类型不匹配（如 `int` 类型，传入非数字）**，则返回 **422 异常**

##### `=` 默认值（必传/非必传）

FastAPI  规定一个 **API 函数的 `形参变量`**可以根据**是否设置了 `=` 默认值**来强制要求这个 Query 查询参数**是否为必传**：

- **未设置默认值：必传（Required）**
- **设置了默认值（None 或 其他值）：非必传**

```python
@app.get('/article_list')  # 实际路径：/article?page=xxx
async def article(page: int): pass # page: 必传

@app.get('/article_list')  # 实际路径：/article?page=xxx 或 /article?page=xxx&limit=yyy
async def article(page: int, limit: bool = True): pass # page: 必传, limit: 非必传
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query查询参数.png)

##### 示例

请求地址：

- http://127.0.0.1:8000/article?id=xxx
- http://127.0.0.1:8000/article_list?page=xxx&limit=yyy
- http://127.0.0.1:8000/article_list_page?page=xxx

```python
# 引入 FastAPI 模块
from fastapi import FastAPI

# 创建 FastAPI 实例
app = FastAPI()


# 1. 获取文章（GET）
@app.get('/article')  # 实际路径：/article?id=xxx
async def article(id: int):
    print('id:', id)  # Query 参数：id=1

    return {"message": "success", "article": {}}  # 返回单个文章数据


# 2. 获取文章列表（GET）
@app.get('/article_list')  # 实际路径：/article_list?page=xxx&limit=yyy
async def article_list(page: int, limit: int):
    # 若 page 或 limit 未设置默认值，则为 必选（Required）
    print(f"page: {page}, limit: {limit}")  # Query 参数：page=1&limit=10

    return []  # 返回文章列表数据


# 3. 获取文章列表（GET）
@app.get('/article_list_page')  # 实际路径：/article_list?page=xxx&limit=yyy
async def article_list_page(page: int, limit: int = None):
    # page: 必选，limit: 可选（默认值：None）
    print(f"page: {page}, limit: {limit}")  # Query 参数：page=1&limit=10

    return []  # 返回文章列表数据

# 启动应用
if __name__ == "__main__":
    import uvicorn

    uvicorn.run('server4:app', host="127.0.0.1", port=8000, reload=True)
```

#### `Query` 类（包装、校验）

FastAPI 中的 Query 查询参数校验主要通过 **`fastapi.Query` 类** 以及 `Pydantic` 数据模型实现。

- 可以实现对 **URL 请求路径中的 `?` 后面的 Query 查询参数的值**校验处理。 

##### 基本语法

```python
from fastapi import FastAPI, Query

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.get('<URL 路径>')
async def <API 路由函数>(<形参变量x>: <数据类型> = Query()):

    return <响应数据>
```

核心作用：`<形参变量x> = Query()` 的主要目的是告诉 FastAPI 框架：**“这个 <函数形参变量x> 的值应该从 URL 路径中 `?` 之后的 `k=v` Query 查询参数中提取。”**

核心要点：`Query()` 是一个集**位置声明（包装）、安全把关（参数值校验）**和**代码即文档（元数据描述）**于一体的生产力工具。

##### 包装

核心定义：在 FastAPI 中，**`Query()` 函数用于显式声明一个形参应该从 URL 路径中 `?` 后面的`k=v` 键值对集合中提取解析**。它除了继承自 Pydantic 的基础校验参数（如 `default`, `description`）之外，还包含了一些专门用于控制**请求体解析行为**和 **OpenAPI/Swagger 文档生成**的高级参数。

`Query` 类提供了如下校验方式：

```python
Query(
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
    title: str | None = None, # 参数标题
    description: str | None = None, # 参数描述信息
    deprecated: bool | None = None, # 是否已被废弃
    #...
)
```

###### default 默认值处理

```python
Query(
	default: Any = Undefined, # 默认值
)
```

- **`default`：**
  - **`...`：一个占位符；表示没有默认值；必传字段**
  - **`None \ 其他值`：表示设定了默认值；非必传字段**

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-default 默认值.png)

```python
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/read_items")
async def read_items(id: str = Query(...)):  # ... 是一个占位符，表示无默认值、必填字段
    print('id:', id)
    return {}


@app.get("/read_items2")
async def read_items2(id: str = Query(None)):  # 表示有默认值、非必填字段
    print('id:', id)
    return {}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run('server6:app', host="127.0.0.1", port=8000, reload=True)
```

规则：

- 若传入 `/read_items?id=123` ，FastAPI 会将其 `123` **自动处理为字符串**
- 若**类型不匹配（如 `int` 类型，传入非数字）**，则返回 **422 异常**

###### default_factory 动态默认值

*defualt 和 default_factory 是互斥的，必须二者选其一*

核心作用：`default_factory` 参数支持**传入一个 `def` 函数、`lambda` 匿名函数**，用于**为数据字段动态生成一个默认值（工厂模式）**。

> 注意：`default_factory` 参数**所传入的函数返回的值必须与 `<数据类型>` 一致**，否则会抛出 **422 `ValidationError` 异常**。

```python
Query(
    default_factory: (() -> Any) | None = _Unset, # 动态默认值（工厂）
)
```

示例：

```python
@app.get("/read_items")
async def read_items(id: int = Query(default_factory=lambda: 1)):  
    print('id:', id)
    return {}
```

##### 参数值校验

###### 长度限制

```python
Query(
    # 长度限制，适用于 str、list\dict\set\tuple 等序列容器数据类型
    min_length: int | None = None, # 最小长度
    max_length: int | None = None, # 最大长度
)
```

作用：适用于 **`str` 字符串、`list\dict\set\tuple` 等序列容器数据类型**的**长度限制**。

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/read_str")
async def read_str(title: str = Query(..., min_length=3, max_length=10)):  # title 长度必须在3到10之间
    print('title:', title)
    return {}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run('server6:app', host="127.0.0.1", port=8000, reload=True)
```

规则：若传入 `/read_str?title=a`，则抛出 422 异常（`min_length` = 3, `max_length` = 10）。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-min_length-max_length长度限制.png)

###### 正则表达式验证

```python
Query(
    # 正则表达式校验
    pattern: str | None = None, # 新写法
    regex: str | None = None, # 旧写法
)
```

作用：适用于 **`str` 、`int` 等基本类型**数据，仅接受**完全匹配**的输入。

```python
@app.get("/pattern_item")
async def pattern_item(title: str = Query(..., pattern=r"^[a-z]+$")): # ^[a-z]+$ 仅输入小写字母
    print('title:', title)
    return {}
```

规则：当输入 `/pattern_item?title=a` 时校验通过，若 `a` 为 `A` 大写字母则校验失败，抛出 422 异常。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-pattern 正则表达式校验.png)

###### 数值范围验证

```python
Query(
    # 数值范围验证，适用于 int、float 数值类型
	gt: float | None = None, # 大于 ＞
    lt: float | None = None, # 小于 ＜
    ge: float | None = None, # 大于等于 ≥
    le: float | None = None, # 小于等于 ≤
)
```

作用：适用于 **`int`、`float` 等数值类型**的数据，用于**限制传入的查询参数值的数值范围**。

```python
@app.get("/user")
async def user(age: int = Query(..., ge=18, le=60)):  # age 必须在18到60之间
    print('age:', age)
    return {}
```

规则：当 `/user?age=xx` 的 `age` 值低于 18 或 超过 60 时，抛出 422 异常。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-ge-le 数值范围限制.png)

###### 多值参数（数据容器）

定义：主要是**针对 `<数据类型>` 的定义限制**，当想要使用**同一个 Query 查询参数接受多个值**时，可以使用 **`List[]` 类型定义并接收**。

```python
@app.get("/get_items")
async def get_items(q: List[str] = Query(...)):  # q 为一个列表，可以传多个值
    print('q:', q)  # q: ['foo', 'bar']
    return {}
```

规则：访问 `/get_items?q=foo&q=bar` 时，`q` 的内容是 `['foo','bar']`。

##### 元数据描述

核心定义：`Query()` 允许为参数（`形参变量`）添加人类可读的元数据。额外添加的元数据配置对程序的运行没有影响，但它们会被 FastAPI 自动提取，并**渲染到自动生成的 Swagger UI (`/docs`) 或 ReDoc 交互式 API 文档中**。

常用的元数据参数如下：

###### 别名与元数据

```python
Query(
    # 其他（Swagger UI 文档标注）
    alias: str | None = None, # 别名
    title: str | None = None, # 参数描述信息
    description: str | None = None, # 参数描述信息
)
```

- **`alias`** ：**解决“形参名”与 “URL查询参数名”不一致的问题。**可使用 `alias` 为其**创建一个别名**去匹配
- **`title`：Query 查询参数的简要标题**
- **`description`：参数的详细描述（支持 Markdown）**

```python
@app.get("/max_items")
async def max_items(id: int = Query(0, alias='item_id', description='根据 item_id 获取某个元素')):
    # 在 发起 API 请求时，id 可以被 item_id 代替；但是后台代码中，仍需使用 id
    print('id:', id)
    return {}
```

规则：可以使用 `/max_items?item_id=01` 进行访问。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-alias别名与元数据.png)

###### deprecated 弃用标记

```python
Query(
    # 其他（Swagger UI 文档标注）
    deprecated: bool | None = None, # 是否已被废弃
)
```

作用：通过 **`deprecated=True` 参数** 在 **Swagger UI API 接口文档**中**标记**该接口**已经被弃用**。（实际上还是能用的）

```python
@app.get("/old_api")
async def old_api(desc: int = Query(0,  description='已被弃用，请使用新的接口', deprecated=True)):
    return {}
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-deprecated 标记是否被弃用.png)

#### 多字段校验处理

当一个 HTTP 请求的 URL 路径 `?` 之后有多个 Query 查询参数时，如果按照传统的写法，将一个个 `Query()` 都写在 API 接口函数的「形参列表」中会出现如下情况：

```python
from datetime import datetime
from fastapi import FastAPI, Query, Path

app = FastAPI()

# 多个 Query 查询参数的定义，不使用 Pydantic 数据模型，一股脑定义到 API 接口函数的形参列表中，代码极其臃肿
@app.get('/source/{source_id}', summary='获取数据源信息')
async def get_source(
    # Path 路径参数
    source_id: int = Path(..., description='数据源ID'),  # 必传

    # Query 查询参数
    source_name: str = Query(..., description='数据源名称'),  # 必传
    source_vp_id: int = Query(..., description='数据源版本ID'),  # 必传
    source_version: str = Query(..., description='数据源版本',
                                pattern='^[0-9].[0-9]+$'),  # 必传
    t: datetime = Query(default=datetime.today(), description='时间戳，格式为 YYYY-MM-DD',
                        examples=['2022-01-01']),  # 非必传
):
    pass

# 实际路径：/source/1?source_name=xxx&source_vp_id=1&source_version=1.0&t=2022-01-01
```

可以看出，一股脑将 Query 查询参数都定义到 API 接口函数的「形参列表」中，会导致代码极其臃肿，可以选择将其提取出来定义为一个结构体。

##### Pydantic 数据模型定义结构体

核心机制：

1. 将 **API 接口函数的「形参列表」**中的**一系列 `形参变量`（Query 查询参数）** 一并**提取收集**到外部的 Pydantic 的数据模型（**`BaseModel` 派生类**）中
2. 在 `BaseModel` 中以 **类结构** 的形式**统一定义 Query 查询参数的数据字段**
3. 并采用 **`Field()` 和 `@field_validator` / `@model_validator` 校验方法**进行**数据字段**的进一步**参数值规则校验**

```python
from datetime import datetime
from fastapi import FastAPI, Query, Path, Depends
from typing import Annotated
from pydantic import BaseModel, Field, field_validator, model_validator

# 使用 Pydantic 数据模型定义查询参数，更优雅。（Path 路径参数不可用，因为它只是一个基本类型）
class SourceQueryModel(BaseModel):
    source_name: str = Field(..., description='数据源名称')
    source_vp_id: int = Field(..., description='数据源版本ID'),
    source_version: str = Field(..., description='数据源版本',
                                pattern='^[0-9].[0-9]+$'),
    t: datetime = Field(default=datetime.today(), description='时间戳，格式为 YYYY-MM-DD',
                        examples=['2022-01-01']),

    # 更进一步的业务校验
    @field_validator('source_name')
    @classmethod
    def validate_source_name(cls, data: str):
        if not data:
            raise ValueError('source_name 不能为空')
        return data
```

##### Query() 统一声明接收

*新特性写法*

###### 历史遗留的 Depends() 用法

​	**早期的 `Query()`** 只能用来**修饰**并解析包装**单个基础类型**的变量，那时候开发者们使用的是 **`Depends()`** （黑魔法）以**依赖注入**的形式**逐个解析 Pydantic 数据模型**中的 数据字段 作为 API 接口函数中的 Query 查询参数；

​	但是这会造成一个问题：**`Depends()` 破坏了代码的语义**——明明是“查询参数（Query）”，代码里写的却是“依赖（Depends）”‘；

​	直到 2024 年底，FastAPI 官方重构了底层架构，**允许 `Query()`** 也**修饰并解析包装一整个 Pydantic 数据模型的 JSON 结构体**。

###### 核心用法

- **`Query()`** 不仅可以**修饰并解析包装 Pydantic 数据模型的数据字段为 Query 查询参数**
- 还可以传入 `title`、`description` 等**元数据参数配置以在 Swagger UI 文档中渲染 API 接口传参说明**

```python
@app.get("/source3/{source_id}")
async def read_item3(
    source_id: Annotated[int, Path(..., description='数据源ID')],  # 必传
    
    source_query2: SourceQueryModel = Query(..., description='数据源模型信息') # 传统写法
    source_query3: Annotated[SourceQueryModel, Query(..., description='数据源模型信息')], # 现代写法
):
    return source_query2
```

###### 多个 Query() 的“坑”

在 FastAPI 的早期和主流版本中，**不能在同一个路由函数里同时把一个 Pydantic 模型和另一个普通参数都用 `Query()` 声明**。

核心原因：**FastAPI 的参数解析引擎（基于 Pydantic）在处理多个 `Query()` 时产生了歧义**。

代码示例：

```python
# 分页数据模型
class PageModel(BaseModel):
    page: int = Field(default=1, title="页码", ge=1)
    size: int = Field(default=5, title="每页大小", ge=1, le=100)

@app.get("/books")
def get_article(
    page_limit_model: Annotated[PageModel, Query()], # Pydantic 数据模型（多个 Query 查询参数）

    name: Annotated[str, Query(..., title="文章名", max_length=10)] = None, # 普通 Query 查询参数
): pass
# 以上代码组合会导致 Swagger 文档混乱
# 解决方案：Pydantic 模型 => 使用 Depends()，单个Query 查询参数 => 使用 Query()
```

当 FastAPI 扫描到接口参数时，它会根据你的函数注解来决定去哪里找数据：*多个 `Query()` 会让 FastAPI “方向迷失”*

- **当它看到 `name: Annotated[str, Query()]`**：因为这是一个**标量（Scalar）**，它知道去 URL 里面找 `?name=xxx` 映射进来。
- **当它看到 `page_limit_model: Annotated[PageModel, Query()]`**：它就糊涂了。
  - `Query()` 告诉它：“去 URL 查询参数里找一个**扁平**的值”。
  - `PageModel` 却告诉它：“我是一个**结构化**的模型，里面包含 `page` 和 `size`”。

当这两个 `Query()` 同时存在时，FastAPI 的底层解析器无法智能地把 `?page=1&size=5` 这种扁平的 URL 字符串，“塞”进 `PageModel` 里，同时还要去兼顾旁边的 `name`。它会倾向于认为你想在 URL 里找一个**名字就叫 `page_limit_model`** 的参数，结果自然就对不上了。

> [!NOTE]
>
> 在 FastAPI 和 Pydantic 的底层逻辑中，`Query()` 的设计初衷是用来定义**单个**查询参数的校验规则。
>
> 当写下 `page_limit_model: PageModel = Query(...)` 时，FastAPI 会误以为想从 URL 中寻找一个**名为 `page_limit_model` 的单一参数**（类似于 `/books/...?...&page_limit_model=xxx`）。 但是，URL 传参其实是 `/books/...?...&page=1&size=10`：
>
> - FastAPI 找不到名为 `page_limit_model` 的查询参数。
> - FastAPI 也不知道该如何把扁平的 `page=1&size=10` 自动映射并实例化到你的 `PageModel` 结构体中。
>
> 因此，代码不仅无法正常接收参数，甚至可能会直接抛出 422 验证错误，或者得到的模型属性全都是默认值。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query()-Pydantic 模型与单个Query查询参数的混用Query()声明.png)

总结：在 FastAPI 较新的版本中，最推荐的优雅写法是使用 **`Query()` 配合 `Depends()`**，或者直接使用 Pydantic 模型作为依赖项。

**避坑提示：**在旧版本或为了代码的稳健性，**“模型用 `Depends()`，普通参数用 `Query()`”** 是最安全的黄金法则。只有这样，FastAPI 才知道一个是需要“拼装”的复杂对象，另一个是“伸手即拿”的普通字符串。

> [!NOTE]
>
> **为什么Depends() 可以？**
>
> `Depends()` 是 FastAPI 的**依赖注入（Dependency Injection）**核心组件。它的工作原理和普通的参数解析完全不同。
>
> 当使用 `Depends(PageModel)`（或者 `Annotated[PageModel, Depends()]`）时，FastAPI 会把 `PageModel` 当作一个**依赖项函数/类**去执行：
>
> 1. **参数平铺（Flattening）：** FastAPI 会去查看 `PageModel` 内部的字段（`page` 和 `size`）
> 2. **提取与校验：** 它会把这两个字段当作独立的查询参数去 URL 中寻找（寻找 `?page=...&size=...`），并分别应用 `Field(ge=1)` 等校验规则
> 3. **组合实例化：** 当所有字段校验通过后，FastAPI 会在后台帮你自动执行 `PageModel(page=page, size=size)`，最后将实例化好的对象注入到你的 `page_limit_model` 变量中
>
> 通过 `Depends()`，FastAPI 成功扮演了“胶水”的角色，把扁平的 URL 查询参数拼装成了你想要的复杂对象。

##### Depends() 解构注入声明接收

*这是一个 FastAPI 历史遗留的黑魔法写法*

###### 核心用法

描述：**将 `BaseModel` 派生类**以 **`<形参变量>: <BaseModel 派生类> = Depends()`** 的**类型声明**形式**定义到 API 接口函数的「形参列表」中**。

核心作用：**FastAPI 底层**会**调用`Depends()` 函数**将 **`<BaseModel 派生类>` 的结构体**进行**逐个解构拆解成一组 Query 查询参数**，最后一并**平铺注入**到 **API 接口函数的「形参列表」中**。

```python
# API 接口函数使用 Depends() 会将 SourceQueryModel 实例逐个解构，并平铺注入到函数的形参列表中
@app.get('/source2/{source_id}', summary='获取数据源信息')
async def get_source2(
    source_id: int = Path(..., description='数据源ID'),  # 必传
    
    source_query: SourceQueryModel = Depends(), # 传统写法
    source_query2: Annotated[SourceQueryModel, Depends()] # 现代写法
):
    # 注意：source_query 是一个 SourceQueryModel 实例，而不是一个字典
    print(source_query.source_name)
    print(source_query.source_vp_id)
    print(source_query.source_version)
    print(source_query.t)
```

###### Depends() 的底层机制

当定义 **`<形参变量>: <SourceQueryModel> = Depends()`** 时，FastAPI 会自动执行以下两步：

1. **平铺（Unpack）：声明时**；

   ​	FastAPI 会**调用 `Depends()`**  把 **`SourceQueryModel` 数据模型内部的所有字段**（`source_name`、`source_vp_id`,...）**全部拆解开**，**变成一个个独立的 Query 查询参数**，**平铺注入到 API 接口函数的「形参列表」中**

2. **重组（Pack）：运行时；**

   ​	当**客户端发起一个 HTTP 请求**时，FastAPI 会**将从 URL 路径中提取到的 散装 Query 查询参数**，调用 **`Depends()` 重新打包组装成一个 `SourceQueryModel` 数据模型对象**，并**交给 API 接口函数中「形参列表」的 `<形参变量>` 去接收**使用 

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/Depends() 对于 Query 查询参数的解构机制.png)

##### Depends() 与 Query() 的区别

共同点：

- **`Depends()`** 与 **`Query()`** 都可以用来**解构并接收 「一整个 Pydantic 复杂数据模型」** 的变量，即**一个 `BaseModel` 类结构体**

> 注：`Query()` 既可以用来修饰**单个**基础类型的变量（`形参`）、也可以接收**「一整个 Pydantic 复杂数据模型」** 的变量（`形参`）。

综合对比：

| **特性**            | **Depends() 包装模型**                                       | **Query() 包装模型（新特性）**                               |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **底层核心系统**    | 走的是 **Dependency Injection（依赖注入）** 系统             | 走的是 **Field/Parameter Validation（参数校验）** 系统。     |
| **Swagger UI 渲染** | 没问题，支持平铺                                             | 没问题，支持平铺。                                           |
| **参数级配置**      | **做不到**。<br />无法在 `Depends()` 里写 `description="分页参数"` 或设置别名 | **可以**。<br />可以写 `Query(description="...", deprecated=True)`，这些配置会应用到整个 Pydantic 模型集 |
| **子依赖嵌套**      | **支持**。该模型内部甚至还可以继续通过 `Depends()` 嵌套其他依赖（比如获取数据库连接） | **不支持**。它只纯粹地把模型当成数据校验对象，不能在里面嵌套别的依赖项 |

###### 总结

- 如果只是纯粹想把一个 Pydantic 模型当成**一堆散装的 URL 查询参数**来用，图个代码清爽，同时**希望在 Swagger 上能配置参数说明** —— **果断用 `Query()`**
- 如果这个 Pydantic 数据模型不是简单的 Query 查询参数集合，它在运行时还要顺便去查个数据库、校验个 Token、或者执行一些复杂的逻辑（即它本身就是一个**业务依赖项**） —— **继续用 `Depends()`**

```python
class QueryModel(BaseModel):
    page: int = Field()
    limit: str = Field()
    
@app.get('/')
async def get_data(
    query_params: QueryModel = Depends() # 一个数据模型结构体（多个参数变量组合）
):pass

@app.get('/')
async def get_data(
    query_params: QueryModel = Query(..., description="这是一个 / 路径的查询参数集合") 
    # 一个数据模型结构体（多个参数变量组合）
):pass
```

### Body 请求体 `{k:v}`

FastAPI 支持通过在 API 接口函数中的**「形参列表」**中**定义一个或多个 `形参变量`**来**表示 Request Body 请求体参数（数据字段）**。

- **Request Body 请求体参数**：是**一个 `key:value` 键值对集合，位于 Request Body 部分**。

> 注意点：FastAPI **严格遵守 RESTful 规范**，对于 **`GET/HEAD` 请求**是**不能携带 Request Body 请求体**部分的。

#### 参数识别机制

默认情况下，**无论什么 HTTP 请求方式（GET、POST、PUT...）**，单纯定义在 **API 接口函数中形参列表的 `形参变量`** （且**没有**与在 URL 路径的 `{x}` 路径参数同名）都会**默认**被 FastAPI **自动识别为 Query 查询参数**；

要想**将（仅限 `POST/PUT/DELETE` 请求）API  接口函数中的某个 `形参变量` 强制声明为一个 Body 请求体**，有以下情况：

- 如果 **API 接口函数的 `形参变量` 类型**是一个 **Pydantic 模型**（继承自 `BaseModel`），会被 **FastAPI 自动识别为一整个 Body 请求体**
- 如果 **API 接口函数的`形参变量` 类型**是一个 **基本数据类型（`int`、`str`...）**，则需要使用 **`Body()` 类进行显式声明**为**一个 Body 请求体的数据字段**

FastAPI 会**自动将它识别**为**从请求体（Body）中解析的 JSON 数据**。

```python
@app.<HTTP 请求方法>('<URL 路径>/{id}'):
async def <API 接口函数>(
    id: int, # 1. 与 URL 路径的 {id} 同名，自动识别为一个 Path 路径参数
    version: str, # 2. 不存在 URL 路径，自动识别为一个 Query 查询参数
    
    name: str = Body(), # 3. 强制给定 Body() 包装，会被识别为一个 Body 请求头参数的数据字段
    items: <BaseModel 数据模型> 
    # 3. 虽然 items 没有给定 Body() 包装，但是 FastAPI 默认会识别 BaseModel 为 Body 请求体参数的 JSON 数据结构体
):pass

# 实际 HTTP 请求：
#  请求行：/<URL 路径>/1?version=3.1
#  请求体：
#  {
#    name: 'string',
#    items: {...}	
#  }
#
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/FastAPI 参数识别机制.png)

#### `Body` 类（包装、校验）

FastAPI 中的 Body 请求体参数校验主要通过 **`fastapi.Body` 类** 以及 `Pydantic` 数据模型实现。

- 可以实现对 **Request Body 请求体的某个参数值（JSON 中的一个数据字段）**进行校验处理。 

##### 基本语法

```python
from fastapi import FastAPI, Body

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.get('<URL 路径>')
async def <API 路由函数>(<形参变量x>: <数据类型> = Body()): 
    # 不加 Body() 是 Query 查询参数，加了 Body() 则是请求体中的一个数据字段

    return <响应数据>
```

核心作用：

1. **`Body()`** 可以将写在 **API 接口函数中「形参列表」中的某个基本类型（`int`、`str`...）的 `形参变量x`** 从默认判定的 Query 查询参数**强制声明作为 JSON Body 请求体的一个数据字段（Field）**；
2. 同时，`Body()` 还可以通过**传入参数配置**来**对 `形参变量x`（请求体的一个字段）**进行**值校验处理、元数据描述**等操作

核心机制：**`<形参变量x> = Body()`** 的主要目的是告诉 FastAPI 框架：**“这个 <函数形参变量x> 的值应该从 HTTP 请求中的 Request Body 请求体部分（Payload）解析提取。”**

核心要点：`Body()` 是一个集**位置声明（包装）、安全把关（参数值校验）**和**代码即文档（元数据描述）**于一体的生产力工具。

###### 工作机制

FastAPI 会**自动将 API 接口函数中定义的包装为 `Body()` 的一系列 `形参变量` 组合到一个 `{}` 字典**中，作为 **API 接口的 Request Body 请求体参数**的数据字段。

```python
@app.post('/update_user')  # 实际路径：POST /update_user
async def update_user(user_id: int = Body(..., embed=True), user_name: str = Body(..., embed=True)):
    # FastAPI 会自动将 user_id 和 user_name 组合到一个字典，作为 /update_user 接口的请求体参数
    print('user_id:', user_id)
    print('user_name:', user_name)

    # POST 请求体参数：
    # {
    #     "user_id": 0,
    #     "user_name": "string"
    # }
    
    return {"message": "success"}
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-请求体参数-Body处理.png)

###### 命名匹配机制

FastAPI 规定，在 **API 接口函数**中定义的 **`<形参变量x>`** 必须**与 HTTP 请求中 Request Body 请求体的数据字段同名**：

- **✅️同名**：FastAPI **正确匹配到对应的 Request Body 请求体的数据字段，解析注入**
- **❌️不同名**：FastAPI **无法匹配到对应的 Request Body 请求体的数据字段**，抛出 **`422 Field required` 异常 **

```python
[ HTTP 请求体 (JSON) ] --->  { "user_name": "Miku" }
                                     |
                             (直接寻找同名字段)
                                     |
                                     v
[ FastAPI 接口函数 ]  --->  def create_user(user_name: str = Body())  --> ✅ 成功注入 'Miku'
                     --->  def create_user(x: str = Body())          --> ❌ 找不到 'x'，抛出 422
```

解决方案：可以使用 **`Body()` 类的 `alias` 参数设定别名**，让 **`alias` 别名去匹配 Request Body 请求体的数据字段**，而 **API 接口函数中的 `形参变量` 则使用其他名称**。

```python
[ HTTP 请求体 (JSON) ] --->  { "user_name": "Miku" }
                                     |
                             (匹配 alias 的值)
                                     |
                                     v
[ FastAPI 接口函数 ]  --->  def create_user(x: str = Body(None, alias="user_name"))
                                         |
                                         +----------------------------> ✅ x 成功接收到 'Miku'
```

###### 自动类型转换

- **自动类型转换**：FastAPI 会**根据类型注解（Type Hints）自动转换 Query 查询参数的数据类型**，**失败则返回 422 异常**。

```python
@app.post('/items')
async def update_item(
    item_id: int = Body(..., description='元素ID'),
    item_name: str = Body(..., description='元素名称'),
):
    return {'message': 'Hello World'}
```

规则：

- 若传入 `{ 'item_id': '123', 'item_name': 'xxx' }` ，FastAPI 会将其 `'123'` **自动处理为数值**
- 若**类型不匹配（如 `'item_id'` 字段为`int` 类型，但传入非数字）**，则**响应体**会返回 **422 异常**

```python
Status Code: 422
Error: Unprocessable Content

Response body
{
  "detail": [
    {
      "type": "json_invalid", # 错误类型
      "loc": [ # 错误位置
        "body", # 请求体
        15
      ],
      "msg": "JSON decode error", # 错误信息
      "input": {},
      "ctx": {
        "error": "Expecting value"
      }
    }
  ]
}
```

##### 包装

核心定义：在 FastAPI 中，**`Body()` 函数用于显式声明一个形参应该从 HTTP 请求体（Body）中提取解析**。它除了继承自 Pydantic 的基础校验参数（如 `default`, `description`）之外，还包含了一些专门用于控制**请求体解析行为**和 **OpenAPI/Swagger 文档生成**的高级参数。

`Body` 类提供了如下校验方式：

```python
Body(
	default: Any = Undefined, # 静态默认值
    default_factory: (() -> Any) | None = _Unset, # 动态默认值（工厂）
    media_type: str = 'application/json', # 预期接收的数据格式（Content-Type）
    embed: bool = False, # 是否将原始二进制的请求字段值转换为一个 JSON 键值对
    
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
    title: str | None = None, # 参数标题
    description: str | None = None, # 参数描述信息
    deprecated: bool | None = None, # 是否已被废弃
    #...
)
```

###### default 默认值处理

```python
Body(
	default: Any = Undefined, # 默认值
)
```

- **`default`：**
  - **`...`：一个占位符；表示没有默认值；必传字段**
  - **`None \ 其他值`：表示设定了默认值；非必传字段**

```python
@app.post('/up_item')
async def update_item2(
    name: str = Body(...),  # 必传
    age: int = Body(default=0)  # 非必传
):
    pass
```

规则：

- 若传入 `{ 'name': 'xxx', age: '123' }` ，FastAPI 会将其 `'123'` **自动处理为字符串**
- 若**类型不匹配（如 `age` 字段为`int` 类型，但传入非数字）**，则返回 **422 异常**

###### default_factory 动态默认值

*defualt 和 default_factory 是互斥的，必须二者选其一*

核心作用：`default_factory` 参数支持**传入一个 `def` 函数、`lambda` 匿名函数**，用于**为数据字段动态生成一个默认值（工厂模式）**。

> 注意：`default_factory` 参数**所传入的函数返回的值必须与 `<数据类型>` 一致**，否则会抛出 **422 `ValidationError` 异常**。

```python
Body(
    default_factory: (() -> Any) | None = _Unset, # 动态默认值（工厂）
)
```

示例：

```python
@app.post("/write_items")
async def write_items(name: str = Body(default_factory=lambda: 'Default Value', embed=True)):  
    print('name:', name)
    return {}
```

###### embed 嵌入模式

核心描述：

​	默认情况下，如果 FastAPI 的 **API 接口函数「形参列表」**中**只有一个被 `Body()` 修饰的 `形参变量` （请求体字段）**，那么该 **`形参变量`** 会被 FastAPI **仅仅视为一个原始值，而非一个 JSON 键值对**。

​	如果希望这个 **`形参变量`** 也被**包装**在**一个 JSON 键值对**中**作为请求体接收**，那么则需要**额外添加 `embed=True` 参数配置**。

**核心作用**：控制当接口**只有一个**基础类型（如 `str`, `int`）的 `Body()` 参数时，JSON 请求体的结构。

```python
# 场景 A：默认情况（embed=False）
@app.post("/items")
async def create_item(name: str = Body()): 
    return {"name": name}
# 前端发送的 Body 只能是纯字符串，不能带大括号： "数据"

# 场景 B：开启嵌入（embed=True）
@app.post("/items_embed")
async def create_item_embed(name: str = Body(embed=True)): 
    return {"name": name}
# 前端发送的 Body 必须是一个 JSON 对象： {"name": "数据"}
```

###### media_type 媒体类型

核心描述：

​	默认情况下，FastAPI 规定 **`Body()` 解析器**会将**客户端发送的 HTTP 请求体解析为 `application/json` 类型（JSON 格式）**来接收。

​	如果想开发一个**需要接收特殊格式（纯文本、XML、图片、字节流...）HTTP 请求体的 API 接口**，则可以**使用 `media_type` 来配置**，`media_type` 配置会**修改 Swagger 文档的显示**，提示前端开发者应该用什么 **Content-Type 类型传参**。

核心作用：用于**在生成的 OpenAPI (Swagger) 文档**中，显式指定该 **HTTP 请求体**所支持的 **HTTP Content-Type**（例如 `application/json`, `application/xml`, `text/plain` 等）。

> 适合用于表单提交、文件上传...等业务场景

```python
@app.post("/upload_file")
async def upload_file(
    file: bytes = Body(..., description='文件表单对象', media_type='multipart/form-data', embed=True),
):
    pass
```

**注意**：`media_type` 更多是用于**文档生成和声明**，FastAPI 内部对基础 **`Body()`** 的**默认自动解析**仍然是**基于 JSON 格式**的。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Body()-请求体字段的 media_type 媒体类型.png)

##### 参数值校验

###### 长度限制

```python
Body(
    # 长度限制，适用于 str、list\dict\set\tuple 等序列容器数据类型
    min_length: int | None = None, # 最小长度
    max_length: int | None = None, # 最大长度
)
```

作用：适用于 **`str` 字符串、`list\dict\set\tuple` 等序列容器数据类型**的**长度限制**。

```python
@app.post("/read_str")
async def read_str(
    # title 长度必须在3到10之间
    title: str = Body(..., min_length=3, max_length=10, embed=True)
):
    pass
```

规则：若传入 `{ 'title': 'a' }`，则抛出 422 异常（`min_length` = 3, `max_length` = 10）。

```python
Status Code: 422
Error: Unprocessable Content

Response body
{
  "detail": [
    {
      "type": "string_too_short",
      "loc": [
        "body",
        "title"
      ],
      "msg": "String should have at least 3 characters",
      "input": "1",
      "ctx": {
        "min_length": 3
      }
    }
  ]
}
```

###### 正则表达式验证

```python
Body(
    # 正则表达式校验
    pattern: str | None = None, # 新写法
)
```

作用：适用于 **`str` 、`int` 等基本类型**数据，仅接受**完全匹配**的输入。

```python
@app.post("/pattern_item")
async def pattern_item(
    title: str = Body(..., pattern=r"^[a-z]+$", embed=True)  # ^[a-z]+$ 仅输入小写字母
):
    pass
```

规则：当输入 `{ title: 'a' }` 时校验通过，若 `a` 为 `A` 大写字母则校验失败，抛出 422 异常。

```python
Status Code: 422
Error: Unprocessable Content

Response body
{
  "detail": [
    {
      "type": "string_pattern_mismatch",
      "loc": [
        "body",
        "title"
      ],
      "msg": "String should match pattern '^[a-z]+$'",
      "input": "QW",
      "ctx": {
        "pattern": "^[a-z]+$"
      }
    }
  ]
}
```

###### 数值范围验证

```python
Query(
    # 数值范围验证，适用于 int、float 数值类型
	gt: float | None = None, # 大于 ＞
    lt: float | None = None, # 小于 ＜
    ge: float | None = None, # 大于等于 ≥
    le: float | None = None, # 小于等于 ≤
)
```

作用：适用于 **`int`、`float` 等数值类型**的数据，用于**限制传入的查询参数值的数值范围**。

```python
@app.post("/user")
async def u_user(age: int = Body(..., ge=18, le=60, embed=True)):  # age 必须在18到60之间
    pass
```

规则：当 `/user?age=xx` 的 `age` 值低于 18 或 超过 60 时，抛出 422 异常。

```python
Status Code: 422
Error: Unprocessable Content

Response body
Download
{
  "detail": [
    {
      "type": "greater_than_equal",
      "loc": [
        "body",
        "age"
      ],
      "msg": "Input should be greater than or equal to 18",
      "input": 0,
      "ctx": {
        "ge": 18
      }
    }
  ]
}
```

###### 多值参数（数据容器）

定义：主要是**针对 `<数据类型>` 的定义限制**，当想要使用**同一个 Query 查询参数接受多个值**时，可以使用 **`List[]` 类型定义并接收**。

```python
@app.post("/get_items")
async def get_items(
    # name_list 为一个列表，可以传多个值
    name_list: List[str] = Body(..., embed=True),

    # person_info 为一个字典，可以传多个键值对
    person_info: Dict[str, str] = Body(..., embed=True)
):
    print('name_list:', name_list)  # name_list: ["jack", "tom"]
    print('person_info:', person_info)  # person_info: {"id":"001","name":"jack","age":"18"}

    # 发送的 HTTP 请求体：
    # {
    #     "name_list": ["jack", "tom"],
    #     "person_info": {
    #         "id": "001",
    #         "name": "jack",
    #         "age": "18"
    #     }
    # }
    return {}
```

##### 元数据描述

核心定义：`Body()` 允许为参数（`形参变量`）添加人类可读的元数据。额外添加的元数据配置对程序的运行没有影响，但它们会被 FastAPI 自动提取，并**渲染到自动生成的 Swagger UI (`/docs`) 或 ReDoc 交互式 API 文档中**。

常用的元数据参数如下：

###### 别名与元数据

```python
Body(
    # 其他（Swagger UI 文档标注）
    alias: str | None = None, # 别名
    title: str | None = None, # 参数简要
    description: str | None = None, # 参数描述信息
)
```

- **`alias`** ：**避免 Body 请求体字段参数名命名冲突**，可使用 `alias` 为其**创建一个别名**
- **`title`：Body 请求体字段参数的简要标题**
- **`description`：Body 请求体字段参数的详细描述（支持 Markdown）**

```python
@app.post("/max_items")
async def max_items(
    id: int = Body(0, alias='item_id', title="元素ID", description='根据 item_id 获取某个元素', embed=True)
):
    # 在 发起 API 请求体时，id 可以被 item_id 代替；但是后台代码中，仍需使用 id
    print('id:', id)
    return {}
```

规则：可以发送一个 `{ 'item_id': 0 }` 请求体。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Body()-请求体字段的 alias 别名.png)

###### deprecated 弃用标记

```python
Body(
    # 其他（Swagger UI 文档标注）
    deprecated: bool | None = None, # 是否已被废弃
)
```

作用：通过 **`deprecated=True` 参数** 在 **Swagger UI API 接口文档**中**标记**该接口**已经被弃用**。（实际上还是能用的）

```python
@app.post("/old_api")
async def old_api(desc: int = Body(0,  description='已被弃用，请使用新的接口', deprecated=True)):
    return {}
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Query 查询参数校验-deprecated 标记是否被弃用.png)

#### 多字段校验处理

##### Pydantic 数据模型定义结构体

定义：如果 API 接口函数中，形参的类型是一个 **Pydantic 模型**（继承自 `BaseModel`），FastAPI 会自动将它识别为**从请求体（Body）中解析的 JSON 数据**。

核心要点：通过**事先定义一个 `Pydantic` 数据模型类**传递**给 API 接口函数的形参，声明为形参的类型**；FastAPI 会**自动解析**这个由 `BaseModel` 创建的**数据模型类结构**，并**作为『非GET』请求方式的 API 接口预期接收的 Request Body 请求体参数**。

###### 基础模型

```python
from fastapi import FastAPI

<FastAPI 应用实例> = FastAPI()

# 数据模型A
class <数据模型A>(BaseModel):
    # 模型字段...
    <字段名>: <数据类型> = Field()
    <字段名>: <数据类型> = Field()
    # ...
```

###### 嵌套模型

```python
from fastapi import FastAPI

<FastAPI 应用实例> = FastAPI()

# 数据模型A
class <数据模型A>(BaseModel):
    # 模型字段...
    <字段名>: <数据类型> = Field()
    <字段名>: <数据类型> = Field()
    # ...

# 数据模型B
class <数据模型B>(BaseModel):
    # 模型字段...
    <字段名>: <数据类型> = Field()
    <字段名>: <数据模型A> = Field()
    # ...
```

##### API 接口形参声明

默认情况下，**无论什么 HTTP 请求方式（GET、POST、PUT...）**，单纯定义在 **API 接口函数中形参列表的 `形参变量`** （且**没有**与在 URL 路径的 `{x}` 路径参数同名）都会**默认**被 FastAPI **自动识别为 Query 查询参数**。

要想**将（仅限 `POST/PUT/DELETE` 请求）API  接口函数中的某个 `形参变量` 强制声明为一个 Body 请求体**，有以下情况：

- **`<形参变量>: <数据模型>`**：FastAPI 会将其**自动识别为一个 Body 请求体**，并**逐个解析组合成一个 `{}` 字典传递给 `形参变量`**

- **`<形参变量>: <数据模型> = Body()`** 或 **`<形参变量> : Annotated[<数据模型>, Body()]`**：

  ​	FastAPI 会**根据 `Body()` 对 `<数据模型>` 派生类的结构体**进行**逐个解析并包装为 `Body()` 请求体的数据字段**，并最后**组合成一个 `{}` 字典传递给 `形参变量`**

###### 工作机制

核心定义：如果 **API 接口函数**中定义的 **`<形参变量>: Pydantic 数据模型`** 有**多个**。

1. FastAPI 会**自动将所有数据模型拆解**
2. 并**重新以 `{<形参变量名>: <数据模型>}`的形式组合成一个`{}` 字典放在一起**，作为 **Request Body 的请求体参数**

<img src="C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/FastAPI-请求体处理机制.png" style="zoom:60%;" />

###### FastAPI 隐式自动识别

核心定义：如果 **API 接口函数的 `形参变量` 类型**是一个 **Pydantic 模型**（继承自 `BaseModel`），会被 **FastAPI 自动识别为一整个 Body 请求体**。即**`形参变量` = 从请求体（Body）中解析的 JSON 数据**。

语法：**`<形参变量>: <数据模型>`**

```python
# 引入 FastAPI 模块
from typing import Annotated
from fastapi import FastAPI, Path
from pydantic import BaseModel

# 创建 FastAPI 实例
app = FastAPI()


# 数据模型1
class Article(BaseModel):
    id: int
    title: str
    content: str


# 数据模型2
class User(BaseModel):
    user_id: int
    name: str
    is_admin: bool


# 1. 修改文章（POST）
@app.post('/set_article')  # 实际路径：POST /set_article
async def set_article(article: Article):
    print('article:', article)

    # POST 请求体参数：
    # {
    #     "id": 0,
    #     "title": "string",
    #     "content": "string"
    # }

    return {"message": "success"}


# 2. 修改文章（POST）
@app.post('/article/{id}')  # 实际路径：POST /article
async def article(
    id: Annotated[int, Path()],  # 路径参数

    # 请求体参数
    article: Article,
    user: User
    # article 和 user 不存在于 URL 路径中，且是一个 Pydantic 模型，会被识别为一个请求体参数
):
    # FastAPI 会自动解析 Article 和 User 组合为一个字典，作为 /article 接口的请求体参数
    print('article:', article)

    print('user:', user)

    # POST 请求体参数：
    # {
    #     "article": {
    #         "id": 0,
    #         "title": "string",
    #         "content": "string"
    #     },
    #     "user": {
    #         "user_id": 0,
    #         "name": "string",
    #         "is_admin": true
    #     }
    # }

    return {"message": "success"}


# 启动应用
if __name__ == "__main__":
    import uvicorn

    uvicorn.run('server16:app', host="127.0.0.1", port=8000, reload=True)
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-请求体参数.png)

###### Body() 显式包装声明

核心定义：FastAPI 本身内部已经能根据 Pydantic 数据模型自动识别为一个 Body 请求体，而使用 **`Body()` 显式声明**主要是**为了传入更多的配置说明（`title`、`description`...），以此来丰富 Swagger 文档的内容**。

语法：

- **`<形参变量>: <数据模型 | None> = Body(None)`**  （❌️传统旧写法）
- **`<形参变量> : Annotated[<数据模型 | None>, Body() = <默认值 | None>`**（✅️现代新写法）

> 注：关于生成后的 JSON 结构，**使用 `embed=True` 会让当前 `形参变量名` 作为 JSON 结构的顶层 Key**。
>
> | **场景**           | **函数形参声明方式**                      | **预期接收的 JSON 结构 (Request Body)**             |
> | ------------------ | ----------------------------------------- | --------------------------------------------------- |
> | **单一模型**       | `item: Item`                              | `{"field1": "val", "field2": "val"}` *(无顶层 Key)* |
> | **多模型组合**     | `item: Item, user: User`                  | `{"item": {...}, "user": {...}}` *(以变量名为 Key)* |
> | **显式单字段包装** | `item: Annotated[Item, Body(embed=True)]` | `{"item": {...}}` *(强制加上顶层 Key)*              |

```python
from typing import Annotated
from fastapi import FastAPI, Path, Body
from pydantic import BaseModel, Field

app = FastAPI()

# ---- 1. 定义数据模型 ----
class Tag(BaseModel):
    id: int = Field(..., title='标签ID')
    name: str = Field(..., title="标签名称")

class Article(BaseModel):
    id: int = Field(..., title='文章ID')
    title: str = Field(..., title="文章标题")
    content: str = Field(..., title="文章内容")
    tag: Tag = Field(..., title="文章标签")

class User(BaseModel):
    user_id: int = Field(..., title="用户ID")
    name: str = Field(..., title="用户名")


# ---- 2. API 接口定义 ----

# 场景一：路径参数 + 单个 Pydantic 模型（隐式解析）
# 注意：修正了原代码中路径 {id} 丢失形参绑定的问题
@app.post('/set_article/{id}')
async def set_article(
    id: Annotated[int, Path(description="文章ID")], 
    article: Article
):
    """
    【接收的 JSON 格式】根节点无包裹：
    {
        "id": 0,
        "title": "string",
        "content": "string",
        "tag": {"id": 0, "name": "string"}
    }
    """
    return {"status": "success", "msg": f"Article {id} updated", "data": article}


# 场景二：路径参数 + 多个 Pydantic 模型组合（自动转换为键值对包裹）
@app.post('/article/{id}')
async def update_article_with_user(
    id: Annotated[int, Path(description="文章ID")],
    article: Article,
    user: User
):
    """
    【接收的 JSON 格式】由于是多模型，FastAPI 自动以形参名作为顶层 Key：
    {
        "article": {
            "id": 0,
            "title": "string",
            "content": "string",
            "tag": {"id": 0, "name": "string"}
        },
        "user": {
            "user_id": 0,
            "name": "string"
        }
    }
    """
    return {"article": article, "user": user}


# 场景三：单个模型，但强行要求加上顶层 Key 包裹（使用 Body(embed=True)）
@app.post('/article/embed')
async def set_article_embed(
	# Body() 传统写法
    # article: Article = Body(..., description='文章信息', embed=True),
    
    # Annotated[] + Body() 现代写法
    article: Annotated[Article, Body(description="文章信息", embed=True)]
):
    """
    【接收的 JSON 格式】即使只有一个模型，也会被强制包裹：
    {
        "article": {
            "id": 0,
            "title": "string",
            "content": "string",
            "tag": {"id": 0, "name": "string"}
        }
    }
    """
    return {"data": article}


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
```

## Annotated 工具类 (现代写法)

在 Python 3.9+ 中，引入了一个 **`typing.Annotated` 工具类**。

核心定义：`Annotated` 是一个 **“类型声明 + 附加元数据（MetaData）”** 的**组合声明写法**。

核心作用：`Annotated` 可以在**不破坏原生类型检查（`Mypy`）的前提下**，给 **`变量` 或 `参数`** 添加**更多的自定义校验规则信息**。

简单来说，`Annotated` 就是可以**给 `变量` 或 `参数` 同时声明数据类型、元素数据信息**的一个强大工具。

### 基本语法

```python
async def <API 路由函数>(<形参变量x>: Annotated[<数据类型>, Path()/Query()/Body()] = <默认值>,...): pass
```

核心优势：

- 给 **`<形参变量x>` 声明一个 `<数据类型>`**，以及 **`Path()/Query()/Body()` 参数的包装、值验证约束**，还可以**赋予一个 `<默认值>`**

### 为什么需要 Annotated？

核心要点：**Annotated** 的出现就是为了**解决 FastAPI 框架的传统写法 与 Python 原生语法** 之间的**语义冲突**问题。

#### ❌️ 传统旧写法

在 `Annotated` 出现之前，FastAPI 为了实现 **参数校验** 和 **文档生成 一体化**，**强行霸占**了 Python **`形参` 的默认值系统**（`x = y` ）：

```python
# 旧写法
async def <API 接口函数>(
	p: int = Path(..., ge=10)
    q: str = Query(default="default_value", min_length=3)
    b: ItemModel = Body()
): pass
```

在这个旧写法里， **`Path()\Query()\Body()`** 被**伪装**成了一个**假的默认值**；这就导致了几个很别扭的问题：

1. **`形参` 默认值的语义冲突**：

   ​	**`形参` 真正的默认值 `default_value` 被隐藏起来**，从而**变成了 `Query()` 函数内部的一个 `default` 参数**，违背了 Python 的原生视觉

2. **必填项的逻辑冲突**：

   ​	**路径参数（Path）**明明是**必填**项，本**不该有默认值**，但 **FastAPI 为了配置校验**，不得不写成 `Path(...)` 这样的写法，**用 `...` 三个点的占位符**来**伪装**一个**必填的默认值**，这样会导致逻辑非常绕，阅读起来难以理解

3. **静态代码检查工具（IDE / Mypy）的检查冲突**：

   ```python
   p: int = Path(..., ge=10)
   ```

   以上旧写法会导致静态类型检查工具（Mypy、Pyright）发生一个错误的理解：“`p` 明明是一个 `int` 类型，却赋值了一个 `Path` 对象”，就会发生一个**类型不匹配**的错误；

   尽管在运行时 FastAPI 会将其替换，但在开发时往往代码检查工具会不通过而显示红波浪线提示。

#### ✅️ 现代写法

```python
# 现代写法
async def <API 接口函数>(
    p: Annotated[int, Path(ge=10)]
    q: Annotated[str, Query(min_length=3)]  = '默认值'
    b: Annotated[ItemModel, Body()]
): pass
```

优势：

- **符合 Python 原生语义写法、解放了 `=默认值` 的约束**：

  ```python
  <形参>: Annotated[<数据类型>, <Path()\Query()\Body()>] = <默认值>` 
  ```

  职责分工明确：

  - **`:` 冒号**专门负责 **“规则与类型”**
  - **`=` 等号**专门负责 **“默认值”**

  语法变得极其纯粹

- **静态类型检查工具（IDE、Mypy、Pyright）能正确识别**：

  当静态类型检测工具（Mypy、Pyright）看到 `Annotated[int, Path()]` 时，会**自动只读取第一个参数 `int`**，将 `形参` 声明为一个 `int` 类型，并**忽略后面的 `Path()` 交给 FastAPI 去处理**；

  这样既保证了**类型安全**、也保证了 **FastAPI 能正常处理 `形参` 值的校验、自动渲染 Swagger UI 文档**

### 综合用法

> 注意：在**使用 `Annotated[<数据类型>, Path()\Query()\Body()]` 包装声明**之后，若**想给定 `=` 默认值**，则**不需要再写成 `Path(...)`、`Query(...)`、`Body(..)`**，否则会导致**歧义【`...` 代表没有默认值，但 `=` 又显式给定了默认值】**。

#### 📌 Path 路径参数

- ❌️**传统写法 `<形参变量>: <数据类型> = Path()`**：

  ```python
  @app.get("/items/{item_id}")
  async def read_item(
      item_id: int = Path(..., ge=0)
  ):
      return {"item": "Success"}
  ```

- ✅️**现代写法 `<形参变量>: Annotated[<数据类型>, Path()]`**：

  ```python
  class TargetEnum(str, Enum):
      A = "A"
      B = "B"
      C = "C"
  
  
  @app.post("/item/{item_id}/{target}/{behavior}")
  async def update_item(
      item_id: Annotated[int, Path(..., description="元素ID")],
      target: Annotated[TargetEnum, Path(..., description="具体元素分类")],
      behavior: Annotated[str, Path(..., description="元素行为")]
      
      # behavior: Annotated[str, Path(...)] = 'A' # ❌️ 会报错，Path 路径参数不支持给定 =默认值
  ):
      return {"item": "Success"}
  ```

注意点：`Path()` 路径参数是**不支持给定 `=<默认值>` 的**。

#### 📌 Query 查询参数

- **单个数据字段**：

  - ❌️**传统写法 `<形参变量>: <数据类型 | None> = Query(None)`**：

    ```python
    @app.get('/demo')
    async def demo_fun(
        name: str = Query(default='jack', description="用户名", max_length=10),
        age: int = Query(default=15, description="年龄", gt=0, le=100)
    ): pass
    ```

  - ✅️**现代写法 `<形参变量>: Annotated[<数据类型 | None>, Query()] = <默认值 | None>`**：

    ```python
    @app.get('/demo2')
    async def demo_fun2(
        name: Annotated[
            str, Query(description="用户名", max_length=10)
        ] = 'jack',
        age: Annotated[int, Query(description="年龄", gt=0, le=100)] = 15,
    ): pass
    ```

- **多个数据字段（Pydantic 数据模型）**：

  ```python
  # 多个数据字段（Pydantic 数据模型）
  class UserModel(BaseModel):
      name: str = Field(..., description="用户名", max_length=10)
      age: int = Field(..., description="年龄", gt=0, le=100)
  ```

  - **`<形参变量>: Annotated[<数据模型 | None>, Query(<其他参数配置...>)] = <默认值 | None>`**（新特性）：

    ```python
    # -- Annotated[] + Query() 写法（✅️推荐）--
    @app.get('/demo4')
    async def demo_fun4(user: Annotated[UserModel, Query(..., description='用户信息')]):
        return user
    
    
    # -- Query() 写法（❌️不推荐）--
    @app.get('/demo3')
    async def demo_fun3(user: UserModel = Query(...)):
        return user
    ```

  - **`<形参变量>: Annotated[<数据模型 | None>, Depends()] = <默认值 | None>`**（稳定）：

    ```python
    # -- Annotated[] + Depends() 现代写法（✅️推荐）--
    @app.get('/demo5')
    async def demo_fun5(user: Annotated[UserModel, Depends()]):
        return user
    
    
    # 传统写法（❌️不推荐）
    @app.get('/demo5')
    async def demo_fun5(user: UserModel = Depends()):
        return user
    ```

  **两者的核心区别**：**`Query()` 方式**会严格**受到 Query 参数的元数据限制**，而 **`Depends()` 属于依赖注入**，**灵活性更高**（比如能在内部追加 `Header` 或异常处理）。

#### 📌 Body 请求体

- **单个数据字段**：

  - ❌️ **传统写法 `<形参变量>: <数据模型 | None> = Body(None)`**：

    ```python
    @app.post('/up_item')
    async def update_item2(
        name: str = Body(),  # 必传
        age: int = Body(default=0)  # 非必传
    ):
        pass
    ```

  - ✅️ **现代写法 `<形参变量>: Annotated[<数据模型 | None>, Body()] = <默认值 | None>`**：

    ```python
    @app.post('/up_item')
    async def update_item2(
        name: Annotated[str, Body()],  # 必传
        age: Annotated[int, Body()] = 0  # 非必传
    ):
        pass
    ```

- **多个数据字段（Pydantic 数据模型）**：

  ```python
  # 多个数据字段（Pydantic 数据模型）
  class UserModel(BaseModel):
      name: str = Field(..., description="用户名", max_length=10)
      age: int = Field(..., description="年龄", gt=0, le=100)
  ```

  - ❌️**传统写法 `<形参变量>: <数据类型 | None> = Body(None)`**：

    ```python
    @app.post('/up_item')
    async def update_item2(
        user_data: UserModel = Body(..., description="用户信息模型")
    ):
        pass
    ```

  - ✅️**现代写法 `<形参变量>: Annotated[<数据模型 | None>, Body(<其他参数配置...>)] = <默认值 | None>`**（新特性）：

    ```python
    @app.post('/up_item')
    async def update_item2(
        user_data: Annotated[UserModel | None, Body(description="用户信息模型")] = None
    ):
        pass
    ```

### 类型别名复用

这是 `Annotated` 的一个核心优势功能。

核心流程：

1. `Annotated` 支持**在 API 接口函数外部**使用  **`变量 = Annotated[]`** 的形式，**得到一个类型别名 `变量A`**
2. 之后，将得到的**类型别名 `变量A` 重新赋值给 API 接口函数中的 `形参: 变量A`**，以实现 **`Annotated` 类型别名的复用**

```python
from pydantic import BaseModel
from fastapi import Query,Path,Body
from typing import Annotated

# 数据模型
class <数据模型Q>(BaseModel): pass

# 类型别名：一次定义、多处复用
<类型别名-变量A> = Annotated[<数据类型>, <包装/约束 Path()\Query()\Body(),...>]
<类型别名-变量B> = Annotated[<数据模型Q>, <包装/约束 Query()\Body()\Depends(),...>]


# API 接口函数的形参复用
async def <API 接口函数A>(<形参X>: <类型别名-变量A> = <默认值>): pass
async def <API 接口函数B>(<形参X>: <类型别名-变量B> = <默认值>): pass
...
```

#### 核心场景

在实际开发中，可能存在**多个 API 接口共同接收一些完全相同的 `形参` 参数**，它们的**校验规则完全一样**。

##### ❌️ 传统旧写法

如果使用旧写法，必须在每个 API 接口函数中重写一遍，代码非常臃肿：

```python
# 比如分页接口，都需要使用 page、limit 查询参数
async def get_users(page: int = Query(ge=1,le=100), limit: int = Query(ge=1,le=1000),...): pass
async def get_books(page: int = Query(ge=1,le=100), limit: int = Query(ge=1,le=1000),...): pass
```

##### ✅️ 现代写法

可以**自定义多个 `变量 = Annotated[]` 的类型别名**，并给**多个 API 接口函数的 `形参`** 进行**类型的声明复用**：

```python
from typing import Annotated
from fastapi import Query

# 类型别名：一次定义，多处复用
PageParam = Annotated[int, Query(ge=1,le=100, title="分页参数")]
LimitParam = Annotated[int, Query(ge=1,le=1000, title="当前页数量参数")]

# 多个 API 接口中，形参类型声明复用
async def get_users(page: PageParam, limit: LimitParam): pass
async def get_books(page: PageParam, limit: LimitParam): pass
```

后续若要**统一调整 `page` 或 `limit` 的上限值**，则**只需修改 `PageParam` 和 `LimitParam` 这一行代码**即可，全局所有复用它们的 API 接口都会刷新。

## 核心总结

### 形参-请求参数识别机制

​	FastAPI 利用 Python 的**类型提示（Type Hints）**，会**自动解析**定义在 **API 接口函数形参列表**里的 **`形参变量`**；

​	根据 **`形参变量`** 给定的 **`:` 数据类型（基本数据类型、Pydantic 复杂数据模型、`Annatoted`）** + 给定的 **`=` 默认值（`Path`、`Query`、`Body`、`Request`、`Response`、`Depends`...）**；

​	由此，来自动判断这些 `形参变量` 应该属于 **路径参数（Path）**、**查询参数（Query）**、 **请求体（Body）**、**请求对象（Request）**还是 **响应对象（Response）**。

#### 核心定义

​	当在 **API 接口函数的「形参列表」**中写下 **`形参变量名: 数据类型 = 默认值（或显式声明）`** 时，FastAPI 会**自动识别这个 `形参变量` 的数据应该从 HTTP 请求的哪个地方（`URL`、`Header`、`Body`）拿取**。

​	之后，当**一个 HTTP 请求进来**时，FastAPI 会**调用对应的 API 接口函数（`@` 路由装饰器修饰的函数）**，并**查看该 API 接口函数的形参列表中的 `形参变量`**，并执行如下推导判断流程：

#### **核心流程**

1. **特殊对象优先通道（Request / Response / BackgroundTasks...）**

   - 如果 **`形参变量`** 被**显式声明**为 **`fastapi.Request`、`fastapi.Response` 或 `BackgroundTasks` 等框架特有类型**
     - 不管变量名和默认值是什么，FastAPI 都会直接把**底层的 HTTP 请求/响应对象塞给该 `形参变量`**
   - 若不属于特殊对象，进入下一通道

2. **URL 路径匹配通道（Path）**

   检查 **`形参变量名`** **是否与 URL 路由路径中的可变参数同名**（如 `/users/{user_id}` 中的 `user_id`）？

   1. **同名**：强制**包装**为一个 **Path（路径参数）**。此时可**给定 `=Path()` 或 `=Annotated(...,Path())` 默认值**用于**附加值校验**
   2. **不同名**：进入下一通道

3. **判断 `形参变量` 给定的 `:` 数据类型** 和 **`=` 默认值**：

   - **基础类型（`int`、`float`、`str`、`bool`...）**

     - **（无论 GET 还是 POST/PUT 请求方式）**：默认**包装**为一个 **`Query`（查询参数）**；可以**给定 `=Query()` 默认值**用于**查询参数值的校验**
     - 改判规则：
       - 若**显式指定了 `:Annotated(...,Body())` 类型 或 `=Body()`默认值**，则强制**包装**为一个 **`Body`（请求体的数据字段）**
       - 若**显式指定了 `=Header()` 或 `=Cookie()` 默认值**，则强制**包装**为**一个 请求头 或 Cookie 对象**

   - **Pydantic 复杂数据模型（`BaseModel` 派生类）**

     - 默认**包装**为一个 **`Body`（整个请求体的 JSON 数据）**；可以**给定 `=Body()` 默认值**用于**请求体单个数据字段值的校验**
     - 改判规则：
       - 若**显式指定了 `:Annotated(...,Query())`类型 或`= Depends()` 或 `= Query()` 默认值**，则**强制解构并包装该 `形参变量` 为一组 `Query`（查询参数）**

     此时，可以**在 `BaseModel` 类中为「单个或多个特定数据字段」**=> **给定 `=Field()` 默认值** 或 **`@field_validator` / `@model_validator` 校验方法**来进行**特定单个或多个数据字段值的校验**

混用示例：

```python
# Path()、Query()、Body()、Depends()、Annotated() 混用
from fastapi import FastAPI, Path, Query, Body, Depends
from pydantic import BaseModel, Field
from typing import Annotated
from datetime import datetime

app = FastAPI()


# 类型别名复用
TagIDParam = Annotated[int, Path(..., description="标签ID", ge=1)]
ArticleIDParam = Annotated[int, Path(..., description="文章ID", ge=1)]


# 分页数据模型
class PageModel(BaseModel):
    page: int = Field(default=1, title="页码", ge=1)
    size: int = Field(default=5, title="每页大小", ge=1, le=100)


# GET 请求：/books/{tag_id}/{article_id}?name=xxx&page=1&size=10
@app.get("/books/{tag_id}/{article_id}")
def get_article(
    tag_id: TagIDParam,
    article_id: ArticleIDParam,
    
	# 单个 Query 查询参数使用 Query()
    name: Annotated[str, Query(..., title="文章名", max_length=10)] = None,
    
    # Python 规定，有默认值的参数必须放在最后，可以使用 * 来消除
    *,
    
    # Pydantic 数据模型用 Depend()
    page_limit_model: Annotated[PageModel, Depends()],
): pass


class Tag(BaseModel):
    id: int = Field(..., title='标签ID')
    name: str = Field(..., title="标签名称")


class Article(BaseModel):
    id: int = Field(..., title='文章ID')
    title: str = Field(..., title="文章标题")
    content: str = Field(..., title="文章内容")

    # 嵌套数据模型
    tag: Tag = Field(..., title="文章标签")


# POST 请求：/books/{tag_id}/{article_id}/up_article?name=xxx&t=2023-01-01
@app.post("/books/{tag_id}/{article_id}")
async def up_article(
    # Path 路径参数
    tag_id: TagIDParam,
    article_id: ArticleIDParam,

    # Query 查询参数
    name: Annotated[str, Query(description="文章名", max_length=10)],
    t: Annotated[datetime, Query(description='时间')],

    # 请求体
    article: Annotated[Article, Body(description='文章信息', embed=True)]
):
    # 发送的 HTTP 请求体
    # {
    #     "article": {
    #         "id": 1,
    #         "title": "xxx",
    #         "content": "xxx",
    #         "tag": {
    #             "id": 1,
    #             "name": "xxx"
    #         }
    #     }
    # }

    return {'msg': "Success"}

# 启动应用
if __name__ == "__main__":
    import uvicorn

    uvicorn.run('server18:app', host="127.0.0.1", port=8000, reload=True)
```

### 综合用法

#### 📌1. Path 路径参数

- **物理位置**：URL 路径内，例如 `/items/{id}`。
- **核心职责**：校验和提取 URL 路由里的动态变量。
- **接口形参包装写法**：
  - **传统写法**：`id: int = Path(..., ge=1, description="元素ID")`
  - **现代写法**：`id: Annotated[int, Path(ge=1, description="元素ID")]`

#### 📌2. Query 查询参数

- **物理位置**：URL 问号后，例如 `?page=1&limit=10`。

- **核心职责**：校验和提取 URL 后面挂载的键值对数据。

- **接口形参包装写法**：

  - **单字段情况**：

    - **传统写法**：`page: int = Query(default=1, ge=1)`
    - **现代写法**：`page: Annotated[int, Query(ge=1)] = 1`

  - **多字段情况（复杂模型打包）**：

    - *Step 1: Pydantic 模型内定义与微观校验*

      ```python
      class ItemQuery(BaseModel):
          page: int = Field(default=1, ge=1)
          limit: int = Field(default=10, le=100)
          
          @field_validator('limit') # 针对单个字段的深度校验
          @classmethod
          def check_limit(cls, v): ...
          
          @model_validator(mode='after') # 针对多字段联动校验
          def check_page_limit(self, data: int): ...
      ```

    - *Step 2: 接口形参声明（二选一，皆会触发全字段自动解构）*

      - **经典依赖注入法（推荐）**：`query_data: ItemQuery = Depends()`  *Depends() 会把 Pydantic 数据模型（ItemQuery）类的结构解构成一组 Query 查询参数放在 API 接口函数的「形参列表」中*
      - **现代注解标签法**：`query_data: Annotated[ItemQuery, Query()]`

#### 📌3. 请求体（Request Body）

- **物理位置**：HTTP 请求体 Body 内，例如 `{...}` (JSON 载荷)。

- **核心职责**：校验和解析客户端提交的复杂结构化数据（常用于 POST / PUT）

- **接口形参包装写法**：

  - **单字段情况**（从 JSON 中单独抠出一个键值对）：

    - **传统写法**：`item_name: str = Body(..., min_length=2)`
    - **现代写法**：`item_name: Annotated[str, Body(min_length=2)]`

  - **多字段情况（复杂模型打包）**：

    - *Step 1: Pydantic 模型内定义与微观校验*

      ```python
      class ItemBody(BaseModel):
          name: str = Field(..., min_length=2)
          price: float = Field(..., gt=0)
          
          @field_validator('name')
          @classmethod
          def check_name(): ... # 针对 name 字段的进一步校验处理
      
          @model_validator(mode='after') # 针对多字段联动校验
          def check_price_and_name(self, data: str | float): ...
      ```

    - *Step 2: 接口形参声明（二选一）*

      - **极简默认法（最常用）**：`body_data: ItemBody`（FastAPI 识别到 BaseModel 默认判定为 Body）
      - **现代明确法**：`body_data: Annotated[ItemBody, Body()]`

#### 📌4. 请求对象（Request）

- **物理位置**：整个 HTTP 请求上下文中。
- **核心职责**：用于获取未经过框架加工的原始请求数据（如客户端真实 IP、全量 Headers、原始 Cookies 等）。
- **接口形参包装写法**：
  - `request: Request` （直接声明类型，无需任何 `=` 默认值或包装函数）

#### 📌5. 响应对象（Response）

- **物理位置**：整个 HTTP 响应上下文中。
- **核心职责**：用于在 API 接口函数内部，动态地向客户端追加自定义的响应头（Headers）或直接写入 Cookie 原始数据。
- **接口形参包装写法**：
  - `response: Response` （直接声明类型，无需任何 `=` 默认值或包装函数）



