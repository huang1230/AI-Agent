# HTTP 错误处理

## 前置概念

在 Web 开发中，**HTTP 响应状态码（Response Status Codes）** 是服务器向客户端（浏览器、App、前端）传达请求处理结果的“三位数字暗号”。

> [!NOTE]
>
> 默认情况下，FastAPI 处理请求成功后会自动返回 `200 OK`。但很多时候这并不够语义化：
>
> - 当**创建了一个新用户**时，更规范的状态码是 `201 Created`。
> - 当**没有权限访问**或**资源未找到**时，需要主动抛出 `401 Unauthorized` 或 `404 Not Found`。
>
> 通过正确声明状态码，不仅可以让前端更清晰地处理业务逻辑，FastAPI 还会**自动将这些状态码同步到 `/docs` 交互式文档中**。

核心价值：通过**为不同场景下设置对应的 HTTP 响应状态码及清晰、一致的错误提示**，能使得 **API 更加符合 RESTful 规范、更加健壮**。

### 常见 HTTP 状态码及相关处理

在 Web 开发中，前后端通过 **HTTP 状态码（Status Code）** 达成默契。**后端负责准确地归类错误并抛出对应的状态码**，**前端则根据状态码的高位数字（4xx 或 5xx）进行相应的全局拦截或业务提示**。

- 通常情况下，用于表示一个 **API 接口通信失败的响应状态码**都为 **"4xx"** 或 **"5xx"**。

以下是开发中最常见的 HTTP 错误码，以及前后端标准的处理最佳实践：

#### 4xx 客户端错误

此类错误意味着**问题出在前端/用户身上**，通常是**请求参数有误、未登录**或**没有权限**。

##### 400 Bad Request（错误请求）

> **含义**：这是一个通用的客户端错误。前端发送的请求语法错误，或者不符合后端的业务规则。
>
> **常见场景**：参数缺失、格式错误、业务校验失败（如“用户名已存在”）。
>
> **后端处理**：
>
> - **校验请求参数，一旦不合法立即拦截**
> - 在 Response Body 中返回明确的错误代码（Code）和提示信息（Message）
>
> **前端处理**：
>
> - 读取返回的 `message`，通常通过 Toast 或弹窗（Dialog）直接展示给用户

##### 412 Unauthorized / 401 Unauthorized（未授权/未登录）

> **含义**：当前请求需要用户验证，但用户未提供凭证（如 Token），或者凭证已过期失效。
>
> **后端处理**：
>
> - 在**认证中间件（Middleware）中校验 JWT 或 Session**
> - 如果失效，直接拦截并返回 `401`，不需要进入具体的业务逻辑
>
> **前端处理**：
>
> - **全局拦截器（Axios Interceptor）捕获 `401`**
> - 清除本地缓存的 Token 和用户信息
> - **重定向（跳转）**到 `/login` 登录页面，或者尝试**使用 Refresh Token 刷新凭证**

##### 403 Forbidden（禁止访问/无权限）

> **含义**：服务器理解了你的身份（你已经登录了），但是**你没有权限**访问这个资源。
>
> **常见场景**：普通用户尝试访问管理员的后台数据。
>
> **后端处理**：
>
> - **利用 RBAC（基于角色的权限控制）或 ABAC 进行权限检查（鉴权）**
> - 鉴权失败时，抛出 `403`
>
> **前端处理**：
>
> - **跳转到专门的 `403 无权限` 提示页面**，或者隐藏掉该用户无权点击的按钮

##### 404 Not Found（未找到）

> **含义**：**请求的资源在服务器上不存在**。
>
> **常见场景**：API 路由地址拼写错误，或者查询的用户 ID 在数据库里不存在。
>
> **后端处理**：
>
> - 数据库查询为空时，主动抛出 `404`
>
> **前端处理**：
>
> - 如果是接口报 `404`，说明前后端路由没对上，开发阶段需立即排查
> - 如果是页面报 `404`，跳转到经典的“404 找不到页面"

##### 422 Unprocessable Entity（无法处理的实体）

> **含义**：**请求格式是正确的**（比如是标准的 JSON），但是里面的**数据逻辑、语义不对**（FastAPI 默认的参数校验失败即为此状态码）。
>
> **常见场景**：邮箱格式不合法、密码长度不够、年龄传了负数。
>
> **后端处理**：
>
> - 通过 **Pydantic 等框架自动校验**，并返回具体是哪个字段（Field）出错
>
> **前端处理**：
>
> - 解析错误列表，精确定位到表单对应的输入框，**在输入框下方标红显示错误提示**

#### 5xx 服务端错误

此类错误意味着**问题出在后端/服务器身上**，前端的请求本身可能完全合规。

##### 500 Internal Server Error（服务器内部错误）

> **含义**：**后端代码崩了，抛出了未捕获的异常（Exception）**。
>
> **常见场景**：代码空指针、数据库断开连接、除以零、数组越界。
>
> **后端处理**：
>
> - **绝对不能**把原生的报错堆栈（Stack Trace）返回给前端（防止暴露源码架构）
> - 通过**全局异常处理器捕获，将详细错误记入日志系统（ELK/Sentry）**，给前端返回一句体面的、模糊的提示
>
> **前端处理**：
>
> - **全局拦截，弹出通用的提示**：“服务器开小差了，请稍后再试” 或 “系统繁忙”

