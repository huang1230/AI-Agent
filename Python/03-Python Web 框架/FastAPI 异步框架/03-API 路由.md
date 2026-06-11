# API 路由

## @ 路由装饰器（定义路由）

核心定义：**路由（Route）**是一种将**客户端请求的 URL 地址** 和 **API 接口的视图函数** 绑定在一起的**一组映射关系**。

- **一个 API 路由 = 一个请求 URL 路径地址**

在 FastAPI 中，定义 API 路由主要是通过 **`@<FastAPI 应用实例>.<Http 方法>()` 路由装饰器** 的语法来实现。

### 基本语法

```python
from fastapi import FastAPI

<FastAPI 应用实例> = FastAPI()

# 定义一个路由函数
@<FastAPI 应用实例>.<HTTP 请求方式>(<URL 请求路径>,<参数>,...)
async def <API 路由函数>(): 
    return <响应数据>
```

> 注：FastAPI 会自动把 **API 路由函数**的返回的**响应数据**进行 **JSON 序列化**处理。

#### 示例

一个标准的路由装饰器由以下三部分组成：

```python
@app.get("/users")
```

一个标准的路由装饰器由以下三部分组成：

- **`@app`**：在代码开头创建的 **`FastAPI()` 应用实例对象**
- **`.get`**：**HTTP 方法操作**（也叫操作/Operation），决定了接口接受什么类型的请求
- **`("/users")`**：**请求接口的路径（Path）**，即用户在浏览器或客户端中访问的 URL 相对地址

被 `@` 装饰器包裹的 **API  路由函数**，在 FastAPI 的术语中被称为**路径操作函数（Path Operation Function）**。

#### 支持的 HTTP 方法

FastAPI 为每一种标准的 HTTP 动作都提供了对应的装饰器。根据 **RESTful API** 的设计规范，它们分工明确：

| **装饰器**          | **对应 HTTP 方法** | **常见使用场景**                                   |
| ------------------- | ------------------ | -------------------------------------------------- |
| **`@app.get()`**    | GET                | **读取**数据（如获取商品列表、查看用户资料）       |
| **`@app.post()`**   | POST               | **创建**数据（如注册新用户、提交订单）             |
| **`@app.put()`**    | PUT                | **更新/替换**数据（更新用户的完整表单信息）        |
| **`@app.patch()`**  | PATCH              | **局部更新**数据（比如只修改用户的头像，其他不变） |
| **`@app.delete()`** | DELETE             | **删除**数据（如删除一篇文章、注销账号）           |

示例：

```python
# 引入 FastAPI 模块
from fastapi import FastAPI
# 数据模型基类
from pydantic import BaseModel


# 定义一个 Article 数据模型类
class Article(BaseModel):
    id: int
    title: str
    content: str


# 创建 FastAPI 实例
app = FastAPI()


# 1. 获取文章（GET）
@app.get('/articles')
async def get_articles():
    # 返回一个包含文章的列表
    return {'articles': [Article(id=1, title='title1', content='content1')]}


# 2. 发布文章 (POST)
@app.post("/create_article/")
def create_article(article: Article):
    return {"msg": "文章创建成功", "data": article}


# 3. 修改文章 (PUT)
@app.put("/articles/{article_id}")
def update_article(article_id: int, article: Article):
    return {"msg": f"文章 {article_id} 已更新"}


# 4. 删除文章 (DELETE)
@app.delete("/articles/{article_id}")
def delete_article(article_id: int):
    return {"msg": f"文章 {article_id} 已删除"}


# 启动应用
if __name__ == "__main__":
    import uvicorn

    uvicorn.run('server2:app', host="127.0.0.1", port=8000, reload=True)
```

### API  接口文档参数

除了指定的 **URL 请求路径**，`@` 路由装饰器还支持传入许多参数，用来**控制接口行为**或**丰富自动生成的 API 文档 (Swagger UI)**。

`@` 路由装饰器提供了如下参数，用于**在 `/docs` 自动生成的 API 文档**中进行**规范化丰富 API 接口内容**：

