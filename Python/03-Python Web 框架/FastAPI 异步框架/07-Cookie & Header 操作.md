# Cookie & Header

## Cookie 凭证

### 前置概念

在 **HTTP 协议**中，**Cookie 的交互**是一个 **“服务器发送 → 浏览器保存 → 浏览器每次请求携带”** 的闭环过程。

- **Cookie 数据**：是**一个 `k=v` 键值对集合，使用 `;` 分号分隔**；

Cookie 分别存储在**请求头（Request Headers）**和**响应头（Response Headers）**中，且表现形式是不同的。

> 注：**Cookie 的 `value` 值**可以是**任何值（`int`、`str`、`bool`、`{}` 对象）**，最终都会转为**字符串形式传输**。

#### 请求头

当浏览器向服务器发送请求时，如果该域名下有可用的 Cookie，浏览器会自动在**请求头**中加入 `Cookie` 字段。

- （**`Cookie` 字段**）：以 **`Cookie: key1=value1; key2=value2;...`** 形式存储

```ini
GET /index.html HTTP/1.1
Host: www.example.com
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...
Accept: text/html,application/xhtml+xml,xml;q=0.9,image/webp,...
Cookie: logined=true; uid=123456; theme=dark; session_id=abc123xyz789 # Cookie 数据键值对（字符串）
```

#### 响应头

如果服务器要设置多个 Cookie，通常会返回**多个独立的 `Set-Cookie` 头**，并且每个 Cookie 后面都可以跟着它的控制属性。

- （**`Set-Cookie` 字段**）：以 **`Set-Cookie: key1=value1; Domain=xxx; Secure; HttpOnly...`** 的形式存储

```ini
Set-Cookie: key1=value1; Expires=Wed, 09 Jun 2021 10:18:14 GMT; Secure; HttpOnly
Set-Cookie: key2=value2; Path=/; Domain=example.com
```

属性解析：

- **Expires / Max-Age**: 决定这个 Cookie 能活多久
- **Domain / Path**: 限制哪些域名和路径下的请求才可以携带这个 Cookie
- **Secure**: 只有在 HTTPS 安全连接时才会发送该 Cookie
- **HttpOnly**: 禁止 JavaScript（如 `document.cookie`）读取该 Cookie，防止 XSS 攻击
- **SameSite**: 限制跨站请求时是否携带 Cookie（防范 CSRF 攻击）

#### 综合对比

> 流程：服务器用 `Set-Cookie` 一条一条精细化地喂给浏览器；浏览器吃下去之后，每次出门（发请求）就把它们打包成一条总的 `Cookie` 捎带上。

| **特性**     | **请求头（Request Header）**     | **响应头（Response Header）**           |
| ------------ | -------------------------------- | --------------------------------------- |
| **字段名称** | **`Cookie`**                     | **`Set-Cookie`**                        |
| **发送方向** | 浏览器 → 服务器                  | 服务器 → 浏览器                         |
| **数据格式** | 单行字符串，多对键值用 `; ` 分隔 | 多行独立字段，包含具体属性              |
| **示例**     | `Cookie: uid=123; theme=dark`    | `Set-Cookie: uid=123; HttpOnly; Secure` |

在 FastAPI 中，Cookie 的操作主要分为**读取、设置**和**删除** 3 个部分，都需要涉及到不同的类。

### 声明读取 Cookie

FastAPI 中的 Cookie 参数的**读取、值校验**主要通过 **`fastapi.Cookie` 类** 以及 `Pydantic` 数据模型实现。

- 可以实现对 **HTTP 请求头中的 Cookie 数据值**校验处理。 

#### `Cookie` 类（包装、校验）

##### 基本语法

FastAPI 支持通过在 API 接口函数中的**形参列表**中**定义一个或多个形参，并使用 `Cookie()` 包装**来**表示接收指定 Cookie 键值对字段**。

```python
from fastapi import FastAPI, Cookie

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>')
async def <API 路由函数>(<Cookie-Key>: <数据类型> = Cookie()):
	# 从 HTTP请求 中提取与 <Cookie-Key> 同名的 Cookie 数据键值对字段
    return <响应数据>
```

> 注：如果**不使用 `Cookie()`**，FastAPI 会**默认**把 `<Cookie-Key>` 形参当作一个**Query 查询参数（Query Parameter）**。

###### 命名匹配机制

```ini
<Cookie-Key>: <数据类型> = Cookie()
<Cookie-Key>: Annotated[<数据类型>, Cookie()]
```

FastAPI 框架会**自动 “从 HTTP请求 中提取与 `<Cookie-Key>` （`形参变量`） 同名匹配的 Cookie 键值对字段，并提取对应 Cookie 字段的值注入到 `<Cookie-Key>` 这个 `形参变量` 中”**

###### 示例

```ini
# HTTP 请求
Cookie: ads_id=xxx;
```

```python
# API 接口函数
@app.get("/items")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    # FastAPI 框架会自动从 HTTP 请求中的 Cookie 部分查找并匹配 ads_id 字段，最后将 xxx 值传递给 ads_id 这个形参变量
    print('提取到的 Cookie： ads_id', ads_id) # xxx
    return {"ads_id": ads_id}
```

核心要点：`Cookie()` 是一个集**位置声明（包装）、安全把关（参数值校验）**和**代码即文档（元数据描述）**于一体的生产力工具。

##### 包装、校验