##### 502 Bad Gateway（错误网关）/ 504 Gateway Timeout（网关超时）

> **含义**：
>
> - **`502`：作为网关或代理的服务器（如 Nginx）从上游服务器（如 FastAPI/Gunicorn）收到了一个无效的响应**（通常意味着后端服务挂了，进程死了）。
> - **`504`：上游服务器在规定时间内没有完成响应**（通常意味着后端某个计算或 SQL 查询太慢，超时了）。
>
> **后端/运维处理**：
>
> - **排查进程是否存在（是否 OOM 崩溃），优化慢 SQL，调整 Nginx 的超时时间**（`proxy_read_timeout`）
>
> **前端处理**：
>
> - 提示用户“网络连接超时”或“服务正在维护中”，并提供“刷新重试”的按钮

### fastapi.status 类

- `fastapi` 模块提供了一个 `status` 类，该类提供了所有常用的响应状态码值。

常用 `status` 状态码如下：

| **状态码分类**     | **常用状态码与 FastAPI 常量**                                | **含义与应用场景**                                           |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **2xx 成功**       | `200 OK` `status.HTTP_200_OK`                                | 请求成功。最常见的默认返回状态                               |
|                    | `201 Created` `status.HTTP_201_CREATED`                      | 创建成功。常用于 `POST` 接口成功创建了新资源                 |
|                    | `204 No Content` `status.HTTP_204_NO_CONTENT`                | 请求成功但无返回内容。常用于 `DELETE` 删除成功               |
| **4xx 客户端错误** | `400 Bad Request` `status.HTTP_400_BAD_REQUEST`              | 客户端请求参数有误，或不符合业务逻辑限制                     |
|                    | `412 Unauthorized` `status.HTTP_401_UNAUTHORIZED`            | 未登录、未认证，或者 Token 已失效                            |
|                    | `403 Forbidden` `status.HTTP_403_FORBIDDEN`                  | 已登录但没有权限（例如普通用户尝试访问管理员接口）           |
|                    | `404 Not Found` `status.HTTP_404_NOT_FOUND`                  | 请求的资源（如用户、商品 ID）在数据库中不存在                |
| **5xx 服务器错误** | `500 Internal Server Error` `status.HTTP_500_INTERNAL_SERVER_ERROR` | 服务器内部程序崩溃。在 FastAPI 中通常由于未捕获的 Python 代码异常引发 |

```python
from fastapi import status

print(status.HTTP_100_CONTINUE)  # 100
print(status.HTTP_200_OK)  # 200
print(status.HTTP_301_MOVED_PERMANENTLY)  # 301
print(status.HTTP_400_BAD_REQUEST)  # 400
print(status.HTTP_500_INTERNAL_SERVER_ERROR)  # 500
```

## 开始使用

**HTTP 错误处理**指的是**当客户端与服务器端进行 API 接口通信交互时，服务器端的 API 接口程序出现某个异常Bug，或者客户端提交了不准确的 HTTP 请求数据**时，通过**返回对应的「非 200 的 HTTP 响应状态码」来告知客户端 “本次API接口的交互有误”**。

在当今的**前后端分离（API 驱动）架构**中，HTTP 错误处理就是**后端与前端（客户端）之间的一种“契约”**。

FastAPI 提供了**多种层次颗粒度控制**的**错误处理机制**：

- **API 接口路由级控制：**
  - **基础使用：`raise HTTPException(...)`**
  - **自定义异常处理器：`@app.exception_handler(<自定义异常类>)`**
- **全局级捕获控制：**
  - **FastAPI 内置异常处理器**：**`@app.exception_handler(RequestValidationError)`**
  - **全局顶层异常处理器：`@app.exception_handler(Exception)`**


### HTTPException 类（路由级）

在 FastAPI 中，**`fastapi.HTTPException` 是最直接、最常用的错误处理工具类**。

核心机制：当需要向客户端**返回非 200 的响应状态码**时，直接使用 **`raise HTTPException(...)`** 即可。

核心优势：可以**在代码的任何地方（路由函数、依赖项 Dependency、底层业务逻辑）直接抛出它**，FastAPI 会**自动拦截**并将其**转化为规范的 HTTP 响应**，立刻**终止当前请求，不会继续执行**后面的代码。

#### 基本语法

```python
from fastapi import HTTPException

raise HTTPException(
	status_code: int, # 必填。HTTP 响应状态码
    detail: str | list | dict | None = None, # 选填。具体错误信息，可以是一个 str、list、dict
    headers: dict | None = None # 选填。自定义响应头
)
```

核心定义：**`status_code` 、`headers` 用于响应头，`detail` 位于响应体**。

参数说明：

- **`status_code`：HTTP 响应状态码**

  ```python
  HTTPException(status_code=500 | status.HTTP_500_INTERNAL_SERVER_ERROR) # 更推荐使用 fastapi.status 类
  ```