#### tags 接口分组标签

描述：当接口变多时，Swagger 文档会变得杂乱。

核心作用：**默认**情况下，**没有显式设定 `tags` 标签的 API 接口都属于 `default` 分组**，为了规整 API 接口的可视化管理，**`tags` 标签可以为接口进行分组归类**。

```python
# 引入 FastAPI 模块
from fastapi import FastAPI
# 数据模型基类
from pydantic import BaseModel


# 定义一个 Article 数据模型类
class Article(BaseModel):
    id: int
    title: str
    content: str


# 创建 FastAPI 实例
app = FastAPI()


# 1. 获取文章（GET）
@app.get('/articles', tags=['文章管理'])
async def get_articles(): pass


# 2. 发布文章 (POST)
@app.post("/create_article/", tags=['文章管理'])
def create_article(article: Article): pass


# 3. 修改文章 (PUT)
@app.put("/articles/{article_id}", tags=['文章管理'])
def update_article(article_id: int, article: Article): pass


# 4. 删除文章 (DELETE)
@app.delete("/articles/{article_id}", tags=['文章管理'])
def delete_article(article_id: int): pass


# 5. 获取用户（GET）
@app.get('/users', tags=['用户管理'])
async def get_users(): pass


# 6. 其他接口
@app.get('/other')
async def get_other(): pass

# 启动应用
if __name__ == "__main__":
    import uvicorn

    uvicorn.run('server2:app', host="127.0.0.1", port=8000, reload=True)
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-tags文档标签参数.png)

#### summary 接口标题

核心作用：API 接口的**简短摘要（标题）**。（如果不写，则默认把 API 函数名称以 驼峰式写法 转为 API 接口标题）。

```python
# 引入 FastAPI 模块
from fastapi import FastAPI
# 数据模型基类
from pydantic import BaseModel


# 定义一个 Article 数据模型类
class Article(BaseModel):
    id: int
    title: str
    content: str


# 创建 FastAPI 实例
app = FastAPI()


# 1. 获取文章（GET）
@app.get('/articles', tags=['文章管理'], summary="获取文章列表")
async def get_articles(): pass


# 2. 发布文章 (POST)
@app.post("/create_article/", tags=['文章管理'], summary="发布文章")
def create_article(article: Article): pass


# 3. 修改文章 (PUT)
@app.put("/articles/{article_id}", tags=['文章管理'], summary="更新文章")
def update_article(article_id: int, article: Article): pass


# 4. 删除文章 (DELETE)
@app.delete("/articles/{article_id}", tags=['文章管理'], summary="删除文章")
def delete_article(article_id: int): pass


# 5. 获取用户（GET）
@app.get('/users', tags=['用户管理'], summary="获取用户")
async def get_users(): pass


# 6. 其他接口
@app.get('/other', summary="其他接口")
async def get_other(): pass

# 启动应用
if __name__ == "__main__":
    import uvicorn

    uvicorn.run('server2:app', host="127.0.0.1", port=8000, reload=True)
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-summary 接口标题参数.png)

#### description 接口详细描述

核心作用：用于定义 **API 接口的具体描述信息**。*支持 Markdown 语法。*

```python
# 1. 获取文章（GET）
@app.get('/articles', tags=['文章管理'], summary="获取文章列表", description="此接口需要请求头中携带 **Token**，返回用户发布的文章列表。")
async def get_articles(): pass
```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/SwaggerUI 交互式API文档-description 接口描述参数.png)

## `APIRouter` 路由分发（路由分组）

在 FastAPI 中，随着项目结构的变大，把**所有的路由（APIs）都写在一个主文件（`main.py`）** 里会变得非常**臃肿**。

为了解决这个问题，FastAPI 提供了 **`APIRouter` 类**，它可以将**所属同一功能模块的多个子路由（Route）收集在一起**，并**统一设置一个路由的路径前缀（Route Path Prefix）**表示**路由分组**，以此将多个子路由进行**模块化**处理。即**以 功能模块 划分多个子路由**。