核心定义：在 FastAPI 中，**`Cookie()` 函数用于显式声明一个形参应该从 HTTP 请求头中的 `Cookie` 键值对集合中提取解析**。

它除了继承自 Pydantic 的基础校验参数（如 `default`, `description`）之外，还包含了一些专门用于控制**请求体解析行为**和 **OpenAPI/Swagger 文档生成**的高级参数。

`Cookie` 类提供了如下校验方式：

```python
Cookie(
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

> 注：`Cookie` 、`Path` 、`Query` 是**兄弟类**，都继承自共用的 `Param` 类。所以它们的使用方式是一致的！

#### 读取方式

FastAPI 中，读取 Cookie 的写法有 2 种，就像 `Path()\Query()\Body` 一样：

##### Cookie() 散装式包装读取

直接在 API 接口函数中，写**多个 `<Cookie-Key>: <数据类型> = Cookie()`** 来**逐个接收读取并解析注入**，适用于**少量 Cookie** 的操作。

```ini
# HTTP 请求
GET /items HTTP/1.1
Cookie: token=XmpjpqQmpAazkosw;user_id=100368;user_name=Jack;x_agent=Android
Host: 127.0.0.1:8000
```

- ❌️**传统写法 `<Cookie-Key>: <数据类型 | None> = Cookie(None)`**：

  ```python
  @app.get("/items")
  async def read_items(
      token: str = Cookie(...),  # 必填
      user_id: str = Cookie(...),  # 必填
      user_name: str | None = Cookie(None),  # 非必填
      x_agent: str | None = Cookie(default=None)  # 非必填
  ):
      '''
      返回结果：
      {
          "user_id": "100368",
          "user_name": "Jack",
          "token": "XmpjpqQmpAazkosw",
          "x_agent": "Android"
      }
      '''
      return {
          "user_id": user_id,
          "user_name": user_name,
          "token": token,
          "x_agent": x_agent
      }
  ```

- ✅️**现代写法 `<Cookie-Key>: Annotated[<数据类型 | None>, Cookie()] = <默认值 | None>`**：

  ```python
  @app.get("/items")
  async def read_items(
      token: str = Cookie(...),  # 必填
      user_id: str = Cookie(...),  # 必填
      user_name: str | None = Cookie(None),  # 非必填
      x_agent: str | None = Cookie(default=None)  # 非必填
  ):
      '''
      返回结果：
      {
          "user_id": "100368",
          "user_name": "Jack",
          "token": "XmpjpqQmpAazkosw",
          "x_agent": "Android"
      }
      '''
      return {
          "user_id": user_id,
          "user_name": user_name,
          "token": token,
          "x_agent": x_agent
      }
  
  ```

##### Pydantic 模型声明

核心要点：通过**事先定义一个 `Pydantic` 数据模型类**传递**给 API 接口函数的形参 `:` 类型声明，并使用 `Cookie()` 或 `Depends()` 一次性包装** 。

FastAPI 会**自动解析**这个由 `BaseModel` 创建的**数据模型类结构**，并**从 HTTP 请求头中的 `Cookie:` 字段集合中批量读取并解析注入到 API 接口的 `形参变量` 中** 。

> 注：**Cookie 的 `value` 值**可以是**任何值（`int`、`str`、`bool`、`{}` 对象）**，最终都会转为**字符串形式传输**。

###### 模型定义

*适用于多量 Cookie 操作*

HTTP 请求：

```ini
# HTTP 请求
GET /?theme=dark HTTP/1.1
Cookie: token=XmpjpqQmpAazkosw; d_ticket=814610eb6e81e7d52a90d3521912fe66d6c77; user={"user_id":"100368","user_name":"Jack","is_login":"true","roles":["user","admin"]}
Host: 127.0.0.1:8000
```

模型定义：

```python
# 用户数据模型
class UserModel(BaseModel):
    user_id: str = Field(..., title="用户ID")  # 必填
    user_name: str = Field(..., title="用户名")  # 必填
    is_login: bool = Field(False, title="是否登录")  # 非必填
    roles: List[str] = Field(['user'], title="角色列表")  # 非必填


# Cookie 数据模型
class CookieModel(BaseModel):
    token: str = Field(..., title="token")  # 必填
    d_ticket: str | None = Field(None, title="通行证ID")  # 非必填

    # 嵌套模型
    user: UserModel = Field(..., title="用户信息")  # 必填
```

###### Cookie() 批量包装读取

```python
from typing import Annotated

from fastapi import Cookie, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Cookies(BaseModel):
    session_id: str
    fatebook_tracker: str | None = None
    googall_tracker: str | None = None


#❌️传统写法
@app.get("/items/")
async def read_items(cookies: Cookies = Cookie()):
    return cookies

# ✅️现代写法
@app.get("/items/")
async def read_items(cookies: Annotated[Cookies, Cookie()]):
    return cookies
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-Cookie() 参数.png)

###### Depends() 批量包装读取

