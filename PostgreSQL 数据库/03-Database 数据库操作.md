# Database 数据库操作

## 默认数据库（`postgres`）

PostgreSQL 默认提供了 3 个数据库：

### `postgres` 管理数据库

- 定位：这是 PostgreSQL 的**默认管理数据库**。

- 作用：当**初始化连接 PostgreSQL 服务但并没有指定具体连接哪一个数据库**时，**客户端（`psql`）通常会默认尝试连接到 `postgres` 数据库**。

- 使用场景：它主要供 **数据库管理员用户（如 `postgres`）、第三方管理工具或自动化脚本连接使用，以便执行创建其他库、创建用户等管理操作**。**不应该**在这个数据库中**创建与业务项目相关的生产数据表**。

### `template1` 模板数据库

- 定位：这是 PostgreSQL 的**静态模板数据库**。
- 作用：**每当使用 `CREATE DATABASE xxx;` 命令创建一个新数据库时，PostgreSQL 实际上是在后台根据 `template1` 数据库的结构，复制（克隆）了一份新数据库给用户使用**。
- 使用场景：如果有一些全局扩展（`Extenstions`）、自定义函数或全局配置，**希望以后每一个新建的数据库都默认自带这些东西时，可以将它们安装/配置到 `template1` 数据库中**

### `template0` 备份模板数据库

- 定位：这是 PostgreSQL 的**系统核心备份模板**。是 **`template1` 模板数据库最开始的状态**，类似于一个 **“出厂设置”**。
- 作用：PostgreSQL 强烈建议（且在很多情况下强制）**不要对 `template0` 进行任何修改**。
- 使用场景：当**想要把 `template1` 模板恢复到最初的状态**，或者系统需要一个绝对干净的初始环境时，PostgreSQL 内部会**使用 `template0` 作为基准进行初始化**。

> 💡注：如果在使用 `psql -U  xxx -d <数据库>` 命令连接数据库时，指定了 `-d my_app_development` ，PostgreSQL 会自动默认创建一个名为 `my_app_development` 业务数据库方便直接连接，而无需手动登录去执行 `CREATE  DATABASE` 手动创建。
>
> > 同理，Docker 容器连接时，在环境变量中指定了 `POSTGRES_DB=my_app_development`，也会做相同的操作。

## 数据库结构的定义（DDL）

PostgreSQL 对数据库的操作主要围绕**创建 (CREATE)**、**修改 (ALTER)** 和**删除 (DROP)** 这三个核心命令展开。可以通过 SQL 命令或更方便的命令行工具（如 `createdb`、`dropdb`）来执行这些操作。

### `CREATE` 创建数据库

> 注：需要先连接到任意一个数据库（比如默认的 `postgres` ）执行 SQL 语句才会生效。

#### 基本写法

分为 2 种写法：

- **普通创建一个数据库**：

```postgresql
CREATE DATABASE <数据库名>;
```

也可以**在 `psql` 命令行工具中使用 `createdb <数据库名>` 来实现**，无需 SQL。

- **创建一个数据库，指定所有者、编码、静态模板**：

```postgresql
CREATE DATABASE <数据库名> OWNER <所有者> ENCODING 'UTF-8' TEMPLATE <template0 | template1>;
```

- **完整创建，借助 `WITH` 关键字**

```postgresql
-- 完整创建（指定编码和模板，推荐的生产环境标准写法）
CREATE DATABASE hr_db 
    WITH 
    OWNER = hr_manager
    ENCODING = 'UTF8'
    LC_COLLATE = 'zh_CN.UTF-8'  -- 指定排序规则（此处以中文为例）
    LC_CTYPE = 'zh_CN.UTF-8'    -- 指定字符分类
    TEMPLATE = template1       -- 以 template1 为模板克隆
    CONNECTION LIMIT = 100;     -- 限制最大并发连接数为 100
```

#### `WITH ENCODING` 指定字符集

##### PostgreSQL 字符集的管理级别

PostgreSQL 的字符集设置遵循**自上而下**的继承逻辑，主要分为三个级别：

- **集群级别 (Cluster)：** 在使用 `initdb` 初始化数据库集群时指定。这是最底层的默认编码
- **数据库级别 (Database)：** 在创建数据库时指定。**这是 PostgreSQL 中管理字符集最常用的级别**。**同一个数据库下的所有表和列都强制使用该数据库的字符集**
- **客户端级别 (Client)：** 客户端连接数据库时使用的编码，PostgreSQL 会自动在客户端编码和数据库编码之间进行转换

PostgreSQL 的设计理念是：**一个数据库内部的所有文本数据都应该使用同一种底层的二进制编码（如 UTF8）存储**，这样可以保证性能和数据一致性。如果需要处理多语言，推荐将整个数据库设置为 `UTF8`。

