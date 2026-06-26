# Schema 模式操作

## 核心概念

核心定义：**Schema 模式**是一种**将数据库中根据业务逻辑所相关的表、视图、函数或其他数据库对象，分组在同一个逻辑命名空间下**的操作方式。

- **Schema 模式是数据库内部的逻辑命名空间 或 “分类文件夹”**
- **一个数据库中可以有多个 Schema 模式表示一个有相关特性的组织划分，一个 Schema 模式中可以有多张 Table 表。**

核心机制：**根据不同业务功能特性，将存储数据划分到同一相关业务功能特性的逻辑命名空间中管理。**

<img src="C:/Users/win/Desktop/学习笔记体系/AI Agent/PostgreSQL 数据库/media/Schema 模式的基本概念.png" style="zoom:50%;" />

语义化：**`<Schema 模式>.<Table 表>`**

### 核心描述

​	PostgreSQL 可以让**一个数据库中设定多个 Schema 模式，每个 Schema 模式中可以定义同一业务相关的 Table 表、View 视图、函数...**，将这些数据库对象**组织联合在一起存储到同一个逻辑命名空间（Schema 模式）中**，可以**很方便地存储、查询同一个相关业务的数据结果**，以保证**业务数据**的**功能完整性**。

​	通过 **Schema 模式的逻辑分组**，让**庞大的数据库表结构变得井然有序、命名不冲突、且极易进行权限管理**，这种 **“数据库 → 模式 → 表”** 的架构设计在应对多部门的大型应用系统中非常高效。

> 比如：
>
> 一个公司的数据库系统中通常会有很多张表，如果没有 Schema 模式，这些庞大数量的表都是**扁平化管理**，极难维护；
>
> 而有了 Schema 模式，则可以根据**存储数据的不同特性**，以 **功能分类** 的形式进行**结构化管理**：
>
> - 员工相关：`employees` 员工表、`salary` 工资表、`shukkin` 出勤表...
> - 业务相关：`orders` 订单表、`products` 商品表、`users` 用户表...
>
> 假设公司的 HR 部门可能只需要员工相关（员工表、工资表、出勤表）的数据，则将这些表划分在一个 `hr` 模式下管理，直接通过 `hr.employees` 找到员工表操作；
>
> 而技术团队则需要商城业务相关（订单表、商品表、用户表）的数据，则将这些表划分在一个 `mall` 模式下管理，直接通过 `mall.orders` 找到订单表进行操作。
>
> ...以此类推。
>
> <img src="C:/Users/win/Desktop/学习笔记体系/AI Agent/PostgreSQL 数据库/media/Schema 模式架构.png" style="zoom:60%;" />

### 核心作用

1. **在物理之上的 “业务逻辑划分”**：

   ​	**在硬盘上，所有的数据都可能存在同一个数据库文件里**。但 **Schema 模式在应用层根据业务功能建立了一堵 ”隔离墙“**，可以**把不同业务的表进行分类管理**。

   例如：

   - `hr.employees` （HR 部分的员工表）
   - `finance.salaries`（财务部分的薪资表）

   虽然这些表都在同一个数据库里，但通过 `hr.` 和 `finance.` 前缀（Schema 模式），业务逻辑被隔离划分地非常清楚。

   > PostgreSQL 可以**根据 `<Schema 模式名称>.<Table 表名>` 精确地找到某些 Schema 模式下对应的 Table 表**

2. **解决表的 “重名” 冲突（命名空间）、结构化管理**：

   如果没有 Schema 模式，在同一个数据库中，所有的表都必须叫不同的名字，且都在一个空间中进行**扁平化管理**，很难维护。

   > 例如：电商业务有一张 `orders` 订单表，积分商城业务也有一张 `orders` 订单表，只能通过修改 `points_mall_orders` 这样的命名来区分，难以阅读，且表的分布很广泛、扁平，通常打开数据库一下子列出来很长的表。

   有了 Schema 模式之后，可以这样命名：

   - **`mall.orders`**（电商系统的订单表）
   - **`points_mall.orders`**（积分系统的订单表）

   它们可以同时存在，虽然两张表都叫 `orders` ，但却**由于有了 Schema 模式的命名空间隔离，所以实际上它们互不干扰**，整体展现出**结构化管理**形式。