> **FastAPI 的限制**
>
> **FastAPI 的 `Cookie()` 参数只能绑定到简单类型（str、int、bool等）或 Pydantic 模型，但当 Pydantic 模型包含嵌套模型时，会出现解析问题。**
>
> 原因是：
>
> 1. Cookie 在 HTTP 协议中**只能是字符串**
> 2. FastAPI 尝试将 Cookie 字符串直接转换为 Pydantic 模型
> 3. 对于扁平模型（所有字段都是简单类型），FastAPI 可以自动转换
> 4. 但对于**嵌套模型**，FastAPI 不知道如何将字符串解析为嵌套对象
>
> ```python
> # ❌ 这段代码会失败
> @app.get('/')
> async def dashboard(
>     cookie_model: CookieModel = Cookie(),  # 包含嵌套的 UserModel
> ): 
>     pass
> ```
>
> 当 FastAPI 尝试解析时：
>
> - Cookie 中的 `user` 字段值是字符串 `'{"user_id":"100368",...}'`
> - FastAPI 期望 `user` 是一个字典或 Pydantic 模型实例
> - 类型不匹配 → 报错
>
> 所以需要额外定义一个 `parse_cookie_model` 函数用于**解析 Cookie 的字符串转为 JSON 格式**。
>
> > 实际上，Cookie 更适合存储简单的标识符（如 session_id），复杂的用户信息应该从后端存储（如 Redis/数据库）中获取，而不是直接存在 Cookie 中。

```python
import json

async def parse_cookie_model(
    token: Annotated[str | None, Cookie()] = None,
    d_ticket: Annotated[str | None, Cookie()] = None,
    user: Annotated[str | None, Cookie()] = None,
) -> CookieModel:
    """手动解析 Cookie 中的 JSON 字符串"""
    user_data = {}
    if user:
        try:
            user_data = json.loads(user)
        except json.JSONDecodeError:
            user_data = {}

    return CookieModel(
        token=token or "",
        d_ticket=d_ticket,
        user=UserModel(**user_data)
    )
```

API 接口函数写法：

- ❌️**传统写法 `<形参变量> : <数据模型> = Depends(<解析函数>)`**：

  ```python
  @app.get('/')
  async def dashboard(
      # Query 查询参数
      theme: Annotated[str, Query()] = "dark",
  
      # Cookie 模型
      cookie_model: CookieModel = Depends(parse_cookie_model),
  ): pass
  ```

- ✅️**现代写法 `<形参变量> : Annotated[<数据模型 | None>, Depends(<解析函数>)] = <默认值 | None>`**：

  ```python
  @app.get('/')
  async def dashboard2(
      # Cookie 模型
      cookie_model: Annotated[CookieModel, Depends(parse_cookie_model)],
  
      # Query 查询参数
      theme: Annotated[str, Query()] = "dark",
  ): pass
  ```

### 设置响应 Cookie

服务器端想要向客户端写入一个 Cookie 数据，需要通过 `Response` 响应对象来实现。

#### Response 类

FastAPI 框架提供了一个 **`fastapi.Response`** 类，它是一个**特殊对象（优先级比 Path 路径参数、Query 查询参数更高）**；

核心定义：

​	可以在 **API 接口函数中定义一个 `形参变量: Response`** ；

​	FastAPI 会**根据 `:` 数据类型声明自动识别**这个 **`形参变量` 是一个响应对象**，可以操作它完成一些**响应体的处理（设置 Cookie、Header 头、返回响应数据...）**。

```python
from fastapi import FastAPI, Response

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>')
async def <API 路由函数>(<形参变量x>: Response):
    return <响应数据>
```

##### set_cookie() - 写入响应 Cookie

`fastapi.Response` 类实例提供了一个 **`set_cookie()` 方法**，它可以实现**向客户端写入一个 Cookie 键值对以及相关的配置参数**。

###### 基本语法

```python
response.set_cookie(
	key: str, # Cookie 键
    value: str = "", # Cookie 值
    max_age: int | None = None, # 有效期（秒）
    expires: datetime | str | int | None = None, # 绝对过期时间点（毫秒）
    path: str | None = "/", # Cookie 所属 URL 路径
    domain: str | None = None, # Cookie 所属域名
    secure: bool = False, # 安全传输协议（False: HTTP、True: HTTPS）
    httponly: bool = False, # 安全策略：禁止前端 JavaScript 读取，防 XSS 攻击
    samesite: Literal['lax', 'strict', 'none'] | None = "lax", # 防 CSRF 攻击
    partitioned: bool = False # 独立分区 Cookie（CHIPS 机制）
)
```

核心作用：**向客户端发出的当前 HTTP 请求中写入一个响应Cookie，发回给客户端接收**。

参数简述：

| **参数名**     | **类型**         | **说明**                                                     |
| -------------- | ---------------- | ------------------------------------------------------------ |
| **`key`**      | `str`            | Cookie 的名称                                                |
| **`value`**    | `str`            | Cookie 的值（必须是字符串）                                  |
| **`max_age`**  | `int`            | 生效时间（秒）。倒计时结束时 Cookie 失效。                   |
| **`expires`**  | `int / datetime` | 具体的过期时间点                                             |
| **`path`**     | `str`            | 限制可以访问该 Cookie 的路由路径，默认 `/`（全站可用）       |
| **`domain`**   | `str`            | 限制可以访问该 Cookie 的域名                                 |
| **`secure`**   | `bool`           | 若为 `True`，则只能在 HTTPS 连接中传输                       |
| **`httponly`** | `bool`           | 若为 `True`，前端 JS 无法通过 `document.cookie` 读取，极大增加安全性 |

###### 参数详细说明

📌 **Cookie 的内容**