> *MySQL 中可以使用 `CHARACTER SET` 写法将字符集精确到 表列。*

##### 基本语法

```postgresql
CREATE DATABASE <数据库名> WITH ENCODING 'UTF8';

-- 在该数据库下的所有 Schema 模式、Table 表、View 视图... 都会统一使用 UTF8 编码
```

##### 查看字符集

- **查看所有数据库的字符集编码**：

  ```postgresql
  SELECT datname, pg_encoding_to_char(encoding) FROM pg_database;
  
  /*
  datname		pg_encoding_to_char
  postgres	UTF8
  test		UTF8
  template1	UTF8
  template0	UTF8
  */
  ```

  也可以**在 `psql` 命令行工具中使用 `\l` 查看具体细节**。

- **查看当前 Session 会话连接的客户端编码 与 服务器编码**：

  ```postgresql
  SHOW server_encoding; -- 服务器（数据库）编码（UTF8）
  SHOW client_encoding; -- 当前客户端编码（Unicode）
  ```

###### 💡**字符集转换支持**

PostgreSQL 支持绝大多数常见编码（如 UTF8, GBK, LATIN1 等）之间的**自动转换**。

但前提是，**服务器编码必须能够兼容客户端写入的数据**。如果把数据库设为 `LATIN1`，却尝试写入中文 UTF8 数据，PostgreSQL 会无情地抛出 `invalid byte sequence` 错误。

- **数据库迁移做法**：

  > 1. 使用 `pg_dump` 导出数据
  > 2. 创建一个编码正确的新数据库
  > 3. 将数据导入新数据库

### `ALTER` 修改数据库

当数据库已创建，需要调整其配置时，可以使用 `ALTER DATABASE` 命令。

如下：

- **重命名数据库（不能重命名当前连接的数据库）**

  ```postgresql
  ALTER DATABASE <数据库> RENAME TO <新数据库名>;
  ```

- **更改数据库所有者**

  ```postgresql
  ALTER DATABASE <数据库> OWNER TO <新所有者>
  ```

- **限制数据库的最大并发连接数**

  ```postgresql
  ALTER DATABASE <数据库> CONNECTION LIMIT <并发数>;
  ```

- **为连接此数据库的所有 Session 会话，设置 `work_mem` 参数的默认值大小**

  ```postgresql
  ALTER DATABASE <数据库> SET work_item = '16MB';
  
  -- 超过 16MB 时，会使用硬盘临时内存进行转存
  ```

- ...

### `DROP` 删除数据库

⚠️注：删除数据库是一个不可逆的操作，会移除库内所有数据。

如下：

- **删除一个数据库（需切换到其他数据库，并且不能有连接会话）**

  ```postgresql
  DROP DATABASE <数据库名>;
  ```

  也可以**在 `psql` 命令行工具中使用 `dropdb <数据库名>` 来实现**，无需 SQL。

- **强制删除数据库（可以强制终止所有连接到该库的会话，然后删除数据库）**

  ```postgresql
  DROP DATABASE <数据库名> WITH (FORCE)
  ```

### `SELECT` 查询操作

- **使用 `current_database()` 函数，查看当前连接的数据库**

  ```postgresql
  SELECT current_database();
  
  -- test
  ```

## 查看数据库的元信息

### `pg_catalog` 系统目录

- **`pg_catalog` 系统目录（模式）** 提供了一个 **`pg_database` 系统数据库目录表，包含了 PostgreSQL 所有数据库信息。**

#### `pg_database` 系统数据库目录表

PostgreSQL 中**通过一系列系统表来管理自身结构**，而 `pg_database` 是 PostgreSQL 系统目录中的一个**系统表**，它存储了关于当前数据库集群中**所有数据库**的**元数据信息**。 

简单理解：

- 可以把它理解为 PostgreSQL 自己的“数据库注册表”

- **存储位置**：它**存储在 `pg_global` 表空间**中，因此**所有数据库共享同一个 `pg_database` 表**，无论连接到哪个数据库，都可以查询它
- **逻辑命名：所有的系统表都使用 `pg_catalog` 模式进行分组，所以可以使用 `pg_catalog.pg_database` 来引用**
- **权限**：**普通用户默认可以查询该表（只读）**，但**只能看到自己有权限访问的数据库记录**

> ⚠️注意：谨慎通过 `ALTER`、`DROP`、`DELETE` 、`UPDATE` 等命令来修改 `pg_database` 表的内容，因为可能会造成系统损坏！

##### 核心字段