- **`detail`：错误消息的主体**

  FastAPI 会**默认包装成一个 `{ "detail": <具体错误信息> }` 的 JSON 响应体**返回给客户端。可以是如下值：

  - **`str` 字符串**：

    ```python
    HTTPException(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        detail="Server Error"
    )
    
    # 前端收到的响应体：{"detail":"Server Error"}
    ```

  - **`list` 列表**：

    ```python
    HTTPException(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        detail=[
            "Query",
            "role_tag_id"
        ]
    )
    
    # 前端收到的响应体：{"detail":["Query","role_tag_id"]}
    ```

  - **`dict` 字典**：

    ```python
    HTTPException(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        detail={
            "error_code": "AUTH_FAILED",
            "message": "密码错误",
            "hints": "密码通常为6位数字"
        }
    )
    
    '''
    前端收到的响应体：
    {
      "detail": {
        "error_code": "AUTH_FAILED",
        "message": "密码错误",
        "hints": "密码通常为6位数字"
      }
    }
    '''
    ```

- **`headers`：自定义响应头**

  > 某些特定的状态码（如 `401 Unauthorized`）要求必须在响应头中附带信息（如 `WWW-Authenticate`）。

  ```python
  HTTPException(
      status_code=status.HTTP_400_BAD_REQUEST,
      detail="不是管理员或用户",
      headers={"X-Error-Tag": "not-admin-or-user"}
  )
  ```

  返回的响应：

  ```ini
  HTTP/1.1 400 Bad Request
  date: Mon, 08 Jun 2026 13:12:01 GMT
  server: uvicorn
  x-error-tag: not-admin-or-user
  content-length: 37
  content-type: application/json
  {"detail":"不是管理员或用户"}
  ```

##### 基础示例

```python
from typing import Annotated
from fastapi import FastAPI, HTTPException, Query, status

app = FastAPI()


@app.get("/http_exception_test")
async def http_exception_test1(role_tag_id: Annotated[int, Query()] = None):
    match role_tag_id:
        case 1:
            raise HTTPException(
                # 推荐使用 status 模块，比直接写数字 404 更具可读性
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="不是管理员或用户",
                headers={"X-Error-Tag": "not-admin-or-user"}
            )
        case 2:
            raise HTTPException(
                status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
                detail={
                    "code": "internal_server_error",
                    "message": "服务器内部错误",
                },
                headers={"X-Error-Tag": "internal-server-error"}
            )
        case 3:
            raise HTTPException(
                status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
                detail=[
                    "Query",
                    "role_tag_id",
                ],
                headers={"X-Error-Tag": "internal-server-error"}
            )

    # 注意：这里是一个正常的 200 响应
    return {'status':200,'message':'ok'}


# 启动应用
if __name__ == '__main__':
    import uvicorn

    uvicorn.run('server29:app', host="127.0.0.1", port=8000, reload=True)
```

##### 注意点（不要用 return）

核心定义：由于 **`HTTPException` 是一个异常对象**，它**必须由 `raise` 关键字抛出让 FastAPI 顶层去接收处理**。

​	如果**一旦写成了 `return HTTPException()`** ，那么 FastAPI 会**误以为要返回一个普通响应对象**，从而**返回一个 `200 OK` 的响应状态码**，虽然**响应体也能解析成功，但是 `headers` 参数会失效，且不符合规范！**

###### 示例

```python
@app.get("/http_exception_test")
async def http_exception_test1(role_tag_id: Annotated[int, Query()] = None):
    # ...

    # ❌️ 一定不要这样用！
    # return HTTPException(status_code=status.HTTP_200_OK, detail="正常响应", headers={'X-Token': 'xxx'})

    # ✅️ 正确示例
    return {"code": 200, "message": "正常响应", "data": None}
```

`return HTTPException(...)` 返回的响应：

```ini
HTTP/1.1 200 OK
date: Mon, 08 Jun 2026 13:34:08 GMT
server: uvicorn
content-length: 71
content-type: application/json
{"status_code":200,"detail":"正常响应","headers":{"X-Token":"xxx"}}
```

> `return HTTPException()` 也能正常返回，但 `header` 并没有生效；
>
> - 且如果返回的是 `HTTPException(status_code=500)` 给的是 非 200 的响应状态码；那就会这样：
>
>   ```python
>   HTTP/1.1 200 OK # 明明是 500 ，且还是 200 的正常响应
>   date: Mon, 08 Jun 2026 13:35:21 GMT
>   server: uvicorn
>   content-length: 71
>   content-type: application/json
>   {"status_code":500,"detail":"正常响应","headers":{"X-Token":"xxx"}}
>   ```

#### Depends() 依赖注入

FastAPI 的一大特色是**依赖注入系统**，即可以**为一些特定的 API 接口函数动态地注入一些依赖项（函数）**，来实现**高内聚、低耦合**的代码结构，以此**提高代码可读性、可维护性、不可重复性**。

##### 核心思想

- 将**一些重复高可用**的**功能代码提取为一个 `def` 外部函数**作为**依赖项（Dependency）**，依赖项函数还可以**使用形参接收外部`Depends()` 函数传入的一些配置项**，完成一些特定的功能