> - **`key: str`**
>   - **说明**：Cookie 的名称（键）。
>   - **注意**：同一个域名和路径下，`key` 是唯一的。如果重复设置相同的 `key`，后面的值会覆盖前面的。
> - **`value: str = ""`**
>   - **说明**：Cookie 的具体内容（值）。
>   - **注意**：**必须是字符串类型**。如果你想存储复杂的数据（如字典、列表），需要先将其序列化为 JSON 字符串（例如使用 `json.dumps()`）或进行 Base64 编码。

📌 **生命周期参数：控制 Cookie 何时失效**

> - **`max_age: int | None = None`**
>   - **说明**：Cookie 的**相对生存周期**，单位为**秒**。
>   - **行为**：
>     - 例如设置 `max_age=3600`，表示该 Cookie 从浏览器收到开始算起，1 小时后自动失效。
>     - 如果设置为 `0` 或负数，浏览器会立即删除该 Cookie。
>     - 如果为 `None`，则生命周期由 `expires` 决定。
> - **`expires: datetime | str | int | None = None`**
>   - **说明**：Cookie 的**绝对过期时间点**。
>   - **行为**：可以传入一个特定的 `datetime` 对象或符合 HTTP 日期格式的字符串（如 `Wdy, DD Mon YYYY HH:MM:SS GMT`）。
>   - **注意（Session Cookie）**：如果 `max_age` 和 `expires` **同时为 `None`**，该 Cookie 将变成**会话 Cookie（Session Cookie）**。它只保存在浏览器的内存中，当用户关闭浏览器标签页或关闭浏览器时，Cookie 会被自动清除。

**📌 作用域参数：控制 Cookie 哪些请求能带上**

> - **`path: str | None = "/"`**
>   - **说明**：限制可以访问该 Cookie 的 **URL 路径**。
>   - **行为**：默认是 `/`，表示当前域名下的所有路径（如 `/items`、`/users`）都可以读取这个 Cookie。如果设置为 `/api`，那么只有发送到 `/api/v1` 等以 `/api` 开头的请求才会携带该 Cookie。
> - **`domain: str | None = None`**
>   - **说明**：限制可以访问该 Cookie 的 **域名**。
>   - **行为**：默认不设置时，该 Cookie 只对当前精确域名生效。如果设置为 `.example.com`（或 `example.com`），则该 Cookie 对主域名及其所有子域名（如 `app.example.com`、`api.example.com`）都共享。

**📌 安全防范参数：保护 Cookie 不被窃取和滥用**

> - **`secure: bool = False`**
>   - **说明**：安全传输开关。
>   - **行为**：若设置为 `True`，该 Cookie **只能通过 HTTPS 协议**（加密连接）传输。如果用户使用的是明文的 HTTP 协议，浏览器将拒绝发送该 Cookie。
>   - **最佳实践**：生产环境下强烈建议设置为 `True`。
> - **`httponly: bool = False`**
>   - **说明**：防范 **XSS（跨站脚本）攻击**。
>   - **行为**：若设置为 `True`，客户端的 JavaScript（如 `document.cookie`）将**无法读取或修改**该 Cookie。它只能在浏览器发起 HTTP/HTTPS 请求时，由浏览器自动放入请求头中发送给服务器。
>   - **最佳实践**：存储敏感信息（如 Session ID、Token）时，必须设置为 `True`。
> - **`samesite: Literal['lax', 'strict', 'none'] | None = "lax"`**
>   - **说明**：防范 **CSRF（跨站请求伪造）攻击**，控制第三方网站请求时是否携带该 Cookie。
>   - **可选值**：
>     1. **`'lax'`（默认值）**：宽松模式。第三方网站的链接跳转（GET 请求）会带上 Cookie，但第三方网站的诸如 POST 表单提交、AJAX 请求不会带上。在安全性和体验之间取得了平衡。
>     2. **`'strict'`**：严格模式。完全禁止第三方请求携带 Cookie。只有当用户直接在当前域名操作时才会发送。
>     3. **`'none'`**：无限制。任何跨站请求都会带上该 Cookie。**注意**：如果设为 `'none'`，则必须同时将 `secure` 设置为 `True`，否则现代浏览器会拒绝写入该 Cookie。
> - **`partitioned: bool = False`**
>   - **说明**：**独立分区 Cookie（CHIPS 机制）**，为了应对现代浏览器逐步淘汰“第三方 Cookie”而推出的新特性。
>   - **行为**：如果设置为 `True`（且同时设置了 `secure=True`），即使该 Cookie 在第三方上下文中被写入，它也会被绑定到用户当前访问的“顶级网站（Top-level site）”。
>   - **应用场景**：当你开发的 FastAPI 服务作为一个 `<iframe>` 嵌入到别人的网站中，而你又需要利用 Cookie 跟踪这个特定嵌入用户的状态，且不希望被浏览器的第三方 Cookie 禁用策略拦截时，需要开启此参数。

###### 示例

```python
from fastapi import FastAPI, Response

app = FastAPI()

# 写入响应 Cookie
@app.get('/set_cookie')
async def set_cookie(
    response: Response # 声明 response 形参是一个 Response 响应对象
):
    # 向客户端发出的当前请求中写入一个响应Cookie，发回给客户端
    response.set_cookie(
        key="token",
        value="XmpjpqQmpAazkosw",
        httponly=True,
        max_age=3600,
        expires=3600,
        path="/",
        domain="127.0.0.1",
        secure=True,
        samesite="lax"
    )
    return "Success"
```

返回的响应体：

