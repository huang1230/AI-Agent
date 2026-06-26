# `TRIGGER` 触发器

## 基本概念

​	**`TRIGGER` 触发器**是一种**特殊的存储过程**，它**不能被直接调用**，而是**当**数据库的**指定表发生特定的事件（`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`）时**，由**数据库自动执行 `TRIGGER` 触发器及触发器函数内部的逻辑**，从而实现**自动化脚本处理**。

- 触发器通常用于**自动维护数据完整性、审计变更日志、同步数据**或**执行复杂的业务逻辑**。

### 核心组成部分

要想使用 `TRIGGER` 触发器，需要以下 2 步：

- 创建一个**触发器函数**：定义**当指定表触发特定的事件（`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`）后要自动执行的业务逻辑**，必须要**返回一个特定的类型 `TRIGGER`**
- 创建一个 **`TRIGGER` 触发器：绑定到具体的表或事件上**

## `TRIGGER` 触发器

### 基本语法结构

- 创建一个 **`TRIGGER` 触发器：绑定到具体的表或事件上，并设定对应的「触发时机、触发颗粒度、要自动执行的触发器函数」**。

```postgresql
CREATE TRIGGER <触发器名>
[BEFORE | ALTER | INSTEAD OF] [UPDATE | INSERT | DELETE | TRUNCATE] ON <模式名>.<表名 | 视图名> 
-- 触发时机（触发器函数的执行时机），并指定要监听的表更新操作
[FOR EACH ROW | FOR EACH STATEMENT] -- 触发颗粒度
EXECUTE FUNCTION <触发器函数>();
```

​	核心作用：**当指定表 `<表名>` 触发特定的事件（`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`）后，会自动执行 `EXECUTE FUNCTION` 指向的函数**，并**根据**所设定的**「触发时机」、「触发颗粒度」**来**决定「触发器函数」中注入**到 **`OLD`、`NEW` 内置变量的数据（旧行、新行）**。

### 触发器函数的执行时机（触发时机）

核心作用：**决定**了**在指定表发生特定事件（`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`）后**，**`EXECUTE FUNCTION` 指向的「触发器函数」所执行的时机**

#### **`BEFORE`** 表修改之前执行

核心描述：**「触发器函数」**会在**实际修改**表数据**之前**执行，并**影响「触发器函数」中的 `OLD | NEW` 内置变量的值，最后可根据 `RETURN NEW | OLD | NULL` 进行新行二次修改、原封不动、取消更新等处理，最后再实际注入到数据库表中**。

常用于数据验证、自动补全字段（如自动填入修改时间）。

- **触发器函数返回 `RETURN NEW` 新行**

> [!IMPORTANT]
>
> 在**数据真正写入磁盘之前**，**`BEFORE` 触发器**可以**拦截并修改数据**。
>
> 此时**「触发器函数」`RETURN NEW` 返回新行记录**的意义如下：
>
> - **作用：** 允许触发器**批准并应用**当前的修改，或者采用**触发器函数内部对 `NEW` 字段**做出的**二次修改**。
>
> - 示例：（自动补全/数据清洗）
>
>   ```postgresql
>   -- 比如在写入前，强行把邮箱转为小写
>   NEW.email := LOWER(NEW.email);
>   RETURN NEW; -- 数据库会把修改后的 NEW 写入表中
>   ```

- **触发器函数返回 `OLD` 旧行**

> [!IMPORTANT]
>
> 在**数据真正写入磁盘之前**，**`BEFORE` 触发器**可以**拦截并修改数据**。
>
> 此时**「触发器函数」`RETURN OLD` 返回旧行记录**的意义如下：
>
> - **作用：** 通常在 `UPDATE` 触发器中。如果**返回 `OLD`，意味着放弃当前用户提交的 `NEW`（新修改），而把原本的 `OLD`（旧数据）重新写回去**
> - **效果：** 相当于本次修改虽然执行了，但数值被原封不动地还原了

- **触发器函数返回 `NULL` 空值**

> [!IMPORTANT]
>
> 在**数据真正写入磁盘之前**，**`BEFORE` 触发器**可以**拦截并修改数据**。
>
> 此时**「触发器函数」`RETURN NULL` 返回空值**的意义如下：
>
> - **作用：** **彻底静默取消（拦截）本次操作**。
>
> - **效果：** 数据库会直接跳过这行数据的 `INSERT`、`UPDATE` 或 `DELETE`，且不会报错。
>
> - **典型场景（条件拦截）：**
>
>   ```postgresql
>   -- 如果用户试图删除管理员，直接静默拦截，不予执行
>   IF OLD.username = 'admin' THEN
>       RETURN NULL; -- 本行删除操作被取消
>   END IF;
>   RETURN OLD;
>   ```