### 核心概念

**`APIRouter`** 是**由 `fastapi` 模块提供的一个类**，可以理解为它是一个 “微型的 `FastAPI` 实例”。

核心机制：在 FastAPI 项目中，以 **包（Package）**的形式，在包内**创建多个路由文件（`.py`）**；**一个路由文件（`.py`）表示一个功能模块**，用于**存放一系列子路由（`@ 路由装饰器` + `async def` API 视图函数）**。

核心逻辑：

1. 在**包（Package）**内的**主文件（`.py`）**中通过 **`APIRouter()`** 来**创建一个路由实例**
2. 在 FastAPI 项目的**主入口文件（`server.py`）中**，以 **包（Package）的形式导入**对应的 **`APIRouter()` 路由实例**
3. 通过 **`<FastAPI 应用实例>.include_router()`** 方法**统一注册挂载到主应用的 `FastAPI()` 实例上**

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/APIRouter 路由分发的概念.png)

### 标准项目目录结构

```python
my_fastapi_app/
│
├── app/
│   ├── __init__.py
│   ├── server.py          	 # 主入口，负责组装路由
│   └── routers/         	 # 存放所有路由模块
│	     └── users_api.py    # 用户管理模块-主文件（创建 API Router 路由管理，定义一系列子路由）
│	     └── product_api.py  # 产品管理模块-主文件（创建 API Router 路由管理，定义一系列子路由）
```

### 基本语法

#### APIRouter()--子路由

核心定义：`fastapi` 模块提供了一个 `APIRouter` 类；它可以在**子模块**（`router/xxx.py`）中，通过 **`APIRouter()`** 方法**创建并返回一个 `router` 路由对象。**

- **`router` 路由对象**：包含了 **`get()`、`post()`、`put()`、`delete()` 等 `@路由装饰器` 方法**，可以用于**定义一系列 API 接口路由**

`routers/users.py`：

```python
# 引入 APIRouter 类
from fastapi import APIRouter

# 创建一个路由对象
<路由对象> = APIRouter()


# 定义一系列子路由
@<路由对象>.<HTTP 方法>('<URL 路径>')
async def <API 接口函数>(): pass
```

FastAPI 项目中的主入口文件（`server.py`）可以以 包（Package）的形式引入这个 `router` 路由对象， 并统一注册挂载。

#### include_router()--主应用注册

核心定义：`fastapi.FastAPI` 类构建的 `app` 主应用实例提供了一个 **`include_router()`** 方法。

核心作用：通过 **`app.include_router()`** 方法，将**多个从外部引入的 `router` 路由对象**，**统一挂载到 FastAPI 主应用 `app` 身上**，完成**多个分组路由的统一注册、挂载**。

`server.py`：

```python
# 主入口文件
from fastapi import FastAPI

# 导入路由
from <包> import <路由模块>


# 主应用
app = FastAPI()

# 导入路由，并添加到主应用中
app.include_router(<路由模块>.<router 路由对象>, [其他参数...])


# 启动服务
if __name__ == '__main__':
    import uvicorn

    uvicorn.run(app='server:app', host='127.0.0.1', port=8000, reload=True)
```

当启动服务后，FastAPI 项目便拥有了某个路由。

##### 参数

描述：在主应用中，可以通过 **`include_router()`**  方法提供的一些参数，**统一为多个 `router` 路由对象**在分发时**直接注入通用配置**。

###### prefix 路由前缀

作用：为**当前 `router` 路由下的所有 API 接口**，**添加上统一的 URL 路径前缀**。

```python
# 导入路由，并添加到主应用中，路由前缀为 /users。
app.include_router(<路由模块>.<router 路由对象>, prefix='/users')

# 假设 <路由模块>.<router 路由对象> 路由对象中有一个子路由 /get_info，那么最终访问的实际 URL 路径是 /users/get_info
```

###### tags API 文档分组标签

作用：自动**为当前 `router` 路由下的所有 API 接口**，在自动生成的 API 文档（Swagger UI）中进行**归类分组**。