```ini
HTTP/1.1 200 OK
date: Sat, 06 Jun 2026 04:50:40 GMT
server: uvicorn
content-length: 9
content-type: application/json
set-cookie: token=XmpjpqQmpAazkosw; Domain=127.0.0.1; expires=Sat, 06 Jun 2026 05:50:40 GMT; HttpOnly; Max-Age=3600; Path=/; SameSite=lax; Secure
"Success"
```

> 注：**`set_cookie()` 每次**只能写入**一个 Cookie 键值对**，若想要**写入多个**，需**调用多次 `set_cookie()` 方法**。

##### delete_cookie() - 删除 Cookie

`fastapi.Response` 类提供了一个 **`delete_cookie()`** 方法，它可以**在当前会话（HTTP 请求）中把客户端发送过来的某个 Cookie 数据删除**。

核心定义：删除 Cookie 的本质是**让服务器通知客户端（浏览器）某个 Cookie 已经过期了**，即**本质上 `delete_cookie()` 其实就是将 Cookie 的过期时间设为当前**。

###### 基本语法

```python
response.delete_cookie(
    key: str, # Cookie 名称
    path: str | None = "/", # Cookie 所属 URL 路径
    domain: str | None = None, # Cookie 所属域名
)
```

核心作用：

​	根据 **`key` 参数**，在**当前会话（HTTP 请求）**中，**查找与之匹配的 Cookie 键值对**并将其**过期时间设为当前（`Max-Age` 有效期为 0 ）**，以达到**删除**的目的；

​	再将这个 Cookie 字段**发回给客户端**，客户端会**自动屏蔽（或 `删除`）这个过期的 Cookie**，下次请求便**不会再携带它**了。

****

⚠️ **注意**：如果**设置 Cookie** 时**指定了特定的 `path` 或 `domain`**，**删除**时也**必须**传入**相同的 `path` 或 `domain` 参数**，否则浏览器可能无法成功删除该 Cookie。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/Response 删除 Cookie 的底层工作机制.png)

###### 示例

```python
# 删除某个 Cookie
@app.get('/delete_cookie')
async def delete_cookie(
    response: Response
):
    response.delete_cookie(
        key="token",
        domain="127.0.0.1",
        path="/",
    )
    return "Success"
```

返回的响应体：

```ini
HTTP/1.1 200 OK
date: Sat, 06 Jun 2026 05:05:23 GMT
server: uvicorn
content-length: 9
content-type: application/json
set-cookie: token=""; Domain=127.0.0.1; expires=Sat, 06 Jun 2026 05:05:23 GMT; Max-Age=0; Path=/; SameSite=lax
"Success"
```

可以看出：`token` 的值已经没了，`Max-Age` 有效期参数也设为了 0。

#### JSONResponse 类

在 FastAPI 中，虽然可以直接返回一个普通的 Python 字典、列表或者 Pydantic 模型，FastAPI 也会在后台自动把它们转换成 JSON 格式，但我们依然经常需要显式地使用 **`JSONResponse`**。

描述：当需要**更高级的控制权**—比如**自定义响应状态码、自定义响应头（Headers）**、**手动设置 Cookie** 时，使用`JSONResponse`  类

```python
from fastapi.responses import JSONResponse

@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>')
async def <API 路由函数>():
    
    <响应数据> = <...>
    
    response = JSONResponse(content=<响应数据>)
    response.set_cookie(key=<Cookie 名称>, value=<Cookie 值>)
    
    return response # 直接返回 JSONResponse 给客户端
```

##### 示例

业务场景：如果想在返回 JSON 响应数据的**同时**，顺便给浏览器塞一个 Cookie 或者自定义的 Header，需使用 `JSONResponse` 类实例。

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/custom-response/")
async def custom_response():
    content = {"message": "Hello World"}
    response = JSONResponse(content=content)
    response.set_cookie(key="flavor", value="chocolate")
    return response
```

## Header 请求/响应头

在 FastAPI 中，Header 请求/响应头的操作主要分为**读取、设置** 2 个部分，都需要涉及到不同的类。

### 声明读取 Header（请求头）

FastAPI 中的 Header 请求/响应头参数的**读取、值校验**主要通过 **`fastapi.Header` 类** 以及 `Pydantic` 数据模型实现。

- 可以实现对 **HTTP 请求中一个或多个请求头**的值校验处理。 

#### `Header` 类（包装、校验）

##### 基本语法

FastAPI 支持通过在 API 接口函数中的**形参列表**中**定义一个或多个形参，并使用 `Header()` 包装**来**表示接收指定 Header 请求头键值对字段**。

```python
from fastapi import FastAPI, Header

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>')
async def <API 路由函数>(<Header-Key>: <数据类型> = Cookie()):
	# 从 HTTP 请求头中提取与 <Header-Key> 同名的 Header 数据键值对字段
    return <响应数据>