- 在 **API 接口函数**中通过 **`dependencies` 参数配置 + `Depends()` 函数**组合使用，为 API 接口函数**动态添加一个依赖项**

  - **`dependencies` 参数**：用于**声明当前 API 接口函数**预期**接收一个依赖项函数的特定功能**

  - **`Depends(<外部依赖项函数>)` 函数**：

    ​	用于**解析并执行外部事先预定义的 `def` 外部函数**，并将其**代码执行过程前置**，在 **API 接口函数执行之前对 HTTP 请求作出校验处理**。

  > `Depends()` 执行流程：**拦截 HTTP 请求 → 前置校验执行（外部依赖项函数） → API 接口函数执行**
  >
  > ```css
  > [客户端请求] ──> [FastAPI 拦截 HTTP 请求]
  >                      │
  >                      │
  >                      ▼
  >   		[ 前置校验执行（外部依赖项函数）]
  >                      │
  >       ┌──────────────┴──────────────┐
  >       ▼                             ▼
  > (❌️校验失败: 抛出异常)          (✅️校验通过: 继续执行)
  >                                     │
  >                                     ▼
  >                     			[API 接口函数执行]
  > ```

  以此实现对 API 接口函数 的额外功能规则添加。

##### 具体实现

###### 基本语法

核心机制：在 **`def` 依赖项函数**中**编写 权限校验、数据预检 等处理**，如果**不通过则直接 `raise HTTPException`，拦截请求直接返回响应，请求甚至不会到达 API 接口函数。**

```python
from fastapi import FastAPI

# 依赖项函数，可提前编写对 HTTP 请求的规则校验处理
async def <依赖项函数名>(<形参变量x?>: <数据类型> = <默认值 or Path()\Query()\Body()...修饰>):
    # 请求校验处理...
    # 在这里可以直接写 raise HTTPException(...)，在 API 接口函数执行之前，不通过校验处理时则直接抛出错误响应。
    
@<FastAPI 应用>.<HTTP 方法>('<URL 路径>', dependencies=[Depends(<依赖项函数>)])
```

核心特性：**`Depends()` 会接管 `<依赖项函数名>` 的控制权和解析权**。

​	当 FastAPI **检测到  `<依赖项函数名>` 需要一个参数**时，会**根据 `=<Path()/Query()/Body()/Cookie()/Header()...>` 给定的默认值去 HTTP 请求中对应的位置去提取并注入给 `<形参变量>`**。

<img src="C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/FastAPI-依赖注入-Depends() 解析执行流程.png" style="zoom:70%;" />

###### 示例

```python
# Depends() 依赖注入
# 依赖项函数
async def verify_token(x_token: str = Header(..., description='X-Token 请求头')):
    if x_token != 'super-secret-token':
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail='X-Token 请求头无效'
        )


# API 接口函数，使用 Depends() 依赖注入，先执行 verify_token 函数，校验通过再进入 API 接口函数中
@app.get("/http_exception_test2", dependencies=[Depends(verify_token)])
async def http_exception_test2():
    # 使用 Depends() 依赖注入
    # await verify_token()
    return {"code": 200, "message": "正常响应", "data": None}
```

执行流程：

```
	[客户端请求] ──> [FastAPI 拦截请求]
        	                │
                            ▼
           		[解析 Header 中的 X-Token]
              	            │
                            ▼
       [自动调用 verify_token(x_token=解析出来的Headers值) 校验处理]
                            │
             ┌──────────────┴──────────────┐
             ▼                             ▼
  (校验失败: 抛出HTTPException异常)     (校验通过: 继续执行)
                                           │
                                           ▼
                               [调用 http_exception_test2()]
```

#### FastAPI 参数校验机制的优先级

核心定义：明确一点，FastAPI 的**参数解析与校验的优先级，高于 `Depends()` 依赖项函数的内部校验逻辑**。

核心流程如下：

```css
[ 收到 HTTP 请求 ]
       │
       ▼
1. 检查Header()请求头\Path()、Query()请求参数\Body()请求体是否验证通过
       │
       ├─► 触发 Pydantic 校验失败 ──► 抛出 422 Unprocessable Entity
       │
       ▼ [参数校验通过]
2. 进入 Depends() 依赖项函数内部
       │
       ▼
3. 执行相关业务逻辑判断代码：如 raise HTTPException():` ──► 抛出 xxx 异常
```

##### 关键点

**`Path()\Query()\Body()\Header()...` 是否给定了默认值**：

- **给定了默认值（比如 `...`）**：

  那么一定会**先走 FastAPI 的 Pydantic 参数校验处理**，判断**是否传入了相关的参数，没有传入则直接抛出 422 异常**

- 如果想**绕过这个 Pydantic 参数校验处理**，而是**执行自定义的 `raise HTTPException()` 异常处理业务逻辑**，则需**给`形参变量` 给定一个 `None` 默认值**

##### 示例

假设没有传入 `x-token` 请求头字段：

```python
# 不给默认值：直接触发 FastAPI 的 Pydantic 参数校验机制，抛出 422 异常
async def verify_token(x_token: str = Header(..., description='X-Token 请求头')):
    # 永远不会进入...
        
        