3. **方便 “权限控制”（安全隔离）**：

   PostgreSQL 支持**为不同 Schema 模式分配不同的权限**，从而更**轻松地管理「某些数据库用户」可以查看、修改的表内容**。

   > 在实际开发中，不应该让所有人看到所有的表，基本都会根据用户的权限来分配能查看、修改的表。
   >
   > PostgreSQL 支持：
   >
   > - 给**财务系统的账号**只分配 `finance` 这个 Schema 的读写权限
   >
   > - 给**前端商品展示账号**只分配 `mall` 这个 Schema 的只读权限
   >
   > 通过对不同 Schema 模式的权限分配，就不用去给成百上千张表逐一配置权限了，极大地简化了数据库的管理。

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/PostgreSQL 数据库/media/Schema 模式的核心作用.png)

### 常见数据库的 Schema 差异

不同的数据库对 Schema 的实现理解稍微有一点点不同：

| **数据库**              | **关系层级**                        | **特点描述**                                                 |
| ----------------------- | ----------------------------------- | ------------------------------------------------------------ |
| **PostgreSQL / Oracle** | `Database` ➔ **`Schema`** ➔ `Table` | 最标准的实现。一个数据库下可以创建多个 Schema，每个 Schema 下包含多张表。 |
| **MySQL / MariaDB**     | `Database` **就是** **`Schema`**    | 在 MySQL 中，`CREATE DATABASE` 和 `CREATE SCHEMA` 是完全等价的同义词。它没有严格的三层结构。 |
| **SQL Server**          | `Database` ➔ **`Schema`** ➔ `Table` | 也是标准的独立层级，默认的 Schema 叫 `dbo`。                 |

## 默认模式（`public`）

核心定义：**PostgreSQL 默认提供了一个 `public` 全局模式**，**所有的 Table 表默认都放在 `public` 模式下管理**。

- 在实际场景中，最佳实践是**创建和使用自定义 Schema 模式**，并在**新建 Table 表时指定要划分的其他 Schema 模式，如果不指定，则默认放在 `public` 默认模式中**。

## 模式结构的定义（DDL）

在 PostgreSQL 中，**模式（Schema）是数据库内的一个命名空间**，可以把它想象成存放数据库对象（如表、视图、函数等）的“文件夹”。它主要用于逻辑组织数据和权限管理，能有效避免对象名称冲突。

PostgreSQL 对 Schema 模式的操作主要围绕**创建 (CREATE)**、**修改 (ALTER)** 和**删除 (DROP)** 这三个核心命令展开。

> 核心要点：在 PostgreSQL 中，最佳实践是**把 Schema 模式当做 Database 数据库来使用**。

### `CREATE` 创建模式

- 普通创建一个 Schema 模式

  ```postgresql
  CREATE SCHEMA <模式名>; -- 如果已存在，则报错
  
  CREATE SCHEMA IF NOT EXISTS <模式名>; -- 如果不存在，则创建
  ```

  可以**在 `psql` 命令行工具中使用 `\dn` 查看当前数据库下的所有模式**。

- **创建一个 Schema 模式，同时指定授权给某个用户**

  ```postgresql
  CREATE SCHEMA IF NOT EXISTS <模式名> AUTHORIZATION <用户名>; -- 用户名 与 模式名 不同
  
  CREATE SCHEMA IF NOT EXISTS AUTHORIZATION <用户名>; -- 用户名 与 模式名 相同
  ```

  <img src="C:/Users/win/Desktop/学习笔记体系/AI Agent/PostgreSQL 数据库/media/Schema 模式列表.png" style="zoom:80%;" />

- **创建一个 Schema 模式，并在该 Schema 模式下创建一张表、视图、授权用户...**

  ```postgresql
  CREATE SCHEMA <模式名>
      CREATE TABLE <表名> (<字段名> <数据类型约束>,...)
      CREATE VIEW <视图名> AS <SELECT 查询子句>
      GRANT USER <用户名>;
      
  -- 示例：
  CREATE SCHEMA company
      CREATE TABLE employees (id INT, name TEXT)
      CREATE VIEW staff AS SELECT * FROM employees;
  ```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/PostgreSQL 数据库/media/Schema 创建模式的同时创建表、视图.png)

### `ALTER` 修改模式

当 Schema 模式已创建，需要调整其配置时，可以使用 `ALTER SCHEMA` 命令。

如下：

- **重命名模式**

  ```postgresql
  ALTER SCHEMA <模式名> RENAME TO <新模式名>;
  ```

