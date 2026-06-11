# Request 请求对象

> [!NOTE]
>
> ​	**`Path()/Query()/Body()`** 在**FastAPI 基于 Starlette 底层框架的 `Request` 对象之上**的一个**上层操作方式（语法糖）**，额外提供了**自动类型转换、Pydantic 数据模型的参数校验、Swagger 文档渲染...**等功能，其底层本质上也是**对 `Request` 对象进行操作**。

## 基本概念

在 FastAPI 中， **`Request` 对象**是由 **Starlette（底层 Web 框架）**所**提供**的一个**核心组件**。

- **`Request`** 对象代表了**客户端发来的原生 HTTP 请求**，包含了**Path 查询参数、Query 查询参数、Body 请求体、Cookie、Header 请求头...**等原始数据

虽然 **FastAPI 提倡使用依赖注入（如 `Query`, `Path`, `Body`）来自动解析参数**，但在某些高级场景下（如获取原始请求头、处理不固定格式的 Body、获取客户端真实 IP 等），直接使用 `Request` 对象会更加**灵活**。

## 基本语法

前提：**由 `fastapi` 模块导入 `Request` 对象**。

核心定义：

- FastAPI 支持通过在 **API 接口函数**中的**形参列表**中**使用 `<形参变量>:Request` 类型声明包装**来**表示接收一个 Request 请求对象**

- FastAPI 会**自动识别 `Request` 类型声明**，并将**当前会话的 HTTP 请求内容注入到 `<形参变量>` 中**。

```python
from fastapi import FastAPI, Request

<FastAPI 应用实例> = FastAPI()

@<FastAPI 应用实例>.<HTTP 方法>('<URL 路径>')
async def <API 路由函数>(<形参变量x>: Request):
    
    return <响应数据>
```

> 注：如果**不使用 `:Request` 类型声明**，FastAPI 会**默认**把 `<形参变量x>` 当作一个**Query 查询参数（Query Parameter）**。

`Request` 对象提供了丰富的属性和异步方法来提取请求数据：

## 属性

`Request` 请求对象中包含的属性、方法可以以**一个 HTTP 请求的结构**进行**分类**：

### 请求行

```ini
# HTTP 请求
GET /1/2?page=1&limit=10 HTTP/1.1
Connection: keep-alive
Host: 127.0.0.1:8000
```

| 属性名              | 描述                                                         | 返回值（示例）                                          |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| **`scope['type']`** | **请求协议**                                                 | **`http`、`websocket`**                                 |
| **`method`**        | HTTP **请求方式**                                            | **`GET`、`POST`、`PUT`、`HEAD`、`DELETE`、`OPTION`...** |
| **`url`**           | HTTP 请求 **URL 全路径**                                     | http://127.0.0.1:8000/1/2?page=1&limit=10               |
| **`base_url`**      | HTTP 请求 **URL 根路径**                                     | http://127.0.0.1:8000/1/2                               |
| **`path_params`**   | HTTP 请求的 **Path `路径参数（字典）**   | `{'id': '1', 'version_id': '2'}` |                                                         |
| **`query_params`**  | HTTP 请求的 **Query 查询参数（字符串）**                     | `page=1&limit=10`                                       |

> 核心机制：在 FastAPI 中，**`path_params` 和 `query_params` 最终都会转换为一个 `{}` 字典形式**来操作，所以可以**使用 `items()`、`get()`、`keys()`、`values()` **等字典方法来操作数据字段。

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.get('/{id}/{version_id}')
async def read_root(
    request: Request,
    id: int = Path(),
    version_id: int = Path(),
    page: int = Query(),
    limit: int = Query()
):
    # 请求行信息
    print(request.scope['type'])  # http
    print(request.method)  # GET
    print(request.url)  # http://127.0.0.1:8000/1/2?page=1&limit=10
    print(request.base_url)  # http://127.0.0.1:8000/

    # --- Path 路径参数 ---
    print(request.path_params)  # {'id': '1', 'version_id': '2'}
    
    # --- Query 查询参数 ---
    print(request.query_params)  # page=1&limit=10
    
    print(request.query_params.get('page'))  # 获取查询参数 page 的值 1
    print(request.query_params.get('limit'))  # 获取查询参数 limit 的值 10

    ...
```

### 请求头

```ini
# HTTP 请求
GET /get_headers HTTP/1.1
Cookie: token=XmpjpqQmpAazkosw; d_ticket=814610eb6e81e7d52d6c77;
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9
X_Token: JOPdqjzpjPOJWNnnPIGnIOwngioa
X_Server_Version: 2026.1.0
Connection: keep-alive
Host: 127.0.0.1:8000
```