```python
# 导入路由，并添加到主应用中，路由前缀为 /users，Swagger UI 中显示的标签为 用户管理
app.include_router(<路由模块>.<router 路由对象>, prefix='/users', tags=['用户管理'])

# 导入路由，并添加到主应用中，路由前缀为 /product，Swagger UI 中显示的标签为 商品管理
app.include_router(<路由模块>.<router 路由对象>, prefix='/products', tags=['商品管理'])
```

###### dependencies 依赖注入

作用：可以**为当前 `router` 路由下的所有 API 接口**，**统一挂载一些依赖项**。

例如：整个 `users` 模块都需要登录权限验证。

```python
from fastapi import Depends, APIRouter
from app.dependencies import get_token_header # 一个自定义模块中的 Token 验证函数

app.include_router(
    <路由模块>.<router 路由对象>, 
    prefix=<URL 路径前缀>, 
    tags=<标签>,
    dependencies=[Depends(get_token_header)], # 整个管理模块都需要验证 Token
)
```

###### responses 通用响应

作用：可以**为当前 `router` 路由下的所有 API 接口**，**统一定义一个通用的 JSON 响应数据**。

例如：返回一个通用的 404 错误响应 `responses={404: {"description": "Not found"}}`。

```python
app.include_router(
    <路由模块>.<router 路由对象>, 
    prefix=<URL 路径前缀>, 
    tags=<标签>,
    responses=<JSON 格式的通用响应数据>
)
```

#### 示例

假设目录结构如下：

```python
my_fastapi_app/
│
├── app/
│   ├── __init__.py
│   ├── server.py          	 # 主入口，负责组装路由
│   └── routers/         	 # 存放所有路由模块
│	     └── users_api.py    # 用户管理模块-主文件（创建 API Router 路由管理，定义一系列子路由）
│	     └── product_api.py  # 产品管理模块-主文件（创建 API Router 路由管理，定义一系列子路由）
```

##### 路由模块

- `router/` 统一路由目录（模块）：

  - `user_api.py` 路由：

    ```python
    # 定义用户管理模块的路由
    # 引入 APIRouter 类
    from fastapi import Path, APIRouter
    
    # 创建一个路由对象
    router = APIRouter()
    
    
    # 定义一系列子路由
    @router.get('/', summary='获取用户列表')
    async def get_users():
        return {'message': 'get users'}
    
    
    @router.post('/{id}', summary='创建用户')
    async def update_user(id: int = Path(..., description='用户ID')):
        return "Success"
    ```

  - `product_api.py` 路由：

    ```python
    # 定义用户管理模块的路由
    # 引入 APIRouter 类
    from fastapi import Path, APIRouter
    
    # 创建一个路由对象
    router = APIRouter()
    
    
    # 定义一系列子路由
    @router.get('/', summary='获取产品列表')
    async def get_products():
        return {'message': 'get products'}
    
    
    @router.post('/{id}', summary='创建产品')
    async def update_product(id: int = Path(..., description='产品ID')):
        return "Success"
    ```

##### 主应用

`server.py`：

```python
# 主入口文件
from fastapi import FastAPI

# 导入路由
from routers import users_api, product_api


# 主应用
app = FastAPI()

# 导入路由，并添加到主应用中，路由前缀为 /users，Swagger UI 中显示的标签为 用户管理
app.include_router(users_api.router, prefix='/users',
                   tags=['用户管理'], responses={404: {"description": "Not Found"}})

# 导入路由，并添加到主应用中，路由前缀为 /product，Swagger UI 中显示的标签为 商品管理
app.include_router(product_api.router, prefix='/products',
                   tags=['商品管理'], responses={404: {"description": "Not Found"}})


# 启动服务
if __name__ == '__main__':
    import uvicorn

    uvicorn.run(app='server:app', host='127.0.0.1', port=8000, reload=True)
```

##### API 接口文档（Swagger UI）

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/Python/03-Python Web 框架/FastAPI 异步框架/media/APIRouter 路由分发的 Swagger UI 接口文档.png)