- **更改模式的所有者用户**

  ```postgresql
  ALTER SCHEMA <模式名> OWNER TO <其他用户>;
  ```

  示例：

  ```postgresql
  -- 示例：
  CREATE USER hkj WITH PASSWORD '123456'; -- 创建用户
  
  SELECT CURRENT_USER; -- postgres
  
  ALTER SCHEMA company OWNER TO hkj; -- 将 company 模式授权给 hkj 用户
  
  -- 根据 pg_authid 系统用户表，查询出与之相关的 pg_catalog.pg_namespace 系统模式表中的模式、所属用户角色
  SELECT 
      n.nspname AS "模式名",
      p_a.rolname AS "所有者角色名"
  FROM 
      pg_catalog.pg_namespace n
  LEFT JOIN 
      pg_catalog.pg_authid p_a ON n.nspowner = p_a.oid;
  
  -- 查询结果：
  public	    			pg_database_owner
  storages				postgres
  information_schema		postgres
  pg_catalog				postgres
  pg_toast				postgres
  company	    			hkj
  ```

### `DROP` 删除模式

⚠️注：删除 Schema 模式是一个不可逆的操作，会移除模式内所有数据。

如下：

- **删除一个空模式（模式中没有任何表、视图...对象）**

  ```postgresql
  DROP SCHEMA <模式名>;
  
  -- 两者等价
  
  DROP SCHEMA IF EXISTS <模式名>; -- 增加 IF EXISTS 判断：如果存在、避免风险
  ```

- **选择性删除 Schema 模式**

  ```postgresql
  DROP SCHEMA IF EXISTS <模式名> <CASCADE | RESTRICT>;
  ```

  - **`CASCADE` (级联) ：强制删除，把该模式下所有数据库对象（表、视图、函数...）一并全部删除**

    > 即不管该模式下有没有数据，都彻底删除干净。

  - **`RESTRICT` (限制) ：如果该模式下定义了下属的数据库对象（如表、视图等），则拒绝该语句的执行。当该模式没有任何下属的对象时才能执行**。

    > 即如果该模式下有数据，那么则会执行出错，必须要先将此模式下的所有数据删除完，才能删除该模式。

## `search_path` 搜索路径

PostgreSQL 内部提供了一个 **`search_path` （搜索路径）的模式寻路机制**。它决定了**当不指定某个 Schema 模式而直接查询某个数据库对象（表、视图、函数...）时，PostgreSQL 应该如何去 “智能查找” 到对应的目标**。

### 为什么需要 `search_path` ？

假设一个数据库中有两张表，分别属于不同的 Schema 模式：

- `production.users` （生产环境的用户表）
- `test.users`（测试环境的用户表）

如果直接执行：

```postgresql
SELECT * FROM users;
```

PostgreSQL 内部会自动通过**`search_path`（搜索路径）**来**决定查找哪个模式**。

### `SHOW search_path` 查看值

可以执行查看 `search_path` 存储的是什么内容：

```postgresql
SHOW search_path;

-- "$user", public
```

默认情况下，**`search_path` 存储的是 `"$user", public`**，代表了 **PostgreSQL 查找数据库对象（表、视图、函数...）的顺序路径**。

### 模式查找机制

当执行：

```postgresql
SELECT * FROM users;
```

PostgreSQL 会**根据 `search_path` 当前的值（路径）去逐条查找**：

- **`pg_catalog`（不可见）**：PostgreSQL 会**第一时间去 `pg_catalog` 系统目录空间里**，**查找用户是否想要引用某个系统表**。

  > 表现形式：当**使用 `SELECT * FROM pg_namespace`** 时，其实 PostgreSQL 内部是**自动补全了 `pg_catalog.pg_namespace` ，所以才能找到**。

- **`"$user"` （动态变量）**：如果不是系统表，则**去与当前用户名同名的 Schema 模式里查找**

  > 表现形式：如果当前登录的数据库用户是 `alice`，PostgreSQL 就会去查找名为 `alice` 的 Schema；如果换成 `bob` 登录，它就会去查找名为 `bob` 的 Schema。如果同名 Schema 不存在，它会直接跳过，不会报错。

- **`public`（全局模式）**：以上两种路径都找不到，则**去 `public` 全局 Schema 模式里查找**