```

> 注：如果**不使用 `Header()`**，FastAPI 会**默认**把 `<Header-Key>` 形参当作一个**Query 查询参数（Query Parameter）**。

###### 命名匹配机制

```ini
<Header-Key>: <数据类型> = Header()
<Header-Key>: Annotated[<数据类型>, Header()]
```

FastAPI 框架会**自动 “从 HTTP请求头 中提取与 `<Header-Key>` （`形参变量`） 同名匹配的 Header 请求头键值对字段，并提取对应的 Header 请求头的字段值注入到 `<Header-Key>` 这个 `形参变量` 中”**。

> 注：**`<Header-Key>` 是不区分大小写**的！
>
> > `user_agent` 既能匹配 `User-Agent`，也能匹配 `user-agent` 或 `USER-AGENT`。

示例：

```ini
# HTTP 请求行
GET /items HTTP/1.1
# HTTP 请求头部分
user-agent: xxx
Host: 127.0.0.1:8000
```

```python
# API 接口函数
@app.get("/items")
async def read_items(
    host: str = Header(None),
    user_agent: str | None = Header(None)
):
	# 从 HTTP请求头 中提取与 <Header-Key> 同名的 Header 请求头键值对字段
    
    # 提取到的 Header 请求头信息：{"Host":"127.0.0.1:8000","User-Agent":"xxx"}
    return {
        "Host": host,
        "User-Agent": user_agent
    }
```

##### 自动转换下划线机制

在 **HTTP 协议**中，**Header 请求/响应头字段**都是以 **`<Xxx-Yxx>: <值>`** 的形式进行传输的，如下：

```ini
Accept: */*
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
Cookie: username=xxx
Host: 127.0.0.1:8000
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: none
Sec-Fetch-Storage-Access: active
___internal-request-id: 1e3c04dd-6b17-4dc4-a10d-ee15561b1a63
sec-ch-ua: "Chromium";v="148", "Google Chrome";v="148", "Not/A)Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
```

大部分标准请求头用**连字符**分隔，即**减号**（`-`）。但是 `user-agent` 这样的变量在 Python 中是**无效**的。

因此，**默认**情况下，FastAPI 会在**读取、写入 Header 请求头字段**时会做如下动作：

- **读取、写入 Header**：**自动**将 **Python 变量名中的下划线（`_`）替换为连字符（`-`）**，从而去**匹配对应同名的 Header 请求头**

示例：

```python
# API 接口函数
@app.get("/items")
async def read_items(
	accept_encoding: str = Header(), # 自动转换为 Accept-Encoding 去匹配
    sec_fetch_site: str = Header() # 自动转换为 Sec-Fetch-Site 去匹配
): pass
```

这一切都是由 `Header()` 的 **`convert_underscores`** 参数控制的。

###### convert_underscores 参数

核心作用：在 **`Header()` 带着 `形参变量` 解析并匹配 Header 请求头字段**时，**是否开启对 Python `形参变量` 的自动转换下划线机制**。

取值：

- **`True`**（默认）：**自动**将 Python **`形参变量名` 的下划线（`_`）替换为连字符（`-`）**，**匹配『原生』的 Header 请求头字段**
- **`False`**：**禁止**自动转换下划线机制，带着 **`形参变量名` 原样去匹配**，可用于**匹配『自定义』的 Header 请求头字段**

示例：

```ini
# HTTP 请求
# 原生 HTTP 请求头
Accept: */*
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
Cookie: username=xxx
Host: 127.0.0.1:8000
...
# 自定义 HTTP 请求头
X_Token: JOPdqjzpjPOJWNnnPIGnIOwngioa
X_Server_Version: 2026.1.0
...
```

```python
# 读取原生 / 自定义的 Header 请求头信息
@app.get('/read')
async def read_header(
    # 原生 HTTP 请求头字段
    accept_encoding: str = Header(),  # 自动转换为 Accept-Encoding 去匹配
    accept_language: str = Header(),  # 自动转换为 Accept-Language 去匹配


    # 自定义 HTTP 请求头字段
    x_token: str = Header(convert_underscores=False),  # 原样拿着 x_token 去匹配
    x_server_version: str = Header(convert_underscores=False)  # 原样拿着 x_server_version 去匹配
):
    '''
    提取到的 Header 请求头信息:
    {
      "accept_encoding":"gzip, deflate, br, zstd",
      "accept_language":"zh-CN,zh;q=0.9",
      "x_token":"JOPdqjzpjPOJWNnnPIGnIOwngioa",
      "x_server_version":"2026.1.0"
    }
    '''

    return {
        "accept_encoding": accept_encoding,
        "accept_language": accept_language,
        "x_token": x_token,
        "x_server_version": x_server_version
    }