> 核心机制：在 FastAPI 中，**请求头最终都会转换为一个 `{}` 字典形式**来操作，所以可以**使用 `items()`、`get()`、`keys()`、`values()` **等字典方法来操作数据字段。

#### `headers` 请求头字段

描述： 一个大小写不敏感的**字典（`Headers` 对象）**，用于**获取所有 Header 请求头内容**。

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.get('/get_headers')
async def get_headers(request: Request):
    # 请求头信息（字典）
    print(request.headers)

    # Headers({'host': '127.0.0.1:8000', 'connection': 'keep-alive', 'x_server_version': '2026.1.0', 'sec-ch-ua-platform': '"Windows"', 'x_token': 'JOPdqjzpjPOJWNnnPIGnIOwngioa', 'accept-language': 'zh-CN,zh;q=0.9', 'sec-ch-ua': '"Chromium";v="148", "Google Chrome";v="148", "Not/A)Brand";v="99"', 'sec-ch-ua-mobile': '?0', '___internal-request-id': '4222c4cd-51e2-451f-9aa2-56a6dd29b6b5', 'accept': '*/*', 'sec-fetch-site': 'none', 'sec-fetch-mode': 'cors', 'sec-fetch-dest': 'empty', 'sec-fetch-storage-access': 'active', 'accept-encoding': 'gzip, deflate, br, zstd', 'cookie': 'token=XmpjpqQmpAazkosw; d_ticket=814610eb6e81e7d52d6c77;'})

    # 获取 Headers 的 Key-Value 键值对
    for key, value in request.headers.items():
        print(f"{key}: {value}")

    '''
    host: 127.0.0.1:8000
    connection: keep-alive
    x_server_version: 2026.1.0
    sec-ch-ua-platform: "Windows"
    x_token: JOPdqjzpjPOJWNnnPIGnIOwngioa
    accept-language: zh-CN,zh;q=0.9
    sec-ch-ua: "Chromium";v="148", "Google Chrome";v="148", "Not/A)Brand";v="99"
    sec-ch-ua-mobile: ?0
    ___internal-request-id: 4222c4cd-51e2-451f-9aa2-56a6dd29b6b5
    accept: */*
    sec-fetch-site: none
    sec-fetch-mode: cors
    sec-fetch-dest: empty
    sec-fetch-storage-access: active
    accept-encoding: gzip, deflate, br, zstd
    '''

    # get(key=): 根据 key 获取指定 Header 的 value 值
    print(request.headers.get('host'))  # 127.0.0.1:8000
    print(request.headers.get('x_server_version'))  # 2026.1.0
    
    ...
```

#### `cookies` Cookie 凭证

描述：一个包含**所有 Cookie** 的标准**字典**。

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.get('/get_cookies')
async def get_cookies(request: Request):
    # cookie 信息（字典）
    print(request.cookies)
    # {'token': 'XmpjpqQmpAazkosw', 'd_ticket': '814610eb6e81e7d52d6c77'}

    # 获取 Cookie 的 Key-Value 键值对
    for key, value in request.cookies.items():
        print(f"{key}: {value}")

    '''
    token: XmpjpqQmpAazkosw
    d_ticket: 814610eb6e81e7d52d6c77
    '''

    # get(key=): 根据 key 获取指定 Cookie 的 value 值
    print(request.cookies.get('token'))  # XmpjpqQmpAazkosw
    print(request.cookies.get('d_ticket'))  # 814610eb6e81e7d52d6c77

    ...
```

### 请求体

`Request` 的请求体都是**调用协程异步方法**来实现的，由于**读取请求体涉及到 I/O 操作**，所以**需要使用 `await`关键字**。

> 核心机制：在 FastAPI 中，**请求体最终都会转换为一个 `{}` 字典形式**来操作，所以可以**使用 `items()`、`get()`、`keys()`、`values()` **等字典方法来操作数据字段。

#### body() 原始二进制数据

核心作用：**异步**获取请求体的**原始二进制字节码数据（`bytes`）**。

> 注：要求 Content-Type 为 **`application/json`**。

```ini
# HTTP 请求
POST /get_body HTTP/1.1
Content-Length: 56
Content-Type: application/json
Host: 127.0.0.1:8000
# 请求体
{"user_id": "1001","user_name": "jack","user_age": "18"}
```

代码示例：