# 给定 None 或其他默认值
async def verify_token(x_token: str = Header(None, description='X-Token 请求头')):
    # 进入....，执行自定义的业务校验逻辑
    if x_token != 'super-secret-token':
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail='X-Token 请求头无效'
        )
```

核心流程：

​	通过设置 `Header\Path\Query...(None)`，FastAPI 的数据解析流水线会认为：“哦，这个 `x-token` 没传也没关系，我传个 `None` 进去就好了。” 这样请求就能顺利通过第一关（Pydantic 校验），成功进入 `verify_token` 函数内部，从而触发自定义的 `raise HTTPException()` 异常处理。

### @app.exception_handler 异常处理器

在 FastAPI 中，提供了一个 **`@app.exception_handler(<异常类>)` 装饰器**，它是一个**全局异常处理器**，用于**修饰一个「异常处理函数」**。

核心要点：异常处理器就像是一个**全局捕获网**。它允许你：**“在底层让代码尽情报错，在顶层由我统一收拾残局”**。

#### 基本结构

想要实现一个**自定义、全局异常处理器**，需要以下 3 个角色共同参与：

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# 自定义异常类
class <异常类>(Exception):
    def __init__(self, <自定义参数...>):
        self.<自定义参数> = <自定义参数...>
        # ...

# 异常处理器
@app.exception_handler(<异常类>)
async def <异常处理函数>(request: Request, exc: <异常类>):
    # 异常处理逻辑...
    
    return JSONResponse(...) # 返回异常响应


# API 接口函数
@app.<HTTP 方法>('<URL 路径>')
async def <API 接口函数>():
    # ...
    if <逻辑判断>:
    	raise <异常类>(<自定义参数...>)
    # ...
```

#### 要素说明

##### **@app.exception_handler(<异常类>)** 

```python
@app.exception_handler(<异常类>)
async def <异常处理函数>(request: Request, exc: <异常类>):
    # 异常处理逻辑...
    
    return JSONResponse(...) # 返回异常响应
```

定义：一个**异常处理装饰器**，用于**修饰一个「异常处理函数」**。

###### 参数说明

- 入参：
  - **`request`：请求对象。FastAPI 会自动调用它，同时允许获取触发错误时的请求信息（如 URL、Header、Method 等）**
  - **`exc`：自定义 或 内置异常类（`RequestValidationError`、`Exception`）**

- 出参：
  - **`JSONResponse()`**：以 **JSON 格式返回响应数据**，可**使用 `exc` 异常类的属性**来**丰富 JSON 响应体的内容**


###### 运行机制

​	FastAPI 中内部维护着一个**字典表（异常类映射表）**，当代码中**使用 `@app.exception_handler(<异常类>)` 装饰一个异常处理函数**时，FastAPI 会**将其传入的 `<异常类>` 和 异常处理函数** 一并**注册**到 **异常类映射表** 中，**等待捕获对应的异常，并进行拦截处理**。

FastAPI 会根据传递的 **`<异常类>` 的继承关系**决定**捕获**的**层级**与**作用域**：

- **`<自定义异常类>`（叶子节点）**：用于**特定的 API 接口函数中使用**，只**影响特定 API 路由的业务逻辑**，返回精准的错误提示
- **`RequestValidationError` 异常类（中间层）**：FastAPI **内置的一个异常处理类**；用于**统一修改 FastAPI 框架默认的报错格式**
- **`Exception`  Python 异常基类（根节点）**：用于**捕获所有未被定义的代码 Bug 异常**，常用于定义一个**全局异常处理器**

##### class <自定义异常类>

定义：

​	用于定义一个**可传入自定义参数的 `<异常类>`**，需**注入到 `@app.exception_handler(<异常类>)` 异常处理器中**，完成一些**特定的功能**（如丰富异常提示、自定义异常处理...）。

##### raise <异常类>(自定义参数...)

定义：用于 **API 接口函数中处理业务逻辑**时，**主动抛出**的一个**`<异常类>`**。

​	当 **API 接口函数 `raise <异常类>()` 主动抛出一个 `<自定义异常>`** 时，FastAPI 内部会去**「异常类映射表（字典）」**中查询匹配，检测**是否注册了当前 `raise` 抛出的 `<自定义异常类>`** ；

- 当**匹配到对应注册的 `<自定义异常类>` **时，会**执行与之关联的 `@app.exception_handler(<异常类>)` 修饰的 异常处理函数**，进行**异常拦截处理（路由级、中间层）**
- 如果**未匹配到对应注册的 `<异常类>`**，则接着会**利用 Python 的继承关系（`issubclass`）**一直**沿着「异常继承链机制 MRO」向上冒泡寻找**它的**异常父类**（如 `Exception`）**是否有注册**相关的异常处理器**（全局处理器）**

<img src="C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/FastAPI-@app.exception_handler 异常处理机制.png" style="zoom:65%;" />

##### 继承关系

核心机制：通过**传入的 Python `<异常类>` 的继承关系**（MRO，Method Resolution Order）来**决定异常捕获的层级与作用域**。