> **一句话记忆：**在 `BEFORE` 触发器里，**触发器函数的 `RETURN` 返回值** 决定了**最终存进数据库里的是什么**。

#### `ALTER` 表修改之后执行

核心作用：**「触发器函数」**会在**实际修改**表数据**之后**执行，并**影响「触发器函数」中的 `RETURN OLD | NEW` 内置变量的值**。

常用于级联更新其他表、记录日志、发送通知。

> [!IMPORTANT]
>
> 在 `AFTER` 触发器执行时，数据**已经实际写入**了数据库，木已成舟。
>
> - **作用：** 此时返回值**不再影响**任何写入结果。
> - **规则：** **对于行级（`FOR EACH ROW`）的 `AFTER` 触发器，PostgreSQL 规范要求**必须返回一个**非空值**（通常习惯 `INSERT`/`UPDATE` 返回 `NEW`，`DELETE` 返回 `OLD`）。如果返回了 `NULL`，随后的其他行级 `AFTER` 触发器将被跳过不再执行。
>   - 简而言之：在 `AFTER` 触发器里，无脑写 `RETURN NEW;` 或 `RETURN OLD;` 即可，它只是为了让语法合规。

#### 总结

| **触发器时机/触发器函数返回值** | **返回 NEW**                               | **返回 OLD**                         | **返回 NULL**                             |
| ------------------------------- | ------------------------------------------ | ------------------------------------ | ----------------------------------------- |
| **`BEFORE`**                    | **允许写入新数据**（包含被触发器改过的值） | **放弃新数据，强行把旧数据重新写入** | **取消/拦截** 本行操作，不写入            |
| **`AFTER`**                     | 正常结束（习惯用法）                       | 正常结束（习惯用法）                 | **不建议**，会使得后续的 AFTER 触发器失效 |

#### **`INSTEAD OF`** 视图更新

核心作用：通常用于**视图（View）**，**替代原有的 DML 操作**，因为**视图本身默认不可直接修改**。

### 触发颗粒度

- **`FOR EACH ROW`（行记触发器）：每受影响一行，触发器就执行一次**。如果一条 `UPDATE` 语句更新了 100 行，它会执行 100 次。
- **`FOR EACH STATEMENT`（语句级触发器，默认）**：无论影响了多少行，整个 SQL 语句执行期间只触发**一次**。

| **特性**                 | **FOR EACH ROW (行级)**            | **FOR EACH STATEMENT (语句级)**   |
| ------------------------ | ---------------------------------- | --------------------------------- |
| **触发频率**             | **每行**触发一次（按影响行数累计） | **每条SQL**触发一次（与行数无关） |
| **可否使用 `NEW`/`OLD`** | **可以**（代表当前行的新旧数据）   | **不可以**（值为 `NULL`）         |
| **性能影响**             | 批量更新大表时，高频调用，开销较大 | 开销极低，只跑一次                |
| **常见用途**             | 数据清洗、单行审计、字段拦截修改   | 权限控制、表级日志、缓存刷新      |

⚠️ **避坑指南**

1. **性能开销：** **行级触发器（`FOR EACH ROW`）**在处理**大批量数据导入或更新**时，会对**性能**产生**明显影响**。
2. **死循环：** 避免在 `A 表` 的触发器中去修改 `B 表`，而 `B 表` 的触发器又反过来修改 `A 表`，这会导致嵌套死循环。

### 基本操作

#### psql 查看表上的触发器

```sh
$\d 表名

test=# \d trigger_test.login_log;
                      Table "trigger_test.login_log"
   Column   |            Type             | Collation | Nullable | Default
------------+-----------------------------+-----------+----------+---------
 log_id     | integer                     |           | not null |
 log_time   | timestamp without time zone |           |          | now()
 user_id    | integer                     |           |          |
 user_agent | text                        |           |          |
 login_way  | text                        |           |          |
Indexes:
    "login_log_pkey" PRIMARY KEY, btree (log_id)
Check constraints:
    "login_log_login_way_check" CHECK (login_way = ANY (ARRAY['手机号'::text, '邮箱'::text, '微信快捷登录'::text]))
Foreign-key constraints:
    "login_log_user_id_fkey" FOREIGN KEY (user_id) REFERENCES trigger_test.users(user_id)
Triggers:
    user_login_trigger BEFORE INSERT ON trigger_test.login_log FOR EACH ROW EXECUTE FUNCTION trigger_test.user_login_script()
```