```python
from fastapi import FastAPI, Request
import json

app = FastAPI()

@app.post('/get_body')
async def get_body(request: Request):
    # 获取二进制格式的原始请求体（异步方法，需等待传输完成才能获取到）
    raw_body = await request.body()

    # b'{"user_id": "1001","user_name: "jack","user_age": "18"}'
    print(raw_body)

    # 转换为 Python 字典格式
    dict_body: Dict = json.loads(raw_body)
    print(dict_body)
    # {'user_id': '1001', 'user_name': 'jack', 'user_age': '18'}

    # 获取 Body 请求体 的 Key-Value 键值对
    for key, value in dict_body.items():
        print(f"{key}: {value}")

    '''
    user_id: 1001
    user_name: jack
    user_age: 18
    '''

    # get(key=): 根据 key 获取指定 Body 请求体字段的 value 值
    print(dict_body.get('user_id'))  # 1001
    print(dict_body.get('user_name'))  # jack
    print(dict_body.get('user_age'))  # 18

    ...
```

#### json() 字典格式

核心作用：`Request` 对象**自动将原始的二进制字节码请求体解析为一个 Python 字典并返回**，并**异步**获取。

```ini
# HTTP 请求
POST /get_body HTTP/1.1
Content-Length: 56
Content-Type: application/json
Host: 127.0.0.1:8000
# 请求体
{"user_id": "1001","user_name": "jack","user_age": "18"}
```

代码示例：

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.post('/get_json')
async def get_json(request: Request):
    # 将请求体解析为一个 Python 字典并返回（异步方法，需等待传输完成才能获取到）
    dict_body = await request.json()

    print(dict_body)

    # 获取 Body 请求体 的 Key-Value 键值对
    for key, value in dict_body.items():
        print(f"{key}: {value}")

    '''
    user_id: 1001
    user_name: jack
    user_age: 18
    '''

    # get(key=): 根据 key 获取指定 Body 请求体字段的 value 值
    print(dict_body.get('user_id'))  # 1001
    print(dict_body.get('user_name'))  # jack
    print(dict_body.get('user_age'))  # 18

    ...
```

#### form() 获取表单数据

核心作用：`Request` 对象**自动将原始的二进制字节码请求体解析为一个 `FormData` 表单对象并返回**，并**异步**获取。

> 注：要求 Content-Type 为 **`multipart/form-data`**。

关键点：需额外引入 **`fastapi.datastructures.FormData` 类型**来方便类型提示。

##### **普通表单数据**

```ini
# HTTP 请求
POST /get_from_data HTTP/1.1
Content-Length: 385
Content-Type: multipart/form-data;boundary=------FormBoundaryShouldDifferAtRuntime # 数据传输格式
Host: 127.0.0.1:8000

# 请求体
------FormBoundaryShouldDifferAtRuntime
Content-Disposition: form-data; name="account"

jack
------FormBoundaryShouldDifferAtRuntime
Content-Disposition: form-data; name="password"

123456
------FormBoundaryShouldDifferAtRuntime--
```

代码示例：

```python
from fastapi.datastructures import FormData
from fastapi import FastAPI, Request

app = FastAPI()

# 普通 FormData 表单数据
@app.post('/get_from_data')
async def get_from_data(request: Request):
    # 等待获取请求体中的 FormData 表单数据
    form_data: FormData = await request.form()

    print(form_data)  # FormData([('account', 'jack'), ('password', '123456')])

    # 获取 Body 请求体 的 Key-Value 键值对
    for key, value in form_data.items():
        print(f"{key}: {value}")

    '''
    account: jack
    password: 123456
    '''

    # get(key=): 根据 key 获取指定 Body 请求体字段的 value 值
    print(form_data.get('account'))  # jack
    print(form_data.get('password'))  # 123456
    ...
```

##### **带文件的混合表单数据**

```ini
# HTTP 请求
POST /get_from_data HTTP/1.1
Content-Length: 12995
Content-Type: multipart/form-data;boundary=------FormBoundaryShouldDifferAtRuntime # 数据传输格式
Host: 127.0.0.1:8000

# 请求体
------FormBoundaryShouldDifferAtRuntime
# 普通表单数据
Content-Disposition: form-data; name="account"

jack
------FormBoundaryShouldDifferAtRuntime
Content-Disposition: form-data; name="password"

123456
------FormBoundaryShouldDifferAtRuntime
# File 文件表单数据
Content-Disposition: form-data; name="avatar"; filename="1.png"
Content-Type: image/png

[message-part-body; type: "image/png", size: 12620 bytes]
------FormBoundaryShouldDifferAtRuntime--
```

代码示例：

> 注：**文件处理**需额外**引入 `fastapi.UploadFile` 类**以方便类型提示。

```python
from fastapi.datastructures import FormData
from fastapi import FastAPI, Request, UploadFile
import aiofiles