如果以上 3 种模式里都**找不到要操作的数据库对象（Table 表、View 视图...）**，则 **PostgreSQL 报错**。

> ```
> pg_catalog（系统目录空间） → "$user"（当前用户同名的 Schema 模式） → public（全局模式）
> ```

![](C:/Users/win/Desktop/学习笔记体系/AI Agent/PostgreSQL 数据库/media/search_path 搜索路径查找机制.png)

### 设置搜索路径（切换当前模式）

核心作用：**设置 `search_path`（搜索路径）之后，可以让 PostgreSQL 知道该怎么按什么顺序查找对应操作的数据库对象（表、视图、函数...）**。

#### 核心要点

​	**一个数据库中有多个 Schema 模式**，**不同 Schema 模式之间**是**隔离**的，如果**设置了 `search_path` （搜索路径）**，那么 PostgreSQL **只会去 `search_path` 中对应的 Schema 模式中去查找**，所以**设置 `search_path`（搜索路径）**等同于**切换当前 Schema 模式**。

> 例如：假设一个数据库中有以下 3 个模式：
>
> - `marketing` 模式：`customers` 会员表
> - `finance` 模式：`customers` 会员表
>
> ```postgresql
>  SET search_path TO marketing, public; -- 设置了 search_path 搜索路径
> 
>  SELECT CURRENT_SCHEMA; -- marketing
> 
>  SELECT * FROM customers; -- 则只会去 marketing 和 public 模式中查找 customers 表，不会去 finance 模式
> ```
>
> <img src="C:/Users/win/Desktop/学习笔记体系/AI Agent/PostgreSQL 数据库/media/search_path 搜索路径概念.png" style="zoom:60%;" />

#### 设置方式

如果觉得每次都写 `marketing.xxx` 太麻烦，可以先用 `SET search_path` 切换“目的地”，然后直接建表、查询等操作。

有 2 种方式：

##### `SET` 临时设置（仅当前会话）

核心作用：如果希望**在当前连接中，优先按 `schema_a` → `public` 模式的顺序查找**，可以这样设置：

```postgresql
SET search_path TO schema_a, public;
```

表示：**SQL 查询会优先从 `schema_a` 中查找数据库对象（表、视图、函数...）**，并且**新创建的数据库对象（如 `CREATE TABLE...、CREATE VIEW...,`） 都会默认放在 `schema_a` 模式中**。

```postgresql
-- 1. 把搜索路径切到 marketing
SET search_path TO marketing;

-- 2. 直接建表，此时这张表会自动落入 marketing 模式中
CREATE TABLE customers (
    id INT,
    name VARCHAR(50)
);

-- 3. 查找 marketing.customers 表，如果找不到再去 public 模式里找
SELECT * FROM customers;
```

如果有**多个 Schema 模式**时，可以把 `search_path` 配置成一个**列表**，让 Postgres 跨模式寻找对象：

```postgresql
SET search_path TO finance, marketing, public;

SELECT * FROM users JOIN customers ON ...

-- 先去 finance 模式中找，再去 marketing 模式中找，最后去 public 模式中找
```

以此，可以在编写 SQL 语句时，直接**扁平化操作表名、视图名、函数名**...

##### 永久设置（数据库、用户）

如果**不想每次连接都设置 `search_path`（搜索路径）**，可以进行**持久化设置**：

- 针对整个数据库：

  ```postgresql
  ALTER DATABASE testdb SET search_path TO finance, marketing;
  
  -- testdb 数据库中有 3 个模式：
  -- 		finance 模式：customers 会员表
  --		marketing 模式：customers 会员表
  -- 		public 模式：user 员工表
  
  SELECT * FROM user; -- 报错：关系 "users" 不存在（public 模式已经被排除）
  
  SELECT * FROM customers; -- 先找到的是 finance.customers 表。
  ```

- **针对某个特定用户**：

  ```postgresql
  ALTER USER myuser SET search_path TO myschema, public;
  ```

#### 最佳实践（排除 public 模式）

默认情况下，PostgreSQL 中的**任何用户都可以往 `public` 模式里创建数据库对象（表、视图、函数...）**。

**如果 `search_path` 中包含 `public`**，**恶意用户**可能会**创建一张与系统表或核心业务表同名的病毒表，从而导致 “安全隐患”（路径劫持）**。

所以，最好的做法是：