```css
		 		     BaseException (Python 内置)
                                 │
                       Exception (Python 内置)
                                 │
       ┌─────────────────────────┴────────────────────────────┐
       │                    			                      │
 HTTPException (FastAPI/Starlette)【HTTP 异常类】			   └─── CustomException (自定义的异常类)
       │
RequestValidationError (FastAPI)【HTTP 请求校验异常类】
       │
ValidationError (Pydantic)【参数校验异常类】
```

###### `Exception` (Python 内置)

- **定位**：所有内置的非系统退出类异常的基类
- **关系**：它是所有自定义异常、FastAPI 异常的**终极祖先**
- **兜底处理**：如果在 FastAPI 中捕获 `Exception`，它会拦截**所有**未被更具体捕获器处理的运行错误

###### `ValidationError` (Pydantic)

- **定位**：当**数据不符合 Pydantic 模型（`BaseModel`）定义的校验规则**时触发
- **关系**：它**继承自 Python 的 `ValueError`**（而 `ValueError` 继承自 `Exception`）
- **特点**：在 FastAPI 中，如果在**代码内部**手动对 Pydantic 模型进行实例化或数据校验失败，会直接抛出这个异常

###### `RequestValidationError` (FastAPI)

- **定位**：专门用于处理**客户端请求数据**校验失败的异常
- **关系**：它**继承自 Pydantic 的 `ValidationError`**
- **触发场景**：当**客户端发送的请求参数（Path, Query, Body）不符合在路由函数中声明的类型**时，FastAPI 内部会捕获 Pydantic 的 `ValidationError`，并将其包装为 `RequestValidationError` 抛出
- **默认行为**：FastAPI 默认自带了一个 `request_validation_exception_handler`，会将该异常转化为 `422 Unprocessable Entity` 响应返回给前端

###### `HTTPException` (FastAPI / Starlette)

- **定位**：用于**向客户端返回特定 HTTP 状态码和错误信息的异常**
- **关系**：**继承自 Python 的 `Exception`**
- **注意点**：FastAPI 的 `HTTPException` 继承自 `starlette.exceptions.HTTPException`。如果要在代码中主动给前端返回如 `404 Not Found` 或 `403 Forbidden`，应该直接抛出这个异常

###### 自定义异常类 (Custom Exception)

- **设计标准**：通常直接**继承自 `Exception`（如果它属于业务逻辑错误）或 `HTTPException`（如果它需要直接对应某个 HTTP 状态码）**

#### class 自定义异常处理器

FastAPI 支持通过 **自定义一个异常类 +  `@app.exception_handler(<自定义异常类>)` + `raise <自定义异常类>`** 的组合写法。

##### 基本语法

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# 自定义异常类
class <自定义异常类>(Exception | HTTPException): # 继承的异常父类取决于业务逻辑，通常是 Exception
    def __init__(self, <自定义参数...>):
        self.<自定义参数> = <自定义参数...>
        # ...

# 异常处理器
@app.exception_handler(<自定义异常类>)
async def <异常处理函数>(request: Request, exc: <自定义异常类>):
    # 异常处理逻辑...
    
    # 返回自定义的异常响应
    return JSONResponse(
    	status_code=<异常响应状态码>,
        content=<响应体>
    ) 


# API 接口函数
@app.<HTTP 方法>('<URL 路径>')
async def <API 接口函数>():
    # ...
    if <逻辑判断>:
        raise <自定义异常类>(<自定义参数...>)
    # ...
```

核心价值：在**特定的 API 接口函数**中实现**自定义的业务校验异常逻辑**（例如：余额不足、密码错误），并**统一配置它们的响应返回格式**。

核心特点：如果 **API 接口函数**中 **`raise`** 关键字抛出的异常**不是一个自定义异常类**，则会**沿着调用栈 + Python 异常类的继承关系 向上冒泡找到顶层异常父类（`Exception`）**，执行**全局异常拦截处理**。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/FastAPI 异常处理机制.png)

**注意异常流**：如果使用了中间件（Middleware），需要注意中间件里的错误处理和 FastAPI 异常处理器的触发顺序（通常**核心路由内的错误会被异常处理器先捕获**）。

##### 示例

代码示例：

```python
from fastapi import FastAPI, Form, Request, status
from fastapi.responses import JSONResponse

app = FastAPI()


# 自定义异常类（表单提交相关）
class FormFieldException(Exception):
    # 自定义异常信息与依赖项
    def __init__(self, field_name: str, error_msg: str):
        self.field_name = field_name
        self.error_msg = error_msg


# 异常处理器：关联 异常类 + 异常处理函数，注册到 FastAPI 的内部映射表（Dict）中
@app.exception_handler(FormFieldException)
# 当 API 接口函数中 raise 抛出 FormFieldException 异常时，会调用此异常处理函数
async def form_field_exception_handler(request: Request, exc: FormFieldException):

    # 构建自定义的 JSON 响应数据（HTTP 报文），返回给客户端接收
    return JSONResponse(
        status_code=status.HTTP_400_BAD_REQUEST,
        content={
            "code": 400,
            "message": f"[{exc.field_name}] 字段, {exc.error_msg}" # 使用自定义异常类的属性丰富响应数据
        }
    )