#### 禁用/启用表上的触发器

```postgresql
ALTER TABLE <模式名>.<表名> DISABLE TRIGGER <触发器名>; -- 禁用此表的某个触发器

ALTER TABLE <模式名>.<表名> ENABLE TRIGGER <触发器名>; -- 启用此表的某个触发器
```

#### `DROP TRIGGER` 删除表上的触发器

```postgresql
DROP TRIGGER <触发器名> ON <模式名>.<表名>; -- 删除此表上的某个触发器
```

## `FUNCTOIN` 触发器函数

### 基本语法结构

- 定义**当指定表触发特定的事件（`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`）后要自动执行的业务逻辑**，必须要**返回一个特定的类型 `TRIGGER`**。

````postgresql
CREATE [ON REPLACE] FUNCTION <触发器函数名>(<形参1> <数据类型>, <形参2> <数据类型>,...) -- 1. 要接收的形参列表（可选）
RETURNS TRIGGER -- 2. 返回值必须是一个 TRIGGER 触发器类型
LANGUAGE plpgsql
AS $$ -- 连接关键字
DECLARE 
	-- 3. 临时局部变量声明区(可选)
	<变量> <数据类型> [:= | DEFAULT] <默认值>;
BEGIN -- 4. 开始标记
	
	-- 5. 自动执行的 SQL 业务逻辑，在这此处使用 OLD 旧行 | NEW 新行 操作更新、删除的行数据，最后再反馈到数据库中
	-- ===
	-- [OLD | NEW].<列名> -- 
	-- ===
	RETURN [OLD | NEW | NULL]; -- 6. BEFORE 触发器中，决定了实际存储到数据库中的行记录；AFTER 触发器中，无效
END $$; -- 7. 结束标记
````

#### 特殊变量

在 PL/pgSQL 触发器函数中，默认提供了如下特殊的**内置变量**。

作用：可以在**当指定表触发特定的事件**（`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`）**后**，**获取对应的旧行、新行记录、事件操作标识...**等信息。

- **`NEW` 内置变量**：
  - **类型： `RECORD`（行记录）**
  - 表示：当**指定表发生 `INSERT` 插入、`UPDATE` 更新 事件操作**时，**存储**所**产生**的**新行记录**
    - **发生 `DELETE` 删除操作**时，**`NEW` 变量的值为 `NULL`**
- **`OLD` 内置变量**：
  - **类型：`RECORD`（记录）**
  - 表示：当**指定表发生 `UPDATE` 更新、`DELETE` 删除 事件操作**时，**存储**所**被覆盖、删除**的**旧行记录**
    - **发生 `INSERT` 插入操作**时，**`OLD` 变量的值为 `NULL`**
- **`TG_UP`：**
  - **类型：字符串**
  - 表示：**`TRIGGER` 触发器所绑定**的**指定表所发生**的**当前事件操作**
  - 取值：**`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`**

<img src="media/TRIGGER 触发器内置变量概念.png" style="zoom:50%;" />

#### 执行机制与逻辑

当**指定表触发特定的事件**（`INSERT`、`UPDATE`、`DELETE` 或 `TRUNCATE`）时：

- **若为 `BEFORE` 触发器**：

  触发器会**先拦截要准备写入到数据库中的行记录**，并**携带旧行、新行记录注入**到**「触发器函数」中的 `OLD`、`NEW` 内置变量**中。

  此时，**「触发器函数」的 `RETURN` 返回值行为**的**意义**如下：

  - **`RETURN OLD`**：**放弃当前用户提交的 `NEW`（新修改），而把原本的 `OLD`（旧数据）重新写回去**
  - **`RETURN NEW`**：**批准并应用**当前表的**修改操作**，如果**「触发器函数」中对 `NEW` 新行**做了**二次修改**，则会**一并写入到数据库中**
  - **`RETURN NULL`**：直接**取消**当前表的**本次修改数据操作**

- **若为 `AFTER` 触发器**：

  此时，数据**已经实际写入**到数据库中了，**「触发器函数」**的 `RETURN` 返回值**不再影响**任何写入结果。