```

##### 包装、校验

核心要点：`Header()` 是一个集**位置声明（包装）、安全把关（参数值校验）**和**代码即文档（元数据描述）**于一体的生产力工具。

核心定义：在 FastAPI 中，**`Header()` 函数用于显式声明一个形参应该从 HTTP 请求头中提取解析**。

它除了继承自 Pydantic 的基础校验参数（如 `default`, `description`）之外，还包含了一些专门用于控制**请求体解析行为**和 **OpenAPI/Swagger 文档生成**的高级参数。

`Header` 类提供了如下校验方式：

```python
Header(
	default: Any = Undefined, # 静态默认值
    default_factory: (() -> Any) | None = _Unset, # 动态默认值（工厂）
    
    convert_underscores: bool = True, # 是否开启自动转换下划线机制

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

> 注：`Header` 、`Path` 、`Query` 是**兄弟类**，都继承自共用的 `Param` 类。所以它们的使用方式是一致的！

#### 读取方式

FastAPI 中，读取 Header的写法有 2 种，就像 `Path()\Query()\Body` 一样：

##### Header() 散装式包装读取

直接在 API 接口函数中，写**多个 `<Header-Key>: <数据类型> = Header()`** 来**逐个接收读取并解析注入**，适用于**少量 Header 请求头** 的操作。

```ini
# 请求行
GET /items4 HTTP/1.1
# 请求头
# 原生 HTTP 请求头
Cookie: token=XmpjpqQmpAazkosw; d_ticket=814610eb6e81e7d52a90d3521912fe66d6c77;
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
Host: 127.0.0.1:8000
# 自定义 HTTP 请求头
X_Token: JOPdqjzpjPOJWNnnPIGnIOwngioa
X_Server_Version: 2026.1.0
```

- ❌️**传统写法 `<Header-Key>: <数据类型 | None> = Header(None)`**：

  ```python
  @app.get("/items")
  async def read_items(
      accept_encoding: str = Header(),
      accept_language: str | None = Header(None)
  ): pass
      # 提取到的 Header 请求头信息
      # {"Accept-Encoding":"gzip, deflate, br, zstd","Accept-Language":"zh-CN,zh;q=0.9"}
  ```

- ✅️**现代写法 `<Header-Key>: Annotated[<数据类型 | None>, Header()] = <默认值 | None>`**：

  ```python
  @app.get("/items")
  async def read_items(
      accept_encoding: Annotated[str, Header()],
      accept_language: Annotated[str | None, Header()] = None 
      # ⚠️注意 在使用 Annotated 时，Header 不能传入默认值，必须使用 = None
  ): pass
      # 提取到的 Header 请求头信息
      # {"Accept-Encoding":"gzip, deflate, br, zstd","Accept-Language":"zh-CN,zh;q=0.9"}
  ```

##### Pydantic 模型声明

核心要点：通过**事先定义一个 `Pydantic` 数据模型类**传递**给 API 接口函数的形参 `:` 类型声明，并使用 `Header()` 或 `Depends()` 一次性包装** 。

FastAPI 会**自动解析**这个由 `BaseModel` 创建的**数据模型类结构**，并**从 HTTP 请求头中批量读取 Header 请求头字段并解析注入到 API 接口的 `形参变量` 中** 。

###### 模型定义

*适用于多量 Header 操作*

HTTP 请求：

```ini
# 请求行
GET /items4 HTTP/1.1
# 请求头
# 原生 HTTP 请求头
Cookie: token=XmpjpqQmpAazkosw; d_ticket=814610eb6e81e7d52a90d3521912fe66d6c77;
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
Host: 127.0.0.1:8000
# 自定义 HTTP 请求头
X_Token: JOPdqjzpjPOJWNnnPIGnIOwngioa
X_Server_Version: 2026.1.0
```

模型定义：

```python
# 多个 Header 请求头，Pydantic 模型
class HeadersModel(BaseModel): # 注意：Field() 在实际运行时可有可无，用于 Header 请求头时主要是渲染 Swagger 文档的
    accept_encoding: str = Field()  # 自动转换下划线匹配
    accept_language: str = Field()  # 自动转换下划线匹配
    host: str = Field()  # 自动转换下划线匹配
    connection: str = Field()  # 自动转换下划线匹配
    cookie: str = Field()  # 自动转换下划线匹配

    x_token: str = Header(convert_underscores=False)  # 原样匹配
    x_server_version: str = Header(convert_underscores=False)  # 原样匹配
```

###### Header() 批量包装读取

- ❌️**传统写法 `<形参变量>: <数据模型> = Header()`**：

  ```python
  @app.get("/items3")
  async def read_items3(
      headers: HeadersModel = Header()
  ):
      '''
      接收到的 Header 请求头信息:
      {
          "accept_encoding": "gzip, deflate, br, zstd", 
          "accept_language": "zh-CN,zh;q=0.9", 
          "host": "127.0.0.1:8000", 
          "connection": "keep-alive",
          "cookie": "token=XmpjpqQmpAazkosw; d_ticket=814610eb6e81e7d52a90d3521912fe66d6c77;", 
          "x_token": "JOPdqjzpjPOJWNnnPIGnIOwngioa", 
          "x_server_version": "2026.1.0"
      }
      '''
  
      return headers.model_dump()
  ```

- ✅️**现代写法 `<形参变量>: Annotated[<数据模型>, Header()] = <默认值>`**：

  ```python
  @app.get("/items4")
  async def read_items4(
      headers: Annotated[HeadersModel, Header()]
  ):
      '''
      接收到的 Header 请求头信息:
      {
          "accept_encoding": "gzip, deflate, br, zstd", 
          "accept_language": "zh-CN,zh;q=0.9", 
          "host": "127.0.0.1:8000", 
          "connection": "keep-alive",
          "cookie": "token=XmpjpqQmpAazkosw; d_ticket=814610eb6e81e7d52a90d3521912fe66d6c77;", 
          "x_token": "JOPdqjzpjPOJWNnnPIGnIOwngioa", 
          "x_server_version": "2026.1.0"
      }
      '''
  
      return headers.model_dump()
  ```

### 设置自定义 Header（响应头）

在 FastAPI 中，**“写入一个 Header”** 通常指的是**在后端处理完逻辑后，给客户端（App、浏览器）返回的响应（Response）中添加自定义的 HTTP 响应头**。

在 FastAPI 中，主要有以下 3 种方式来写入响应头，取决于使用场景：

#### Response 类

FastAPI 框架提供了一个 **`fastapi.Response`** 类，它是一个**特殊对象（优先级比 Path 路径参数、Query 查询参数更高）**；

核心定义：

​	可以在 **API 接口函数中定义一个 `形参变量: Response`** ；

​	FastAPI 会**根据 `:` 数据类型声明自动识别**这个 **`形参变量` 是一个响应对象**，可以操作它完成一些**响应体的处理（设置 Cookie、Header 头、返回响应数据...）**。

```python
from fastapi import FastAPI, Response

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>')
async def <API 路由函数>(<形参变量x>: Response):
    return <响应数据>
```

##### headers 属性（动态）

**`Response` 类实例**提供了一个 **`headers` 属性**，该属性内**存储着客户端发送的 HTTP 请求头信息**；

- 可以直接以 **字典** 的形式**修改 `response.headers` 属性**，以达到**动态添加、修改 Header 响应头**的效果

在 API 接口函数返回响应数据时，`Response` 类会**自动将 `headers` 属性内容注入到响应数据的 Header 响应头中发回给客户端**。

```python
# 写入 Header (Response)
# Header 请求头操作
from fastapi import FastAPI, Response

app = FastAPI()

# Response 类方式
@app.get("/set_header")
async def set_header(response: Response):
    # 写入多个响应头，并注入到响应中发回给客户端
    response.headers['X-Token'] = 'JOPdqjzpjPOJWNnnPIGnIOwngioa'
    response.headers["X-Server-Version"] = "2026.1.0"

    return {"message": "Success"}
```

返回的响应：

```ini
HTTP/1.1 200 OK
date: Sat, 06 Jun 2026 09:01:08 GMT
server: uvicorn
content-length: 25
content-type: application/json
# 自定义添加的响应头
x-token: JOPdqjzpjPOJWNnnPIGnIOwngioa
x-server-version: 2026.1.0
```

##### 跨域问题

FastAPI 返回的**响应注入了自定义的 Header**（比如 `X-Token`），前端使用 Axios 或 Fetch 异步请求时**默认是获取不到的**。

- **原因**：浏览器的**安全策略（CORS）限制。**

**解决办法**：必须在**响应头中额外添加**一个 **`Access-Control-Expose-Headers` 响应头**，明确**告诉浏览器允许前端读取这个字段**。

```python
response.headers["Access-Control-Expose-Headers"] = "X-Token"
```

#### JSONResponse 类

在 FastAPI 中，虽然可以直接返回一个普通的 Python 字典、列表或者 Pydantic 模型，FastAPI 也会在后台自动把它们转换成 JSON 格式，但我们依然经常需要显式地使用 **`JSONResponse`**。

描述：当需要**更高级的控制权**—比如**自定义响应状态码、自定义响应头（Headers）**、**手动设置 Cookie** 时，使用`JSONResponse`  类

```python
from fastapi.responses import JSONResponse

@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>')
async def <API 路由函数>():
    
    <响应数据> = <...>
    
    <Header 头数据> = <...>
    
    return JSONResponse(content=<响应数据>, headers=<Header 头数据>) # 直接返回 JSONResponse
```

##### 示例

业务场景：如果想在返回 JSON 响应数据的**同时**，顺便给浏览器塞一个 Cookie 或者自定义的 Header，需使用 `JSONResponse` 类实例。

```python
from fastapi.responses import JSONResponse

app = FastAPI()

# JSONResponse 类方式
@app.get("/set_header2")
async def set_header2():
    # 响应数据
    content = {"message": "Success"}

    # Header 头信息数据
    headers = {
        "X-Token": "JOPdqjzpjPOJWNnnPIGnIOwngioa",
        "X_Server_Version": "2026.1.0"
    }

    # 直接将 JSONResponse 返回
    return JSONResponse(content=content, headers=headers)
```

#### 中间件（middleware）-全局处理

在 FastAPI 中，提供了一个 **`@app.middleware()` 装饰器**，被**它修饰的『普通函数』**会**被所有 `@app.get\post...` API 接口函数执行一遍**，目的是**为了给当前 FastAPI 应用的所有 API 接口路由添加一定的额外功能**。

业务场景：如果希望**所有的 HTTP 请求在 API 接口函数返回响应**时都**带上一些特定的 Header**，可以**使用 `@app.middleware()` 中间件来实现**。

```python
# 中间件函数-全局处理（会被所有 API 接口函数执行一次）
@app.middleware('http')  # 针对 HTTP 请求
async def add_global_headers(request: Request, call_next):
    # 1. 挂起请求，等待 API 接口函数处理完毕后，拦截并获取响应对象
    response = await call_next(request)

    # 2. 在响应中统一写入全局 Header
    response.headers['X-Global-Token'] = 'JOPdqjzpjPOJWNnnPIGnIOwngioa'
    response.headers['X-Security'] = 'Max-Protection'

    # 3. 释放并返回响应给客户端
    return response
```

```python
# 以下 2 个 API 接口函数返回的响应都会带上 @app.middleware('http') 中写入的全局 Header
@app.get('/test_header1')
async def test_header1():
    return {"message": "test1"}

@app.get('/test_header2')
async def test_header2():
    return {"message": "test2"}
```

返回的响应：

```ini
HTTP/1.1 200 OK
date: Sat, 06 Jun 2026 09:24:16 GMT
server: uvicorn
content-length: 21
content-type: application/json
x-token: JOPdqjzpjPOJWNnnPIGnIOwngioa
x_server_version: 2026.1.0
# 全局 Header
x-global-token: JOPdqjzpjPOJWNnnPIGnIOwngioa
x-security: Max-Protection
```

<img src="media/FastAPI 全局中间件-统一响应头概念.png" style="zoom:60%;" />