| 字段名          | 类型      | 含义                                                   |
| :-------------- | :-------- | :----------------------------------------------------- |
| `oid`           | OID       | 数据库的对象标识符，是全局唯一ID                       |
| `datname`       | name      | 数据库名称                                             |
| `datdba`        | OID       | 数据库所有者（对应 `pg_authid.oid`）                   |
| `encoding`      | int4      | 字符编码的数字编码（如 6 代表 UTF8）                   |
| `datcollate`    | name      | LC_COLLATE（字符串排序规则）                           |
| `datctype`      | name      | LC_CTYPE（字符分类规则）                               |
| `datistemplate` | bool      | 是否为模板数据库（`t` 表示可作为模板）                 |
| `datallowconn`  | bool      | 是否允许连接（`t` 允许，`f` 禁止）                     |
| `datconnlimit`  | int4      | 最大并发连接数限制（-1 表示无限制）                    |
| `datlastsysoid` | OID       | 系统OID阈值，用于区分系统/用户对象                     |
| `dattablespace` | OID       | 默认表空间（对应 `pg_tablespace.oid`）                 |
| `datconfig`     | text[]    | 数据库级别的配置参数（通过 `ALTER DATABASE SET` 设置） |
| `datacl`        | aclitem[] | 数据库的访问权限列表（ACL）                            |

##### 常见 SQL 查询操作

- **查看所有数据库的基本信息**

  ```postgresql
  SELECT oid, datname, datdba, encoding FROM pg_database;
  ```

- **查看当前连接的数据库信息**

  ```postgresql
  SELECT * FROM pg_database WHERE datname = current_database();
  ```

- **查看编码为 UTF-8  的数据库**

  ```postgresql
  SELECT datname FROM pg_database WHERE encoding = 6;
  ```

- **查看允许连接的数据库**

  ```postgresql
  SELECT datname FROM pg_database WHERE datallowconn = true;
  ```

- **查看数据库所有者（需 `JOIN ON` 内关联 `pg_authid` 系统用户表）**

  ```postgresql
  SELECT d.datname AS database_name, u.rolname AS owner 
  FROM pg_database d 
  JOIN pg_authid u ON d.datdba = u.oid;
  ```

- ...

### psql 命令行工具

#### `\c` 切换数据库

> **❌️注意点：PostgreSQL 不支持 `USE` 关键字在同一个会话中“原地”切换数据库上下文，必须重新建立 Session 会话连接。**

切换数据库通常有两种方式：

- **在 SQL 交互界面（psql）中重连**：使用 `\c` 或 `\connect` 元命令，后面跟上数据库名（可选用户名和主机）

  ```bash
  $\c mydb
  -- 或者指定用户名
  $\c mydb myuser
  ```

  > 执行后，psql 会断开当前连接并重新连接到新数据库，同时显示连接成功的提示。

- **在操作系统命令行直接指定**：启动 `psql` 时通过 `-d` 参数指定目标数据库

  ```bash
  $psql -d mydb -U myuser
  ```

#### `\l` | `\list` 查看数据库列表

- **查看所有数据库列表**：在 psql 中使用 `\l` 或 `\list` 元命令，会列出数据库名称、所有者、编码、访问权限等信息。

  ```bash
  test=# \l
                                                  List of databases
     Name    |  Owner   | Encoding | Locale Provider | Collate | Ctype | Locale | ICU Rules |   Access privileges
  -----------+----------+----------+-----------------+---------+-------+--------+-----------+-----------------------
   postgres  | postgres | UTF8     | libc            | C       | C     |        |           |
   template0 | postgres | UTF8     | libc            | C       | C     |        |           | =c/postgres          +
             |          |          |                 |         |       |        |           | postgres=CTc/postgres
   template1 | postgres | UTF8     | libc            | C       | C     |        |           | =c/postgres          +
             |          |          |                 |         |       |        |           | postgres=CTc/postgres
   test      | postgres | UTF8     | libc            | C       | C     |        |           |
  (4 rows)
  
  test=# \l+
                                                                                    List of databases
     Name    |  Owner   | Encoding | Locale Provider | Collate | Ctype | Locale | ICU Rules |   Access privileges   |  Size   | Tablespace |                Description
  -----------+----------+----------+-----------------+---------+-------+--------+-----------+-----------------------+---------+------------+--------------------------------------------
   postgres  | postgres | UTF8     | libc            | C       | C     |        |           |                       | 7510 kB | pg_default | default administrative connection database
   template0 | postgres | UTF8     | libc            | C       | C     |        |           | =c/postgres          +| 7510 kB | pg_default | unmodifiable empty database
             |          |          |                 |         |       |        |           | postgres=CTc/postgres |         |            |
   template1 | postgres | UTF8     | libc            | C       | C     |        |           | =c/postgres          +| 7582 kB | pg_default | default template for new databases
             |          |          |                 |         |       |        |           | postgres=CTc/postgres |         |            |
   test      | postgres | UTF8     | libc            | C       | C     |        |           |                       | 7582 kB | pg_default |
  (4 rows)
  ```

  如果希望以更详细的表格形式查看，可以加上 `+` 后缀：`\l+`，还会显示大小、表空间等额外信息。