app = FastAPI()

# 带文件的 FormData 表单数据
@app.post('/get_from_data_file')
async def get_from_data_file(request: Request):
    form_data: FormData = await request.form()

    print(form_data)
    # FormData([('account', 'jack'), ('password', '123456'), ('avatar', UploadFile(filename='1.png', size=12620, headers=Headers({'content-disposition': 'form-data; name="avatar"; filename="1.png"', 'content-type': 'image/png'})))])

    # 获取 Body 请求体 的 Key-Value 键值对
    for key, value in form_data.items():
        print(f"{key}: {value}")

    '''
    account: jack
    password: 123456
    avatar: UploadFile(filename='1.png', size=12620, headers=Headers({'content-disposition': 'form-data; name="avatar"; filename="1.png"', 'content-type': 'image/png'}))
    '''

    # get(key=): 根据 key 获取指定 Body 请求体字段（表单数据）的 value 值
    print(form_data.get('account'))  # jack
    print(form_data.get('password'))  # 123456

    # ---文件处理---
    avatar: UploadFile = form_data.get('avatar')
    print(avatar)
    # UploadFile(filename='1.png', size=12620, headers=Headers({'content-disposition': 'form-data; name="avatar"; filename="1.png"', 'content-type': 'image/png'}))

    # 获取文件名
    print(avatar.filename)  # 1.png

    # 获取文件大小
    print(avatar.size)  # 12620

    # 获取文件头信息
    print(avatar.headers)
    # Headers({'content-disposition': 'form-data; name="avatar"; filename="1.png"', 'content-type': 'image/png'})
    
    # 读取文件内容并保存到本地
    file_content = await avatar.read()
    async with aiofiles.open(f"./{avatar.filename}", 'wb') as f:
        # open() 不支持 async with（不支持异步上下文管理器协议），需使用 aiofiles 第三方库
        print(file_content)
        f.write(file_content)
    ...
```

#### ⚠️ 流式数据机制

核心机制：**`await request.body()` 或 `request.json()` 默认只能读取一次**。

核心原因：Starlette 的 Body 请求体是**基于流（Stream）读取**的。

- 一旦在某个地方**读取了 `await request.json()`**，流就会被**耗尽**。后续**再次调用**就会**卡住或报错**。

解决方案：如果必须**提前读取 Body 请求头的流数据**，并**缓存到 `Request` 对象中的自定义属性**中。

```python
from fastapi.datastructures import FormData
from fastapi import FastAPI, Request, UploadFile

app = FastAPI()

@app.post('/test_api')
async def test_api(request: Request):
    # 提取读取
    body = await request.body()

    # 重新包装流，以便后续路由还能读到
    async def receive():
        return {"type": "http.request", "body": body, "more_body": False}

    # 缓存
    request._receive = receive
```

### `scope` 底层字典

在 FastAPI（以及它底层的 Starlette 框架）中，**`request.scope` 是一个底层的 Python 字典（`dict`）**，它包含了当前这个 HTTP 请求的**所有 ASGI 原始上下文信息**。

简单来说，FastAPI 是基于 **ASGI（异步服务器网关接口）** 标准构建的。当一个请求到达服务器时，ASGI 服务器（如 Uvicorn）会创建一个名为 `scope` 的字典，里面记录了这次连接的所有元数据。

- FastAPI 的 **`Request` 对象**，本质上就是**对这个 `scope` 字典**进行的**高级封装**。

一个典型 HTTP 请求的 `scope` 结构如下：

```python
{
    'type': 'http',                  # 协议类型：http 或 websocket
    'asgi': {'version': '3.0', 'spec_version': '2.3'},
    'http_version': '1.1',           # HTTP 版本
    'server': ('127.0.0.1', 8000),   # 服务器 IP 和端口
    'client': ('127.0.0.1', 54321),  # 客户端 IP 和端口
    'scheme': 'http',                # 协议：http 或 https
    'method': 'GET',                 # 请求方法
    'root_path': '',                 # 应用的根路径
    'path': '/items/42',             # 请求路径（未解码）
    'raw_path': b'/items/42',        # 原始二进制路径
    'query_string': b'active=true',  # 原始查询字符串（Bytes）
    'headers': [                     # 原始请求头列表（元组形式）
        (b'host', b'127.0.0.1:8000'),
        (b'user-agent', b'Mozilla/5.0...')
    ],
    'app': <fastapi.applications.FastAPI object at 0x...>, # FastAPI 实例本身
    'router': <fastapi.routing.APIRouter object at 0x...>, # 路由对象
    'endpoint': <function read_item at 0x...>,             # 最终处理请求的函数
    'path_params': {'item_id': '42'}                       # 路径参数
}
```

```python
# scope 属性（Starlette 底层框架属性）
@app.get('/get_scope')
async def get_scope(request: Request):

    print(request.scope['client'])  # 客户端地址
    print(request.scope['method'])  # 请求方法
    print(request.scope['path'])  # 请求路径
    print(request.scope['raw_path'])  # 请求路径
    print(request.scope['scheme'])  # 请求协议
    print(request.scope['query_string'])  # 请求参数
    print(request.scope['root_path'])  # 根路径
    print(request.scope['server'])  # 服务器地址
    print(request.scope['http_version'])  # HTTP 版本
    print(request.scope['asgi'])  # ASGI 属性
    ...