<img src="media/TRIGGER 触发器执行机制.png" style="zoom:60%;" />

## 示例

假设有如下表：

```postgresql
CREATE SCHEMA trigger_test;
SET search_path TO trigger_test;
CREATE TABLE users
(
	user_id INTEGER PRIMARY KEY, -- 用户ID
	user_name VARCHAR(4) NOT NULL, -- 用户姓名
	age INTEGER NOT NULL, -- 年龄
	sex VARCHAR(2) CHECK (sex IN ('男', '女')) -- 性别
);

CREATE TABLE login_log
(
	log_id INTEGER PRIMARY KEY,	 -- 登录ID
	log_time TIMESTAMP DEFAULT NOW(), -- 登录的时间，默认为当前时间
	user_id INTEGER REFERENCES users(user_id) -- 登录的用户ID
);

SELECT * FROM users;
SELECT * FROM login_log;
```

创建触发器并绑定到表，当 `employees` 员工表发生更新操作（`INSERT`、`UPDATE`、`DELETE`...）时，会自动执行触发器函数的逻辑：

- **`BEFORE` 触发器：拦截表的当前更新行记录**，并**决定实际写入到数据库中的更新结果**
- **`ALTER` 触发器**：只是做一些**不关数据库实际写入结果的日志处理**...

```postgresql
-- 创建一个 BEFORE 触发器，绑定到 login_log(登录日志表) 中。
-- 作用：当用户登录（INSERT login_log 添加数据事件）时，自动执行修改登录日志，修改用户登录状态... 等二次修改操作之后写入到数据库中的脚本。
CREATE TRIGGER user_login_trigger
BEFORE INSERT ON login_log -- 当 login_log 表 INSERT 添加新数据时，自动执行 user_login_script() 触发器函数
FOR EACH ROW -- 每次登录都会执行一次触发器函数（以 行 为单位）
EXECUTE FUNCTION user_login_script(); -- 自动执行的 触发器函数


-- 删除触发器函数
DROP FUNCTION user_login_script;
-- 删除触发器
DROP TRIGGER user_login_trigger ON login_log;

-- 开启 login_log 表上的触发器
ALTER TABLE login_log ENABLE TRIGGER user_login_trigger;

-- 关闭 login_log 表上的触发器
ALTER TABLE login_log DISABLE TRIGGER user_login_trigger;
```

触发器函数逻辑：

```postgresql
-- 创建一个触发器函数，当 login_log 登录日志表发生 INSERT、UPDATE、DELET  操作时会自动执行
CREATE OR REPLACE FUNCTION user_login_script()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
	-- NEW 内置变量: 当前 INSERT 插入的新行记录，对其做二次修改、对其他表的级联操作...
		
	-- 1. 更新当前用户的登录状态（已登录成功）
	UPDATE users SET login_status = TRUE WHERE user_id = NEW.user_id;
	
	-- 2. 添加用户登录方式、登录设备
	NEW.user_agent = 'Apple';
	NEW.login_way = '手机号';
	
	RETURN NEW; -- 把二次修改后的新行（当前插入的登录日志数据），实际写入到数据库中
	-- RETURN OLD; -- 不做二次修改，以 INSERT 新插入的记录为准，写入到数据库中
	-- RETURN NULL; -- 直接取消本次表更新的 INSERT 操作，不写入数据库中
	
END $$;

-- 更新事件发生（张三 和 李四 用户登录）
INSERT INTO login_log (log_id, user_id) VALUES (1, 1), (2, 2); -- 这里并没有传入 user_agent、login_way 字段的值

-- 验证结果
SELECT * FROM users;
/*
 user_id | user_name | age | sex | login_status
---------+-----------+-----+-----+--------------
       3 | 王五      |  22 | 男  | 				
       1 | 张三      |  25 | 男  | 		t	  <-- user_login_script 触发器函数自动修改的列 
       2 | 李四      |  30 | 女  | 		t
*/

SELECT * FROM login_log;
/*
 log_id |         log_time         | user_id | user_agent | login_way
--------+--------------------------+---------+------------+-----------
      1 | 2026-06-24 02:04:51.4676 |       1 | Apple      | 手机号    <-- user_login_script 触发器函数自动修改的列 
      2 | 2026-06-24 02:04:51.4676 |       2 | Apple      | 手机号    <-- user_login_script 触发器函数自动修改的列 
*/
```









