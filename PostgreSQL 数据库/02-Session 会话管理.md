# Session 会话管理

## psql 命令行工具

### `\conninfo` 查看连接会话信息（psql）

- 在 psql 中执行 `\conninfo`，会显示连接字符串的详细信息

  ```bash
  test=# \conninfo
  You are connected to database "test" as user "postgres" via socket in "/var/run/postgresql" at port "5432".
  ```

## SQL 函数操作

PostgreSQL 提供了如下变量，用于**返回当前 Session 会话的相关信息参数，可使用 `SELECT` 查看**。

### 当前连接的用户

#### `CURRENT_USER` 当前会话用户

- **查看当前会话连接的用户**

  ```postgresql
  SELECT CURRENT_USER;
  
  -- postgres
  ```

#### `SESSION_USER` 会话用户原始名

- **查看建立会话连接时的原始用户名**

  ```postgresql
  SELECT SESSION_USER;
  
  -- postgres
  ```

#### `CURRENT_ROLE` 当前用户的角色

- **查看当前连接用户的角色**

  ```postgresql
  SELECT CURRENT_ROLE;
  
  -- postgres
  ```

### 当前连接的数据库

#### `CURRENT_CATALOG` 函数

PostgreSQL 中一个非常简洁的**内置函数**，它的作用是**返回当前会话所连接的数据库名称**。

```postgresql
SELECT CURRENT_CATALOG;

-- postgres
```

#### `CURRENT_DATABASE()` 函数

与 `CURRENT_CATALOG` 等价；

```postgresql
SELECT CURRENT_DATABASE();

-- postgres

SELECT CURRENT_CATALOG = CURRENT_DATABASE();  -- 返回 t (true)
```

### 当前 Schema 模式

#### `CURRENT_SCHEMA` 函数

- 返回当前 `search_path` 搜索路径中第一个模式（如 `public`）

  ```postgresql
  SELECT CURRENT_SCHEMA;
  
  -- public
  ```

### 当前时间

#### **`CURRENT_DATE` **日期

表示：**当前系统的日期（年-月-日）**

#### **`CURRENT_TIME` **时间

表示：**当前系统的时间（时:分:秒.毫秒）**

#### **`CURRENT_TIMESTAMP` **日期时间

表示：**当前系统的日期时间（年-月-日 时:分:秒.毫秒）**

```postgresql
SELECT CURRENT_DATE AS "当前时间（年月日）"; -- 2026-06-18

SELECT CURRENT_TIME AS "当前时间（年月日-时分秒）"; -- 11:12:07.598553+08

SELECT CURRENT_TIMESTAMP AS "当前时间（年月日-时分秒.毫秒）"; -- 2026-06-18 11:12:07.59884+08
```