```

#### 与 Request 对象的关系

类似于 `request.method` 或 `request.headers`...，在底层其实都是**去读取或解析 `request.scope` 里的数据**。

例如：

- 当调用 `request.method` 时，内部执行的是 `return self.scope["method"]`
- 当调用 `request.path_params` 时，内部执行的是 `return self.scope.get("path_params", {})`

### 其他环境属性

| 属性名       | 描述                                             |
| ------------ | ------------------------------------------------ |
| **`app`**    | 所属的 FastAPI 实例                              |
| **`client`** | 包含客户端 IP 和端口的 `Client` 对象（命名元组） |

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.get('/get_other')
async def get_other(request: Request):
    # 其他属性
    # 所属的 FastAPI 实例 <fastapi.applications.FastAPI object at 0x0000018844C60C20>
    print(request.app)
    print(request.client)  # 客户端地址 Address(host='127.0.0.1', port=64975)

    print('客户端IP地址：', request.client.host)  # 127.0.0.1
    print('客户端端口：', request.client.port)  # 64975

    ...
```

## 方法

在 FastAPI 中的 **Starlette 底层框架**提供了如下方法，用于**编写底层中间件、开发长连接（如 SSE/WebSocket）**及**优化网络性能**的场景中。

### is_disconnected() 客户端检测

描述：用于在服务器进行耗时的大型计算、数据库查询或流式数据传输时，**实时检测前端/客户端是否已经关闭了网页或取消了请求**。

> [!NOTE]
>
> 业务场景：如果一个复杂的导出任务需要运行 10 秒，而用户在第 2 秒就关闭了浏览器。如果没有 `is_disconnected()` 检查，服务器仍会傻傻地继续跑完剩下的 8 秒计算，白白浪费 CPU 和内存资源。

#### 代码示例

```python
import asyncio
from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/heavy-task")
async def heavy_task(request: Request):
    for i in range(10):
        # 每次循环都检查一下用户还在不在
        if await request.is_disconnected():
            print("用户已经离线，立即停止耗时任务！")
            return
            
        await asyncio.sleep(1) # 模拟耗时 1 秒的计算
        print(f"正在处理步骤 {i}...")
        
    return {"status": "done"}
```

### stream() 分块流式接收

FastAPI 的 `Request` 对象提供了一个 **`request.stream()`** 方法。

- 核心作用：**分块流式接收客户端上传的超大文件**，防止服务器在接收请求时内存溢出

> 在 FastAPI 中，流式传输共有 2 个模型：
>
> - **服务器端 → 客户端**：使用 **`fastapi.StreamingResponse`** 类**分块流式传输**（音视频流传输、大模型 API 流式文本（SSE）...）
> - **客户端 → 服务器端**：使用 **`fastapi.Request.stream()`** 方法**分块流式接收**（接收超大文件）

```python
from fastapi import FastAPI, Request

app = FastAPI()

# 流式分块接收客户端上传的超大文件数据，防止服务器内存溢出
@app.post('/upload_file')
async def upload_file(request: Request):

    with open('test.pdf', 'ab') as f:
        # 每次只从网路通道中读取 256 KB 的数据（分块流式读取）
        async for chunk in request.stream():
            print(f"收到数据块，大小: {len(chunk)} 字节")
            '''
            收到数据块，大小: 262144 字节
            收到数据块，大小: 262144 字节
            收到数据块，大小: 262144 字节
            收到数据块，大小: 223714 字节
            ...
            收到数据块，大小: 0 字节
            '''
            # 边接收边写入本地硬盘
            f.write(chunk)

    return {'Message': '上传成功！'}
```