# API 接口函数
@app.post("/login")
async def login_api(
    name: str = Form(..., description='用户名'), age: int = Form(..., description='年龄')
):
    # 检查表单字段，如果不符合要求，抛出自定义异常，执行异常处理函数（form_field_exception_handler）
    if not name:
        raise FormFieldException('name', '不能为空')
    if age < 18:
        raise FormFieldException('age', '不能小于18岁')

    # 返回正常响应数据
    return {'code': 200, 'message': 'success'}


# 启动应用
if __name__ == '__main__':
    import uvicorn

    uvicorn.run('server30:app', host="127.0.0.1", port=8000, reload=True)
```

发送的请求：

```ini
POST /login HTTP/1.1
Content-Length: 16
Content-Type: application/x-www-form-urlencoded
Host: 127.0.0.1:8000
name=jack&age=10
```

返回的响应：

```ini
HTTP/1.1 400 Bad Request
date: Tue, 09 Jun 2026 03:20:55 GMT
server: uvicorn
content-length: 56
content-type: application/json
{"code":400,"message":"[age] 字段, 不能小于18岁"} # 自定义错误提示与响应信息
```

##### 底层工作机制

###### 一、基于 ASGI 中间件的异常捕获

​	FastAPI 本质是一个符合 **ASGI 标准**的 Web 框架；当**一个 HTTP 请求进来**时，并不是直接进入 API 路由接口函数中，而是**先经过一层层的 中间件（Middleware）**。

​	在这些中间件里，有一个非常核心的内置中间件叫做 **`ServerErrorMiddleware`**（或者在更上层的 `ExceptionMiddleware`）。

​	它的**核心伪代码结构逻辑**其实非常经典，就是**一个巨大的 `try...except` 块**：

```python
# 框架底层的核心伪代码逻辑示意
async def __call__(self, scope, receive, send):
    try:
        # 继续向后执行：穿过其他中间件，最终到达指定的 API 接口路由函数
        await self.app(scope, receive, send)
    except Exception as exc:
        # ❗ 关键点：一旦 API 接口路由函数抛出了任何异常，都会被这里捕捉到
        await self.handle_exception(exc, scope, receive, send)
```

###### 二、`raise` 的异常冒泡机制

当在 **API 接口函数**中**执行 `raise FormFieldException('name', '不能为空')` **时，会执行如下流程：

1. **异常向上冒泡**

   ​	Python 的 `raise` 关键字会立即**中断**当前函数的执行。如果 API 接口函数内没有 `try...except`  异常处理代码块捕获的话，**抛出的 `FormFieldException` 异常**会**沿着调用栈（CallStack）向上层层抛出（冒泡）**。

2. **异常中间件捕获**

   ​	异常一路向上抛出，直到**被 `ExceptionMiddleware` 中间件的 `try...except Exception as exc` 捕获到**；

   ​	此时，FastAPI 框架拿到 API  接口函数中抛出的 `FormFieldException('name', '不能为空')` 异常对象。

3. **注册表查找匹配与多态查找**

   当在业务代码中编写了如下代码时：

   ```python
   # 异常处理器：关联 异常类 + 异常处理函数，注册到 FastAPI 的内部映射表（Dict）中
   @app.exception_handler(FormFieldException)
   # 当 API 接口函数中 raise 抛出 FormFieldException 异常时，会调用此异常处理函数
   async def form_field_exception_handler(request: Request, exc: FormFieldException):
   
       # 构建自定义的 JSON 响应数据（HTTP 报文），返回给客户端接收
       return JSONResponse(
           status_code=status.HTTP_400_BAD_REQUEST,
           content={
               "code": 400,
               "message": f"[{exc.field_name}] 字段, {exc.error_msg}" # 使用自定义异常类的属性丰富响应数据
           }
       )
   ```

   ​	FastAPI  会**将 `FormFieldException` 异常类 和 `def form_field_exception_handler()` 函数 关联**起来**建立一个 `{ k:v }`映射关系**，并**存储**在内部的一个 **Dict 字典（异常类映射表）**中。大致结构如下：

   ```python
   app.exception_handlers = {
       RequestValidationError: dict_error_handler,
       HTTPException: http_exception_handler,
       FormFieldException: form_field_exception_handler,  # ✨ 自定义异常在这里建立了映射
   }
   ```

   ​	`ExceptionMiddleware` 中间件在捕获到 `raise FormFieldException()` 异常类后，会**在这个 Dict 字典（异常类映射表）中查找对应的 `def 异常处理函数`** ：

   1. 它会首先**通过 `type(FormFieldException)`** 发现该**异常类的类型是 `FormFieldException`**

   2. 检查 Dict 字典（异常类映射表）刚好**对应了 `form_field_exception_handler()` 异常处理函数**

      > 注：如果 Dict 字典（异常类映射表）里**没有直接匹配的类**，FastAPI 还会**利用 Python 的继承关系（`issubclass`）**，去**查找它的父类（比如 `Exception`）有没有注册处理器**。

###### 三、调用处理器并返回响应

​	`ExceptionMiddleware` 中间件会**通过 `await form_field_exception_handler()` 执行它**，并**将当前请求的 `Request` 对象 和 `FormFieldException()` 异常对象作为参数传递进去**；

​	最后，在 `form_field_exception_handler()` 异常处理函数中，**执行并返回一个 `JSONResponse(...)` **，中间件会将其返回的 Response 响应对象**序列化为一个 HTTP 响应报文，发回给前端客户端**。

​	至此，整个异常处理流程完毕。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/FastAPI 异常处理底层机制.png)

#### FastAPI 内置异常处理器

FastAPI 内部有一些默认的异常处理器：

- **`HTTPException`（HTTP 响应码）**
- **`RequestValidationError`（请求体/参数校验失败时触发）**

可以通过**重写**这些 **FastAPI 内置异常处理器**来**统一处理或美化 API 接口函数返回的错误响应格式**。

##### 示例

```python
from typing import Annotated
from fastapi import FastAPI, Form, Request, status
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel, Field