- **在每次 SQL 操作数据库对象（表、视图、函数...）时，都显式指定所属的 Schema**：

  ```postgresql
  SELECT * FROM marketing.customers;
  
  SELECT * FROM finance.customers;
  ```

- **直接通过 `SET ... TO` 排除 `public`，只留下业务核心模式**：

  ```postgresql
  SET search_path TO finance, marketing; -- 只查找 finance, marketing 两个模式，public 模式留空
  ```

## 查看模式的元信息

### `pg_catalog` 系统目录

- **`pg_catalog` 系统目录（模式）** 提供了一个 **`pg_namespace` 模式目录表**，包含了**当前数据库中创建的所有 Schema 模式信息**。

#### `pg_namespace` 模式目录表

> Schema 模式本质上是一个命名空间，所以 PostgreSQL 内部 **`namespace` （命名空间） = Schema 模式**

PostgreSQL 中一个非常重要的**系统目录表**，它属于 `pg_catalog` 模式。它的作用很简单但很核心：**存储了数据库中所有模式（Schema）的定义信息**。

- **每 `CREATE SCHEMA` 创建的每一个模式，都会在这里有一条对应的记录**

简单理解：

- 可以把它理解为 PostgreSQL 中每一个数据库的 “模式注册表”

- **逻辑命名：所有的系统表都使用 `pg_catalog` 模式进行分组，所以可以使用 `pg_catalog.pg_namespace` 来引用**
- **权限**：**普通用户默认可以查询该表（只读）**，但**只能看到自己有权限访问的数据库记录**

> ⚠️注意：谨慎通过 `ALTER`、`DROP`、`DELETE` 、`UPDATE` 等命令来修改 `pg_namespace` 表的内容，因为可能会造成系统损坏！

##### 核心字段

| 字段名         | 类型        | 说明                                                         |
| :------------- | :---------- | :----------------------------------------------------------- |
| **`oid`**      | `oid`       | 对象标识符，是此模式在系统中的**唯一ID**，常用于与其他系统表关联 |
| **`nspname`**  | `name`      | 模式的**名称**，如 `public`、`pg_catalog`                    |
| **`nspowner`** | `oid`       | 模式的**所有者**的角色ID，可以通过关联 `pg_authid` 表查到具体用户名 |
| **`nspacl`**   | `aclitem[]` | 模式的**访问权限列表**，存储了该模式上的权限设置             |

##### 常见 SQL 查询操作

- **查看当前数据库下的所有 Schema 模式信息**

  ```postgresql
  SELECT * FROM pg_namespace ORDER BY oid; -- pg_catalog.pn_namespace 等价
  
  -- 输出结果：
  oid 	nspname				nspowner	nspacl
  11		pg_catalog			10			{postgres=UC/postgres,=U/postgres}
  99		pg_toast			10	
  2200	public				6171		{pg_database_owner=UC/pg_database_owner,=U/pg_database_owner}
  14860	information_schema	10			{postgres=UC/postgres,=U/postgres}
  16641	storages			10	
  16642	company				16654	
  ```

- **查看模式的所有者（关联 `pg_authid` 系统用户角色表）**

  ```postgresql
  SELECT ns.nspname AS schema_name, 
  			 o.rolname AS owner_name
  FROM pg_namespace ns 
  JOIN pg_authid o ON ns.nspowner = o.oid 
  ORDER BY ns.nspname;
  
  -- 输出结果：
  schema_name 		owner_name
  company				hkj
  information_schema	postgres
  pg_catalog			postgres
  pg_toast			postgres
  public				pg_database_owner
  storages			postgres
  ```

- **查看某个表属于哪个 Schema 模式（配合 `pg_class` 或 `pg_tables` 查出对应的数据记录）**

  ```postgresql
  SELECT * FROM pg_class c
  JOIN pg_namespace n ON c.relnamespace = n.oid
  WHERE c.relname = 'file'
  ```

- ...

### `information_schema.schemata` 视图查询

- **`information_schema.schemata` 视图：存储的是数据库中所有 Schema 模式的 SQL 查询视图结果**

```postgresql
-- 通过标准化视图查询模式（底层也是查 pg_namespace）
SELECT schema_name 
FROM information_schema.schemata;
```

### psql 命令行工具

#### `\dn` 查看模式信息

```shell
test=# \dn

      List of schemas
  Name  |       Owner
--------+-------------------
 public | pg_database_owner
(1 row)
```