app = FastAPI()


# 重写 FastAPI 内置异常处理器（RequestValidationError）
# 统一处理或美化，当客户端提交的请求体/参数不符合校验规则时，返回的错误响应数据
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    # exc.errors() 包含了所有的错误信息，可以将其简化
    errors = [{"field": err["loc"][-1], "msg": err["msg"]}
              for err in exc.errors()]

    # 构建自定义的 JSON 响应数据（HTTP 报文），返回给客户端接收
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
        content={
            "code": status.HTTP_422_UNPROCESSABLE_CONTENT,
            "message": {
                "status": "fail",
                "errors": errors
            }
        }
    )


# 请求表单模型与规则
class LoginModel(BaseModel):
    # 当请求参数校验不通过时，会被 FastAPI 内置异常处理器（RequestValidationError）捕获到
    name: Annotated[str, Field(title='用户名', max_length=10)]
    password: Annotated[str, Field(
        title='密码', description="必须是数字 + 大小写字符", pattern="^[a-zA-Z0-9]+$")]


# API 接口函数
@app.post("/login")
async def login_api(
    login_body_model: Annotated[LoginModel, Form(..., description='登录模型参数')]
):

    # 返回正常响应数据
    return {'code': 200, 'message': 'success'}


# 启动应用
if __name__ == '__main__':
    import uvicorn

    uvicorn.run('server31:app', host="127.0.0.1", port=8000, reload=True)
```

发送的请求：

```ini
POST /login HTTP/1.1
Content-Length: 31
Content-Type: application/x-www-form-urlencoded
Host: 127.0.0.1:8000
name=jack&password=169! # password 不符合校验规则
```

返回美化后的响应：

```ini
HTTP/1.1 422 Unprocessable Content
date: Tue, 09 Jun 2026 06:50:31 GMT
server: uvicorn
content-length: 125
content-type: application/json
{"code":422,"message":{"status":"fail","errors":[{"field":"password","msg":"String should match pattern '^[a-zA-Z0-9]+$'"}]}} # 返回的是自定义的错误响应
```

原来的响应：

```ini
HTTP/1.1 422 Unprocessable Content
date: Tue, 09 Jun 2026 06:53:26 GMT
server: uvicorn
content-length: 178
content-type: application/json
{"detail":[{"type":"string_pattern_mismatch","loc":["body","password"],"msg":"String should match pattern '^[a-zA-Z0-9]+$'","input":"169！","ctx":{"pattern":"^[a-zA-Z0-9]+$"}}]}
```

#### Exception 全局异常处理器

核心定义：通过为**全局应用兜底捕获一个未定义的 Bug 异常（通常是 `500 Error`）**，以构建更健壮的 Web 应用。

核心机制：通过**重写 `Exception` 全局异常处理器**。

> ⚠️ **注意**：在捕获 `Exception` 时，务必将真正的错误记录到日志中（Log），否则排查 Bug 会变得极其困难。

```python
import logging
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse

app = FastAPI()
logger = logging.getLogger('uvicorn.error')


# 配置日志
MY_LOGGING_CONFIG = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "default": {
            "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s",
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "default",
        },
    },
    "loggers": {
        "uvicorn.error": {
            "level": "INFO",
            "handlers": ["console"],
            "propagate": False,
        },
    },
}


# 定义一个全局异常处理器（Exception），用于捕获并处理任何未被定义捕获的 Bug 异常
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    # 1. 记录详细的日志供后台排查
    logger.error(f'{request.url} -- {exc}', exc_info=True)

    # 构建自定义的 JSON 响应数据（HTTP 报文），返回给客户端接收
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={"message": "系统内部错误，请稍后再试或联系管理员。"},
    )


# API 接口函数
@app.post("/login")
async def login_api():

    return 1 / 0  # 触发 ZeroDivisionError，会被全局异常处理器（Exception）捕获


# 启动应用
if __name__ == '__main__':
    import uvicorn

    uvicorn.run('server32:app', host="127.0.0.1", port=8000, reload=True, log_config=MY_LOGGING_CONFIG)
    # log_config：以 Log 日志模式启动应用
```

