# `JSON` & `JSONB` 文档型数据处理

## 基本概念

​	在 PostgreSQL 中，默认支持存储与处理 JSON 格式的半结构化数据类型，它是 PostgreSQL 最强大的核心特性之一，也是被誉为 “世界上最先进的开源关系型数据库” 的核心原因之一。

​	PostgreSQL 虽然是一款经典的**关系型数据库**（RDBMS），但它对**文档型数据（NoSQL / JSON）**的支持甚至可以媲美许多专门的文档数据库（如 MongoDB）。

​	PostgreSQL 处理文档型数据之所以强大，核心在于它能将**结构化 SQL** 与**半结构化 JSON** 完美结合，使得在**同一张表**里**既能享受 ACID 事务，又能体验 Schema-less（高性能无模式数据）**的灵活性。

### 与 MongoDB 数据库的区别

PostgreSQL 默认支持 JSON 格式的数据，这意味着它也能在部分场景下代替 MongoDB 来存储文档型数据，但也不是绝对的。

前言：虽然近年来 PostgreSQL 不断增强对 JSON 数据的支持（特别是通过JSONB类型），让两者在某些场景下有了重叠，但它们的设计哲学、核心优势和适用场景依然很不一样。

#### 核心差异

| 维度         | **PostgreSQL**                                               | **MongoDB**                                                  |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **核心定位** | 关系型数据库（RDBMS）的集大成者，强调**数据完整性、一致性和标准SQL能力**，适合处理复杂关系数据。 | 非关系型数据库（NoSQL）的领导者，核心是**文档模型**，为应对海量数据、高并发写入和快速迭代而设计。 |
| **数据模型** | **严格的表结构（Schema）**。数据以行和列的形式存储，模式需要预先定义，修改起来成本较高。 | **灵活的文档模型**。数据以类似JSON的BSON格式存储，同一个集合（Collection）里的文档可以拥有完全不同的字段，非常灵活。 |
| **查询语言** | **SQL**。强大的、标准化的查询语言，支持复杂的多表关联（JOIN）、窗口函数、公共表表达式（CTE）等高级分析功能。 | **MongoDB查询语言（MQL）**。基于JSON风格的API，对于嵌套数据的查询非常直观，但在处理复杂关联查询时不如SQL强大。 |
| **扩展方式** | 主要**垂直扩展**（升级硬件）。支持通过Citus等扩展实现水平扩展，但并非原生内置。 | **原生水平扩展（分片）**。通过分片（Sharding）技术，可以轻松地将数据分布到成百上千台服务器上。 |

##### 高并发下的性能差异

PostgreSQL 虽然也能存储文档型数据，在处理**频繁、高效的文档更新**时，两者差异巨大。

- **在 PostgreSQL 中修改 JSONB 字段**，由于**MVCC（多版本并发控制）**机制，**每次修改**都会**在磁盘上**产生一个**新的行记录版本**。如果 **JSONB 文档较大或更新频繁**，这会导致**大量的 I/O 和 CPU 开销**，性能可能随时间推移而下降
- **在 MongoDB 中修改 BSON 文档**，由于其存储引擎和文档模型是原生设计的，可以精准地定位和修改文档中的某个字段，无需重写整个文档，因此在高频更新的场景下性能**更稳定、更高效**

一份来自 MongoDB 官方博客的对比测试也印证了这一点：在同样**高并发**的更新负载下，**MongoDB 能保持平稳的吞吐量**，而**PostgreSQL** 的 **吞吐量**则**随时间推移**出现**显著下滑**，同时伴随着**更高的CPU和延迟**。

简单来说，对于小型应用，使用 PostgreSQL 足够；而对于大型高并发应用，MongoDB 的性能更强。

#### 场景选型

##### **更推荐 PostgreSQL 的场景**

- **需要高度数据一致性**：例如**金融交易系统、企业级ERP、订单管理**，要求**严格的 ACID事务 和引用完整性（外键约束）**
- **数据结构稳定，关系复杂**：数据模型清晰，且需要进行复杂的多表关联查询、报表分析
- **希望利用强大的SQL生态**：团队对SQL非常熟悉，需要利用其强大的分析能力和丰富的工具链
- **存储效率优先**：学术研究表明，PostgreSQL在存储相同数据时，其资源消耗和所需存储空间（尤其是使用规范化结构时）显著低于MongoDB（一项研究显示可达6倍差距）

> [!IMPORTANT]
>
> 关键点：**核心数据依然是关系型数据**（如用户表、账户表），但**部分业务**（如商品扩展属性、用户行为日志、配置项）**需要高度灵活的文档结构，且需要强 ACID 事务保障**。

##### **更推荐 MongoDB 的场景**

- **快速迭代，模型多变**：项目处于早期，数据结构频繁调整，无法接受数据库迁移带来的停机或复杂操作
- **海量数据和高并发写入**：需要处理物联网、用户行为日志等海量、高速写入的数据，且未来数据量会爆发式增长，对水平扩展有硬性要求
- **数据天然是文档结构**：例如内容管理系统（CMS）、产品目录，数据本身就是嵌套的、层次化的，用文档模型存储更自然，可以减少应用层对象-关系映射（ORM）的复杂性
- **写入性能优先于复杂查询**：系统对写入速度要求极高，而查询模式相对简单（例如按ID或简单条件查询）

> [!IMPORTANT]
>
> 关键点：**数据完全没有关系型特征，数据量极度庞大（PB 级别），需要天然的横向分片扩展（Sharding）**，或者 **Schema 模型彻底无规律**。

## 运算符

### 📌JSON / JSONB 运算符

PostgreSQL 对 JSON（尤其是二进制的 `jsonb`）的支持极其强大。

| **运算符** | **描述**                                            | **示例**                    | **意图**                            |
| ---------- | --------------------------------------------------- | --------------------------- | ----------------------------------- |
| `->`       | 通过键/索引获取 JSON 对象/数组元素（返回 **JSON**） | `data -> 'user'`            | 获取 data 列中的 user 属性值        |
| `->>`      | 通过键/索引获取 JSON 对象/数组元素（返回 **文本**） | `data ->> 'name'`           | 获取 `name` 字符串                  |
| `#>`       | 通过路径获取 JSON 对象（返回 JSON）                 | `data #> '{user, address}'` | 获取 user 键的 address 值           |
| `#>>`      | 通过路径获取 JSON 对象（返回 **文本**）             | `data #>> '{user, name}'`   | 获取 user 下的 name 文本            |
| `@>`       | JSONB 包含（左侧是否包含右侧）                      | `info @> '{"status": 1}'`   | 筛选 status 为 1 的行（带索引优化） |
| `?`        | 字符串是否存在于 JSON 键中                          | `info ? 'email'`            | 检查是否存在 email 键               |

### 📌字符拼接运算符

- **`||`**：用于将两个或多个字符串连接成一个字符串。

  ```sql
  SELECT 'Hello ' || 'World'; -- 结果: 'Hello World'
  ```

💡 提示：关于运算符的性能优化

在进行大规模数据查询时，使用 **`@>` (JSONB包含)** 或 **`&&` (数组交集)** 运算符通常可以完美命中 GIN（广义倒排索引）索引，从而实现毫秒级的海量数据检索；而使用 `ILIKE` 或带有前缀通配符的 `LIKE '%abc'` 则可能导致全表扫描，需要配合 `pg_trgm`（三元组扩展）建立索引。

## 核心数据类型 `JSON` & `JSONB`

在 PostgreSQL 中，**`JSON`** 和 **`JSONB`** 是两种用于存储 JSON 数据的强大类型。虽然它们都能存储 JSON，但**底层的存储方式和适用场景有很大区别**。

> 注：绝大多数场景下应该选择 **`JSONB`**。

### 数据定义

- **`JSON` 与 `JSONB` 直接声明**为**表列**的**数据类型**，表示该表中**一半为 SQL 结构化数据**，**另一部分为类似于 Python 键值对字典的半结构化数据**。

```postgresql
-- JSON 格式
CREATE TABLE <表名>
(
	<字段名> JSON
);


-- JSONB 格式
CREATE TABLE <表名>
(
	<字段名> JSONB
);
```

#### 两者的核心差异

- **`JSON`** 与 **`JSONB`** 都表示**存储 JSON（JavaScript 对象表示法） 格式的数据**

  但 **`JSON`** 是**纯文本形式**的**JSON 原始格式**，而 **`JSONB`** 是经过**解析处理之后**的 **JSON 二进制格式**

| 特性         | `JSON` 纯文本                                                | `JSONB` 二进制                                               |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **存储格式** | **原样存储**的**纯文本**                                     | **解析处理**之后的**二进制格式**                             |
|              | **保留：空格、换行符、重复 Key键** <br />**保持 Key键 的插入顺序**进行存储 | **去除空格、换行符、重复 Key键，仅保留最后一个键值对**<br />**对 Key键 内部进行了自动排序** |
|              | **原样保留数字格式**；如 `1e3 => 1e3`                        | **转换为实际的数值**；如 `1e3 => 100`                        |
| **写入速度** | **快（直接写入、无需解析）**                                 | **慢（先解析、后写入）**                                     |
| **查询速度** | **慢**（每次查询都要解析整个 JSON 纯文本）                   | **快**（直接读取二进制，且支持索引）                         |
| **索引支持** | **不支持**                                                   | **完美支持 GIN 索引**                                        |

💡 **总结建议**：绝大多数场景下，**直接闭眼选 `JSONB`**。只有当**需要完整保留原始 JSON 的空格、键顺序，或者几乎全是写入、极少查询时，才考虑 `JSON`**。

### 数据更新

#### `INSERT` 插入数据

##### 基本写法

两者都采用同样的插入写法：

```postgresql
INSERT INTO <模式名>.<表名>(...) VALUES (...,'{"Key":"Value"}',...);

-- 先以纯文本形式插入：
--  JSON  会原样存储
--  JSONB 会对纯文本解析处理之后转为二进制格式存储
```

###### JSON 格式

在 `INSERT` 插入 JSON 数据时，必须**严格按照 JSON 字符串的格式**来传入：

- **外层使用 `''` 单引号包裹，表示一个字符串**

- **内层：**

  - **`{}` 对象： 原样保留**

    - **`Key` 属性键：使用 `""` 双引号包裹**
    - **`Value` 属性值：使用 `""` 双引号包裹**

    ```json
    // 单层属性
    '{"x": "y"}'
    
    // 双层嵌套属性
    '{"x": {"y":"z"}}'
    
    // 多层嵌套属性
    '{"x": { "y": "z": { "a": "b" }}}'
    ```

  - **`[]` 数组**：**原样保留**

    - **`element` 元素：使用 `""` 双引号包裹**

    ```json
    '["x"]' // 单层
    
    '["x", ["y", "z"]]' // 多层嵌套
    
    '["x", {"z": "b"}]' // 数组 + 对象
    ```

##### 差异示例

假设有如下表：
```postgresql
CREATE TABLE json_test
(
	json_info JSON  -- JSON 纯文本格式
);

CREATE TABLE jsonb_test
(
	json_data JSONB -- JSON 二进制格式
);
```

###### Key键排序

- **`JSON` 纯文本格式：保留 `INSERT`、`UPDATE` 插入时的顺序**

  ```postgresql
  -- 插入 JSON 数据
  INSERT INTO json_test(json_info) VALUES ('{"name":"Jack","age":18,"department":"HR"}');
  
  -- 查询 JSON 示例：
  SELECT * FROM json_test;
  /*
  {"name":"Jack","age":18,"department":"HR"}
  */
  ```

- **`JSOB` 二进制格式：内部会对 Key键 进行从小到大的排序**

  ```postgresql
  -- 插入 JSONB 数据
  INSERT INTO jsonb_test(json_data) VALUES ('{"name":"Jack","age":18,"department":"HR"}');
  
  -- 查询 JSONB 示例：
  SELECT * FROM jsonb_test;
  /*
  {"age": 18, "name": "Jack", "department": "HR"}  <-- 对键进行了排序
  */
  ```

###### 重复键

- **`JSON` 纯文本格式：保留 `INSERT`、`UPDATE` 插入时的原样数据**

  ```postgresql
  -- 插入带有重复键的 JSON 数据：
  INSERT INTO json_test(json_info) VALUES ('{"name":"Jack","name":"Jack","age":18,"age":18,"department":"HR"}');
  
  -- 查询 JSON 示例：
  SELECT * FROM json_test;
  /*
  {"name":"Jack","name":"Jack","age":18,"age":18,"department":"HR"} <-- 原样存储	
  */
  ```

- **`JSOB` 二进制格式：去除重复的 Key键，只保留最后一个键值对**

  ```postgresql
  -- 插入带有重复键的 JSONB 数据：
  INSERT INTO jsonb_test(json_data) VALUES ('{"name":"Jack","name":"Jack","age":18,"age":18,"department":"HR"}');
  
  -- 查询 JSONB 示例：
  SELECT * FROM jsonb_test;
  /*
  {"age": 18, "name": "Jack", "department": "HR"}  <-- 去除了重复的键值对
  */
  ```

###### 空格、换行符

> 注意：仅限于**键与值之间的空白符、换行符**，如果**键名、值中有空白符**，则会**原样保留**。

- **`JSON` 纯文本格式：保留 `INSERT`、`UPDATE` 插入时的原样数据**

  ```postgresql
  -- 插入带有空格、换行符的 JSON 数据：
  INSERT INTO json_test(json_info) VALUES ('{"name":  "Jack" ,"age": 18  ,"department":  "HR"}');
  
  -- 查询 JSON 示例：
  SELECT * FROM json_test;
  /*
  {"name":  "Jack" ,"age": 18  ,"department":  "HR"} <-- 原样存储	
  */
  ```

- **`JSOB` 二进制格式：去除空格、换行符**

  ```postgresql
  -- 插入带有空格、换行符的 JSONB 数据：
  INSERT INTO jsonb_test(json_data) VALUES ('{"name":  "Jack" ,"age": 18  ,"department":  "HR"}');
  
  -- 查询 JSONB 示例：
  SELECT * FROM jsonb_test;
  /*
  {"age": 18, "name": "Jack", "department": "HR"}  <-- 去除了属性与值之间的空白符
  */
  ```

###### 数字格式（科学计算法）

- **`JSON` 纯文本格式：保留 `INSERT`、`UPDATE` 插入时的原样数据**

  ```postgresql
  -- 插入带有科学计数的数字格式 的 JSON 数据
  INSERT INTO json_test(json_info) VALUES ('{"size": 10e-2}');
  
  -- 查询 JSON 示例：
  SELECT * FROM json_test;
  /*
  {"size": 10e-2} <-- 原样存储	
  */
  ```

- **`JSOB` 二进制格式**：将**科学计算法的数字格式**转为**实际的数值**

  ```postgresql
  -- 插入带有科学计数的数字格式 的 JSONB 数据
  INSERT INTO jsonb_test(json_data) VALUES ('{"size": 10e-2}');
  
  -- 查询 JSONB 示例：
  SELECT * FROM jsonb_test;
  /*
  {"size": 0.10}  <-- 转为实际的数值
  */
  ```

###### 存储大小

- **行记录的数据大小：`JSONB` > `JSON`**，因为 **`JSONB` 会以二进制格式存储数据，对读取进行了更多优化**，所以**存储时会稍重一些**

  ```postgresql
  -- JSON 存储大小
  SELECT pg_column_size(json_info) AS json_size FROM json_test; -- 43
  
  
  -- JSONB 存储大小
  SELECT pg_column_size(json_data) AS jsonb_size FROM jsonb_test; -- 63 <-- 更大，因为对 JSON 数据做了优化
  ```

#### `UPDATE` 更新数据

##### `{key}` JSON 属性的提取

在 PostgreSQL 中，JSON 属性的提取都是通过以下方式实现：

- **`{key}` 提取一个顶层单属性**
- **`{x,y}`：提取一个双层嵌套子属性**。如 `{user,name}` 表示提取 `user.name` 属性
- **`{x,y,z...}`：提取一个多层嵌套的子属性**。如 `{user,address,city}` 表示 提取 `user.address.city` 属性

> 关键点：使用 **`{}` 表示提取属性**，使用 **`,` 逗号**表示**属性嵌套的层级**，**从左往右逐层往下深入**

<img src="media/JSON 属性的提取.png" style="zoom:70%;" />

##### `::jsonb` 数据类型转换

关键点：**所有 `UPDATE` 更新的 JSON 字符串数据都需要通过 `::jsonb` 转换为 JSONB 二进制格式写入表中**。

##### `jsonb_set()` 函数

PostgreSQL 内置了一个 **`jsonb_set()` 函数**专门用于**更新/修改表中指定列的数据类型为 `JSONB` 二进制 JSON 格式的值**。

###### 语法

```postgresql
-- 顶层属性
<指定 JSONB类型 列> = jsonb_set(<指定 JSONB类型 列>, <'{要更新的 Key 键}'>, <'Value 新值'>::jsonb);


-- 嵌套子属性
<指定 JSONB类型 列> = jsonb_set(<指定 JSONB类型 列>, <'{键1}.{键2}.{要更新的 Key 子键}'>, <'Value 新值'>::jsonb);
```

> 注：**Value 值以字符串形式传入**。**数字采用 `'xx'` 包裹，字符串采用 `'"xxx"'` 包裹**

示例：
```postgresql
-- 更新前
SELECT * FROM jsonb_test;
/*
{"age": 18, "name": "Jack", "department": "HR"}
{"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
*/

-- 更新 jsonb_test 表中 json_data 列中 age 属性的值为 30
UPDATE jsonb_test SET json_data = jsonb_set(json_data, '{age}', '30'::jsonb);
-- 更新 jsonb_test 表中 json_data 列中 department 属性的值为 "研发"
UPDATE jsonb_test SET json_data = jsonb_set(json_data, '{department}', '"研发"'::jsonb);

-- 更新后
SELECT * FROM jsonb_test;
/*
{"age": 30, "name": "Jack", "department": "研发"}
*/
```

```postgresql
-- 提取所有 user_info 中全量包含 {name:Tom} 顶层键值对的行记录中，并将 address.city.city_name 修改为 '北京'
UPDATE json_test.jsonb_test2 
SET user_info = jsonb_set(user_info, '{address,city,city_name}', '"北京"'::jsonb)
WHERE user_info @> '{"name":"Tom"}';

SELECT * FROM json_test.jsonb_test2 WHERE user_info @> '{"name":"Tom"}';
/*
                                       user_info
----------------------------------------------------------------------------------------
{"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "北京"}}}
*/
```

##### `||` 添加与拼接 JSON 字符串

> [!IMPORTANT]
>
> 关键点：在**任意数据库对象中存储**的**JSON 数据**本质上**可以看做**就是**一个「字符串」（无论是表列、JSON 字面量）**，所以可以**使用 `||` 对两者**进行**拼接成一个「字符串」**，实现进一步的 **`SELECT` 查询输出、`INSERT`、`UPDATE` 更新操作**。

###### 追加与部分覆盖

```postgresql
<指定 JSONB类型 列> || '{key:value}'

-- <指定 JSONB类型 列> 会转为字符串形式
```

###### 核心作用

- 通过**使用 `||` 将 <表列> 与 <字符串值> 以 JSON 字符串的形式拼接在一起**，可用于**追加/覆盖一个<表列>中的 JSON 属性** 
  - 若 **Key 键存在即覆盖**，**不存在即追加**
- 最后**使用 `::jsonb` 转化为 JSON 的二进制格式存储**

```postgresql
-- 更新之前
SELECT * FROM jsonb_test;
/*
{"age": 30, "name": "Jack", "department": "研发"}
*/

-- || 字符串拼接，可用于追加/覆盖一个属性
-- 存在即覆盖
UPDATE jsonb_test SET json_data = json_data || '{"department":"客服"}'::jsonb;
-- 不存在即追加
UPDATE jsonb_test SET json_data = json_data || '{"gender":"male"}'::jsonb;

-- 更新之后
SELECT * FROM jsonb_test;
/*
{"age": 30, "name": "Jack", "gender": "male", "department": "客服"}
*/
```

###### 完全覆盖

- `UPDATE .. SET` 本质上就是给一个 表列 更新赋值，所以完全可以直接使用**全新的字符串值**进行**完全覆盖**。

```postgresql
UPDATE <表名> SET <指定JSONB列> = <JSON 字符串> || <JSON字符串>;

-- <JSON 字符串> 
-- 可以是一个 '{key:value}' 对象字面量字符串
-- 也可以是一个使用 -> / ->> 或 #> / #>> 提取表列某个属性的值字符串
```

示例：

```postgresql
INSERT INTO json_test.jsonb_test2 (user_info) VALUES ('{"name": "Kalay", "skills": ["Python", "Java", "JavaScript"]}');
INSERT INTO json_test.jsonb_test2 (user_info) 
VALUES ('{"name": "Bob", "department": {"depart_id": "005", "depart_name": "HR"}}');

SELECT * FROM json_test.jsonb_test2 WHERE user_info ->> 'name' IN ('Kalay', 'Bob');
/*
                                user_info
--------------------------------------------------------------------------
 {"name": "Kalay", "skills": ["Python", "Java", "JavaScript"]}
 {"name": "Bob", "department": {"depart_id": "005", "depart_name": "HR"}}
*/

-- 为 user_info 列中 name=Bob 的行记录中的 department 属性添加一个 emp_count:4 键值对
UPDATE json_test.jsonb_test2 
SET user_info = jsonb_set(user_info, '{department}', '{"emp_count":4}' || (user_info -> 'department'))
WHERE (user_info #>> '{name}') = 'Bob';


SELECT * FROM json_test.jsonb_test2 WHERE user_info ->> 'name' = 'Bob';
/*
{"name": "Bob", "department": {"depart_id": "005", "emp_count": 4, "depart_name": "HR"}}
*/
```

###### 配合 `jsonb_set` 函数使用（嵌套属性）

​	**单纯的 `||` 字符串拼接**仅能实现 **JSON 中顶层属性的追加/覆盖**，如果是**对于一个 JSON 嵌套属性的操作**，那么则**需使用 `jsonb_set()` 函数与 `->` JSONB 操作符**来配合实现。

- **`<JSONB 表列> -> '<属性名>'`**：**从表列中**，**以字符串形式提取出一个 `<属性名>` JSON 属性对应的 Value 值**。

```postgresql
-- 插入一个嵌套键值对
INSERT INTO jsonb_test (json_data) VALUES ('{"detail":{"name":"jack"}}');

SELECT  (json_data -> 'detail') AS "json_data 列中 detail 属性值"  FROM  jsonb_test;
/*
json_data 列中 detail 属性值
	{"name": "jack"}
	...
就像编程中的 jsonb_test.json_data.detail
*/

-- UPDATE 更新数据
UPDATE jsonb_test SET json_data = jsonb_set(json_data, '{detail}', (json_data -> 'detail') || '{"age":18}'::jsonb);
/*
核心逻辑：
	1. jsonb_set(json_data, '{detail}') ：表示要更新的是 json_data 列中的 detail 属性
	2. (json_data -> 'detail')：将 json_data 列中的 detail 属性值以 JSON 对象字面量的形式提取出来
	3. || '{"age":18}'::jsonb：将 json_data.detail 属性的值 与 '{"age":18}' 进行拼接合并
			-- 如果 json_data.detail 属性中有 age 子属性，则覆盖为 18
			-- 否则，为 json_data.detail 属性 追加一个 age:18 的键值对
	4. 最后转为 jsonb 二进制格式存入表中
*/

SELECT  (json_data -> 'detail') FROM  jsonb_test;
/*
	{"age": 18, "name": "jack"}
*/
```

##### `-` / `#-` 操作符-删除属性或元素

- **使用 `-` 操作符可以根据 `Key键` 删除指定 JSONB 表列中某个 `Key键` 同名属性的 `{key:value}` 键值对**

```postgresql
<JSONB 列名> = <JSONB 列名> - '{Key 键}'; -- 根据 Key 键匹配，删除顶层的某个键值对

<JSONB 列名> = <JSONB 列名> #- '{Key 键,嵌套 Key 键}'; -- 根据 Key 键匹配，删除某个顶层属性下的指定嵌套键值对
```

###### 示例

- **`-` 删除顶层属性**

  ```postgresql
  -- 插入一个键值对
  INSERT INTO jsonb_test (json_data) VALUES ('{"name":"jack","age":18}');
  
  SELECT  * FROM jsonb_test;
  /*
                   json_data
  -------------------------------------------
   {"age": 18, "name": "jack"}
  */
  
  
  -- 删除所有行记录中 json_data 列中的顶层 age 属性
  UPDATE jsonb_test SET json_data = json_data - '{age}';
  
  SELECT  * FROM jsonb_test;
  /*
                   json_data
  -------------------------------------------
   {"name": "jack"}
  */
  ```

- **`#-` 删除嵌套键值对**

  ```postgresql
  -- 插入一个嵌套键值对
  INSERT INTO jsonb_test (json_data) VALUES ('{"id":"002","detail":{"name":"jack"}}');
  
  SELECT  * FROM jsonb_test;
  /*
                   json_data
  -------------------------------------------
   {"id": "002", "detail": {"name": "jack"}}
  */
  
  -- 删除所有行记录中 json_data 列中的嵌套键值对，如 detail.name 属性
  UPDATE jsonb_test SET json_data = json_data #- '{detail,name}';
  
  SELECT  * FROM jsonb_test;
  /*
                   json_data
  -------------------------------------------
   {"id": "002", "detail": {}}
  */
  ```

## `SELECT` 数据查询

### 提取某个属性值

PostgreSQL 提供了如下运算符用来提取某个 JSONB 列中指定属性的值：

#### `->` / `->>` 链式操作符

##### 说明

| 运算符    | 描述                                                         | 示例                     | 意图                                                  |
| --------- | ------------------------------------------------------------ | ------------------------ | ----------------------------------------------------- |
| **`->`**  | **根据 Key键/索引**，以**对象形式**<br />获取一个**JSON 属性值** | **`data -> 'user'`**     | 获取 **`data` 列下 `user` 属性的 JSON 对象字面量**    |
|           |                                                              | **`data -> 'a' -> 'b'`** | 获取 **`data` 列下 `a.b` 属性**的 **JSON 对象字面量** |
| **`->>`** | **根据 Key键/索引**，以**字符串形式**<br />获取一个**JSON 属性值** | **`data ->> 'name'`**    | 获取 **`data` 列下 `name` 属性**的**字符串值**        |

##### 语法

> [!IMPORTANT]
>
> 关键点：
>
> - **`->` 可以理解为一个 `.` 操作符，用于链式获取：如果是嵌套属性，则输出对象字面量，否则直接输出值**
> - **`->>` 可以理解为就是一个 `print()` 输出符**，可以用于**直接输出一个具体属性的字符串值**
>
> 语法：
>
> ```postgresql
> JSONB 列字段 -> '键1' -> '键2' ->> '子键' -- 返回的是一个子健的字符串值
> 
> JSONB 列字段 -> '键1' -> '键2' -> '子键'  -- 返回的是一个子健的JSON 对象字面量
> ```

##### 示例

```postgresql
-- 测试表
CREATE TABLE jsonb_test2
(
	user_info JSONB
);

-- 插入几条键值对数据
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Jack", "age":18}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Alice", "age":18, "department":{"depart_id":"005", "depart_name":"HR"}}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Tom", "age":18, "address":{"city":{"city_id":100, "city_name":"上海"}}}');

SELECT * FROM json_test.jsonb_test2;

/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
*/
```

按链式操作：

```postgresql
-- 以 JSON 格式获取所有行记录中 user_info 列中的某个属性值，如果是嵌套属性，则输出对象字面量，否则直接输出值
SELECT (user_info -> 'address') AS "地址" FROM json_test.jsonb_test2;
-- 输出结果：
/*
       地址
------------------
{"city": {"city_id": 100, "city_name": "上海"}}
*/

-- 以字符串形式获取所有行记录中 user_info 列中某个属性值
SELECT (user_info ->> 'name') AS "姓名", (user_info ->> 'age')::integer AS "年龄" FROM json_test.jsonb_test2; 
-- 输出结果：
/*
 姓名 | 年龄
------+------
 Jack	| 18
 Tom	| 18
 Alice|	18
*/

-- 组合链式操作
-- 可以理解为 -> 就是一个 . 操作符，最后一个 ->> 以字符串形式输出属性值，也可以使用 -> ，表现形式一样
SELECT (user_info -> 'address' -> 'city' ->> 'city_name') AS "具体城市" FROM json_test.jsonb_test2;
-- 理解为是获取 user_info.address.city.city_name 的值
/*
 具体城市
----------
 上海
*/
```

#### `#>` / `#>>` 属性路径操作符

##### 说明

| 运算符    | 描述                                                         | 示例                        | 意图                                                    |
| --------- | ------------------------------------------------------------ | --------------------------- | ------------------------------------------------------- |
| **`#>`**  | **根据 Key键/索引**，以**对象形式**<br />获取一个**JSON 属性值** | **`data #> '{user}'`**      | 获取 **`data` 列中 `user` 属性的 JSON 对象字面量**      |
|           |                                                              | **`data #> '{user,name}'`** | 获取 **`data` 列中 `user.name` 属性的 JSON 对象字面量** |
| **`#>>`** | **根据 Key键/索引**，以**文本形式**<br />获取一个**JSON 属性值** | **`data #>> '{a}'`**        | 获取 **`data` 列中 `a` 属性的字符串值**                 |
|           |                                                              | **`data #>> '{a,b,c}'`**    | 获取 **`data` 列中 `a.b.c` 属性的字符串值**             |

##### 语法

> [!IMPORTANT]
>
> 在 PostgreSQL 中，JSON 属性的提取都是通过以下方式实现：
>
> - **`{key}` 提取一个顶层单属性**
> - **`{x,y}`：提取一个双层嵌套子属性**。如 `{user,name}` 表示提取 `user.name` 属性
> - **`{x,y,z...}`：提取一个多层嵌套的子属性**。如 `{user,address,city}` 表示 提取 `user.address.city` 属性
>
> > 关键点：使用 **`{}` 表示提取属性**，使用 **`,` 逗号**表示**属性嵌套的层级**，**从左往右逐层往下深入**
>
> <img src="media/JSON 属性的提取.png" style="zoom:70%;" />
>
> 语法：
>
> ```postgresql
> JSONB 列字段 #> '{键1, 键2, 子键}' -- 返回的是 子键 的 JSON对象字面量
> 
> 
> JSONB 列字段 #>> '{键1, 键2, 子键}' -- 返回的是 子键 的 字符串值
> ```

相较于 **`->` / `->>`** 来说，**`#>` / `#>>`** 是**更直观**的一种写法。

##### 示例

```postgresql
-- 测试表
CREATE TABLE jsonb_test2
(
	user_info JSONB
);

-- 插入几条键值对数据
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Jack", "age":18}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Alice", "age":18, "department":{"depart_id":"005", "depart_name":"HR"}}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Tom", "age":18, "address":{"city":{"city_id":100, "city_name":"上海"}}}');

SELECT * FROM json_test.jsonb_test2;

/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
*/
```

按属性路径获取值：

```postgresql
-- 以 JSON 格式获取某个属性值
SELECT (user_info #> '{address}') AS "地址" FROM json_test.jsonb_test2;
-- 输出结果：
/*
       地址
------------------
{"city": {"city_id": 100, "city_name": "上海"}}
*/

-- 以字符串形式获取某个属性值
SELECT (user_info #>> '{name}') AS "姓名", (user_info #>> '{age}')::integer AS "年龄" FROM json_test.jsonb_test2; 
-- 输出结果：
/*
 姓名 | 年龄
------+------
 Jack	| 18
 Tom	| 18
 Alice|	18
*/

-- 组合链式操作
-- 可以理解为 -> 就是一个 . 操作符，最后一个 ->> 以字符串形式输出属性值
SELECT (user_info #> '{address,city,city_name}') AS "具体城市" FROM json_test.jsonb_test2;
/*
 具体城市
----------
 上海
*/
```

### `WHERE` 条件查询

PostgreSQL 提供了如下条件查询操作符：。

| **运算符** | **描述**                       | **示例**                      | **意图**                            |
| ---------- | ------------------------------ | ----------------------------- | ----------------------------------- |
| **`@>`**   | JSONB 包含（左侧是否包含右侧） | **`info @> '{"status": 1}'`** | 筛选 status 为 1 的行（带索引优化） |
| **`?`**    | 字符串是否存在于 JSON 键中     | **`info ? 'email'`**          | 检查是否存在 email 键               |

##### `@>` 检查包含键值对

```postgresql
WHERE [<JSONB 列> | 其他] @> '{key:value}';
```

核心作用：

- 检测**左边的 JSON（`<JSONB 列>` | 其他） 是否包含右边的 JSON（`{key:value}`）子属性**。
  - **包含（TRUE），则提取当前行记录**
  - **不包含（FALSE），则跳过**

###### 最左匹配原则

> [!IMPORTANT]
>
> - **`@> {key:value}`** 遵守 **“最左匹配原则”**：即**先匹配 `{key}` 键，再匹配 `{value}` 值**。
>   - **如果 `{key}` 键存在，则判断 `{value}` 值是否相同**
>     - **全量匹配**：假设 **`{value}` 值是一个具体的值**（如 `"name"`），则**判断具体值是否完全相等**
>     - **仅匹配 `key` 键**：假设 **`{value}` 值是一个 `{}` 空对象**，则**直接返回，代表匹配全部，只要有 `{key}` 键即可**
>   - **如果 `{key}` 键都不存在，直接返回 FALSE**

###### 示例

假设有一张表：

```postgresql
-- 测试表
CREATE TABLE jsonb_test2
(
	user_info JSONB
);

-- 插入几条键值对数据
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Jack", "age":18}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Alice", "age":18, "department":{"depart_id":"005", "depart_name":"HR"}}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Tom", "age":18, "address":{"city":{"city_id":100, "city_name":"上海"}}}');

SELECT * FROM json_test.jsonb_test2;

-- 假设有如下数据：
/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
*/
```

代码示例：

```postgresql
-- 全量匹配 key 和 value
SELECT * FROM json_test.jsonb_test2 WHERE user_info @> '{"age": 18, "name": "Jack"}';
-- 必须同时匹配 age=18,name=Jack，才会返回行记录
/*
          user_info
-----------------------------
 {"age": 18, "name": "Jack"}
*/

SELECT * FROM json_test.jsonb_test2 WHERE user_info @> '{"age": 18}';
/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
*/
```

```postgresql
-- 仅匹配 key 键
SELECT * FROM json_test.jsonb_test2 WHERE user_info @> '{"department": {}}';
/*
                                       user_info
---------------------------------------------------------------------------------------
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
*/

-- 匹配一个不存在的键值对
SELECT * FROM json_test.jsonb_test2 WHERE user_info @> '{"hobby": {}}';
/*
                                       user_info
---------------------------------------------------------------------------------------
*/
```

##### `?` 是否存在某个属性键

```postgresql
WHERE [<JSONB 列> | 其他] ? 'key';
```

核心作用：

- 检测**左边的 JSON（`<JSONB 列>` | 其他） 的「顶层属性」中是否存在有右边的 `key` 子属性键**。
  - **有（TRUE），则提取当前行记录**
  - **无（FALSE），则跳过**

###### 示例

假设有一张表：

```postgresql
-- 测试表
CREATE TABLE jsonb_test2
(
	user_info JSONB
);

-- 插入几条键值对数据
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Jack", "age":18}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Alice", "age":18, "department":{"depart_id":"005", "depart_name":"HR"}}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Tom", "age":18, "address":{"city":{"city_id":100, "city_name":"上海"}}}');

SELECT * FROM json_test.jsonb_test2;

/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
*/
```

代码示例：

```postgresql
-- 列名(JSON) ? '属性名': 判断 JSONB 列中是否包含有 '属性名' 的键，有返回TRUE，没有返回FALSE
SELECT * FROM json_test.jsonb_test2 WHERE user_info ? 'address'; 
-- 返回 user_info 列中包含有 address 顶层属性键的行记录
/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
*/


SELECT * FROM json_test.jsonb_test2 WHERE user_info ? 'name';
-- 返回 user_info 列中包含有 address 顶层属性键的行记录
/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
*/

-- 查询所有包含 user_info 列中 name=Kalay 或 name=Bob 的行记录
SELECT * FROM json_test.jsonb_test2 WHERE user_info ->> 'name' IN ('Kalay', 'Bob');
```

##### 查询属性值的类型转换

> [!IMPORTANT]
>
> JSON 内部提取出来的属性值默认是**文本或 JSON 格式**，**不能直接和数字进行大小比较**，需要先通过 `::integer` 进行类型转换。

```postgresql
--- 查询出所有 age > 15 的行记录
SELECT * FROM json_test.jsonb_test2 WHERE (user_info #>> '{age}')::integer > 15;
-- 核心要点：
--	1.先 (user_info #>> '{age}') 提取出所有行记录中的 user_info.age 属性值。（18,18,18）
--	2. 再通过 ::integer > 15 ，转为 INTEGER 数值类型之后判断是否大于 15
/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
*/
```

### 综合写法

假设有如下表：

```postgresql
-- 测试表
CREATE TABLE jsonb_test2
(
	user_info JSONB
);

-- 插入几条键值对数据
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Jack", "age":18}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Alice", "age":18, "department":{"depart_id":"005", "depart_name":"HR"}}');
INSERT INTO jsonb_test2(user_info) VALUES('{"name":"Tom", "age":18, "address":{"city":{"city_id":100, "city_name":"上海"}}}');

SELECT * FROM json_test.jsonb_test2;

/*
                                       user_info
----------------------------------------------------------------------------------------
 {"age": 18, "name": "Jack"}
 {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
 {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
*/
```

- 测试1：

  ```postgresql
  ---- 查询出所有 user_info 列（JSONB 对象）中键值对 age=18 的行记录，并输出姓名、所在城市、所在部门的 JSON 属性值
  SELECT 
  	(user_info #>> '{name}') AS "姓名", 
  	(user_info #> '{address,city,city_name}') AS "所在城市",
  	(user_info #> '{department,depart_name}') AS "所在部门"
  FROM json_test.jsonb_test2 WHERE user_info @> '{"age":18}';
  
  /*
   姓名    | 所在城市 | 所属部门 |
  --------+----------+---------+
   Jack	|   	   |	     |
   Tom	|   上海   |	       |
   Alice  |         |	  HR    |
  */
  ```

- 测试2：

  ```postgresql
  --- 查询出所有 age > 15 的行记录
  SELECT * FROM json_test.jsonb_test2 WHERE (user_info #>> '{age}')::integer > 15;
  -- 核心要点：
  --	1.先 (user_info #>> '{age}') 提取出所有行记录中的 user_info.age 属性值。（18,18,18）
  --	2. 再通过 ::integer > 15 ，转为 INTEGER 数值类型之后判断是否大于 15
  /*
                                         user_info
  ----------------------------------------------------------------------------------------
   {"age": 18, "name": "Jack"}
   {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "上海"}}}
   {"age": 18, "name": "Alice", "department": {"depart_id": "005", "depart_name": "HR"}}
  */
  
  --- 查询出所有 name = Jack 的行记录
  SELECT * FROM json_test.jsonb_test2 WHERE (user_info ->> 'name') = 'Jack'; -- 顶层属性
  SELECT * FROM json_test.jsonb_test2 WHERE (user_info #>> '{name}') = 'Jack'; -- 可以嵌套子属性
  -- 核心要点：
  --	1.先 (user_info ->> '{name}') 提取出所有行记录中的 user_info.name 字符串属性值。(Jack,Null,...)
  --	2. 最后判断是否 = 'Jack'
  /*
                                         user_info
  ----------------------------------------------------------------------------------------
   {"age": 18, "name": "Jack"}
  */
  ```

- 测试3：

  ```postgresql
  -- 提取所有 user_info 中全量包含 {name:Tom} 顶层键值对的行记录中，并将 address.city.city_name 修改为 '北京'
  UPDATE json_test.jsonb_test2 
  SET user_info = jsonb_set(user_info, '{address,city,city_name}', '"北京"'::jsonb)
  WHERE user_info @> '{"name":"Tom"}';
  
  SELECT * FROM json_test.jsonb_test2 WHERE user_info @> '{"name":"Tom"}';
  /*
                                         user_info
  ----------------------------------------------------------------------------------------
  {"age": 18, "name": "Tom", "address": {"city": {"city_id": 100, "city_name": "北京"}}}
  */
  ```

- ...

## `GIN` 索引优化

如果直接在 JSON 字段中查询，数据库会进行全表扫描。

JSONB 支持 **GIN（Generalized Inverted Index，通用倒排索引）**，这是高性能处理文档数据的关键。

### 全文 GIN 索引

为**一个字段创建通用的 GIN 索引**，支持**任意结构的 `@>` 包含查询**：
```postgresql
CREATE INDEX idx_orders_info ON orders USING gin (info);
```

### 特定路径索引（更省空间）

如果只需要**频繁查询 JSON 中的某一个特定属性**，可以创建基于表达式的 B-Tree 索引：

```postgresql
CREATE INDEX idx_orders_customer ON orders ((info->>'customer'));
```

## 数组元素操作

JSON 中基本都会包含有数组元素，如下：

### `INSERT` 插入数据元素

#### JSON 格式

在 `INSERT` 插入 JSON 数据时，必须**严格按照 JSON 字符串的格式**来传入：

- **外层使用 `''` 单引号包裹，表示一个字符串**

- **内层：**

  - **`{}` 对象： 原样保留**

    - **`Key` 属性键：使用 `""` 双引号包裹**
    - **`Value` 属性值：使用 `""` 双引号包裹**

    ```json
    // 单层属性
    '{"x": "y"}'
    
    // 双层嵌套属性
    '{"x": {"y":"z"}}'
    
    // 多层嵌套属性
    '{"x": { "y": "z": { "a": "b" }}}'
    ```

  - **`[]` 数组**：**原样保留**

    - **`element` 元素：使用 `""` 双引号包裹**

    ```json
    '["x"]' // 单层
    
    '["x", ["y", "z"]]' // 多层嵌套
    
    '["x", {"z": "b"}]' // 数组 + 对象
    ```

```postgresql
SET search_path TO json_test;

CREATE TABLE json_test
(
	json_info JSON  -- JSON 纯文本格式
);

-- INSERT 插入数组元素
INSERT INTO json_test.jsonb_test2(user_info) 
VALUES ('{
	"name": "Kevin", 
	"department": [{"depart_id": "235", "roles": ["管理员"]}], 
	"skills": ["Python", "Java", "JavaScript"]
}');

SELECT * FROM json_test.jsonb_test2 WHERE user_info ->> 'name' = 'Kevin';
/*
{
	"name": "Kevin", 
	"skills": ["Python", "Java", "JavaScript"], 
	"department": [
		{"roles": ["管理员"], "depart_id": "235"}
	]
}
*/
```

### `SELECT` 查询数组元素

PostgreSQL **对于 JSON 中的数组、对象元素的提取查询操作**，都是**通过 `->` / `->>` 和 `#>` / `#>>`** 这几个操作符来实现的。

#### `->` / `->>` 链式操作符

##### 说明

| 运算符    | 描述                                                         | 示例                     | 意图                                                  |
| --------- | ------------------------------------------------------------ | ------------------------ | ----------------------------------------------------- |
| **`->`**  | **根据 Key键/索引**，以**对象形式**<br />获取一个**JSON 属性值** | **`data -> 'user'`**     | 获取 **`data` 列下 `user` 属性的 JSON 对象字面量**    |
|           |                                                              | **`data -> 'a' -> 'b'`** | 获取 **`data` 列下 `a.b` 属性**的 **JSON 对象字面量** |
| **`->>`** | **根据 Key键/索引**，以**字符串形式**<br />获取一个**JSON 属性值** | **`data ->> 'name'`**    | 获取 **`data` 列下 `name` 属性**的**字符串值**        |

##### 语法

关键点：如果 JSON 中**包含有数组元素**，则 **JSON 属性路径中的 Key 键名**换成**数字下标**（从 `0` 开始计数）。

- **链式写法**：**`json_data -> 'a' -> 'b' -> 0 ->> 'c'`**

```postgresql
JSONB 列 -> 'a' -> 0 -- 获取 JSONB 列中 a 属性的第 0 个数组。如： a[0]

JSONB 列 -> 'a' -> 'b' -> 2 -- 获取 JSONB 列中 a.b 属性的第 2 个数组。如：a.b[2]

JSONB 列 -> 'a' -> 'b' -> 0 -> 'c' -- 获取 JSONB 列中 a.b 属性的第 0 个数组中的 c 属性（对象字面量）。如：a.b[0].c

JSONB 列 -> 'a' -> 'b' -> 0 ->> 'c' -- 获取 JSONB 列中 a.b 属性的第 0 个数组中的 c 属性（字符串）。如：a.b[0].c

....
```

##### 示例

```postgresql
-- 假设 jsonb_test2 表中的 user_info 列有一条这样的数据：
/*
{
	"name": "Kevin", 
	"skills": ["Python", "Java", "JavaScript"], 
	"department": [
		{"roles": ["管理员"], "depart_id": "235"}
	]
}
*/

-- 获取 jsonb_test2 表中 user_info 列中的 department 数组属性的 第 0 个数组元素的 roles 属性值（对象字面量）
SELECT (user_info -> 'department' -> 0 -> 'roles') FROM json_test.jsonb_test2;
/*
  ?column?
------------
 ["管理员"]
*/

-- 获取 jsonb_test2 表中 user_info 列中的 skills 数组属性的 第 2 个数组元素（字符串），WHERE 条件为 name=Kevin
SELECT (user_info -> 'skills' ->> 2) FROM json_test.jsonb_test2 WHERE (user_info ->> 'name') = 'Kevin';
-- 这里最后之所以使用 ->> 是因为已经知道 skills[2] 数组元素是一个字符串，而非对象
/*
   ?column?
--------------
 "JavaScript"
*/

-- 获取 jsonb_test2 表中 user_info 列中的 department 数组属性的 第 0 个数组元素的 roles 属性中的第 0 个数组元素值
-- WHERE 条件为 skills 的第 0 个数组元素是 'Python'
SELECT (user_info -> 'department' -> 0 -> 'roles' ->> 0) FROM json_test.jsonb_test2 
WHERE (user_info -> 'skills' ->> 0) = 'Python';
/*
 ?column?
----------
 "管理员"
*/
```

#### `#>` / `#>>` 属性路径操作符

##### 说明

| 运算符    | 描述                                                         | 示例                        | 意图                                                    |
| --------- | ------------------------------------------------------------ | --------------------------- | ------------------------------------------------------- |
| **`#>`**  | **根据 Key键/索引**，以**对象形式**<br />获取一个**JSON 属性值** | **`data #> '{user}'`**      | 获取 **`data` 列中 `user` 属性的 JSON 对象字面量**      |
|           |                                                              | **`data #> '{user,name}'`** | 获取 **`data` 列中 `user.name` 属性的 JSON 对象字面量** |
| **`#>>`** | **根据 Key键/索引**，以**文本形式**<br />获取一个**JSON 属性值** | **`data #>> '{a}'`**        | 获取 **`data` 列中 `a` 属性的字符串值**                 |
|           |                                                              | **`data #>> '{a,b,c}'`**    | 获取 **`data` 列中 `a.b.c` 属性的字符串值**             |

##### 语法

> [!IMPORTANT]
>
> 在 PostgreSQL 中，JSON 属性的提取都是通过以下方式实现：
>
> - **`{key}` 提取一个顶层单属性**
> - **`{x,y}`：提取一个双层嵌套子属性**。如 `{user,name}` 表示提取 `user.name` 属性
> - **`{x,y,z...}`：提取一个多层嵌套的子属性**。如 `{user,address,city}` 表示 提取 `user.address.city` 属性
>
> > 关键点：使用 **`{}` 表示提取属性**，使用 **`,` 逗号**表示**属性嵌套的层级**，**从左往右逐层往下深入**
>
> <img src="media/JSON 属性的提取.png" style="zoom:70%;" />
>
> 语法：
>
> - **路径写法**：**`json_data #>> '{detail, users, 0, name}'`**
>
> ```postgresql
> JSONB 列 #> '{a,0}' -- 获取 JSONB 列中 a 属性的第 0 个数组。如： a[0]
> 
> JSONB 列 #> '{a,b,2}' -- 获取 JSONB 列中 a.b 属性的第 2 个数组。如：a.b[2]
> 
> JSONB 列 #> '{a,b,0,c}' -- 获取 JSONB 列中 a.b 属性的第 0 个数组中的 c 属性（对象字面量）。如：a.b[0].c
> 
> JSONB 列 #>> '{a,b,0,c}' -- 获取 JSONB 列中 a.b 属性的第 0 个数组中的 c 属性（字符串）。如：a.b[0].c
> 
> ....
> ```

相较于 **`->` / `->>`** 来说，**`#>` / `#>>`** 是**更直观**的一种写法。

##### 示例

```postgresql
-- 假设 jsonb_test2 表中的 user_info 列有一条这样的数据：
/*
{
	"name": "Kevin", 
	"skills": ["Python", "Java", "JavaScript"], 
	"department": [
		{"roles": ["管理员"], "depart_id": "235"}
	]
}
*/

-- 获取 jsonb_test2 表中 user_info 列中的 department 数组属性的 第 0 个数组元素的 roles 属性值（对象字面量）
SELECT (user_info #> '{department,0,roles}') FROM json_test.jsonb_test2;
/*
  ?column?
------------
 ["管理员"]
*/

-- 获取 jsonb_test2 表中 user_info 列中的 skills 数组属性的 第 2 个数组元素（字符串），WHERE 条件为 name=Kevin
SELECT (user_info #>> '{skills,2}') FROM json_test.jsonb_test2 WHERE (user_info #>> 'name') = 'Kevin';
-- 这里最后之所以使用 #>> 是因为已经知道 skills[2] 数组元素是一个字符串，而非对象
/*
   ?column?
--------------
 "JavaScript"
*/

-- 获取 jsonb_test2 表中 user_info 列中的 department 数组属性的 第 0 个数组元素的 roles 属性中的第 0 个数组元素值
-- WHERE 条件为 skills 的第 0 个数组元素是 'Python'
SELECT (user_info #>> '{department,0,roles,0}') FROM json_test.jsonb_test2   
WHERE (user_info #>> '{skills,0}') = 'Python'; -- 相当于 skills[0]（'Python'） = 'Python'
/*
 ?column?
----------
 "管理员"
*/
```

### `UPDATE` 更新数据

与表操作一致....

# `ARRAY` 数组类型处理

## 基本概念

PostgreSQL 的**数组类型（Array）**功能非常强大，它允许**在单个字段中存储多个同类型的值**。掌握数组的操作符和函数，能在处理一对多关系、标签系统等场景时游刃有余。

## 运算符

操作符是数组查询的高频工具，特别是在 `WHERE` 子句中做条件筛选时。

| **操作符** | **含义**                                  | **示例**                     | **结果**    |
| ---------- | ----------------------------------------- | ---------------------------- | ----------- |
| **`=`**    | 等于（元素和顺序必须完全一致）            | `ARRAY[1,2] = ARRAY[1,2]`    | `true`      |
| **`@>`**   | **包含**（左边是否包含右边的所有元素）    | `ARRAY[1,2,3] @> ARRAY[2,3]` | `true`      |
| **`<@`**   | **被包含**（左边是否是右边的子集）        | `ARRAY[2,3] <@ ARRAY[1,2,3]` | `true`      |
| **`&&`**   | **重叠/有交集**（是否有任意一个共同元素） | `ARRAY[1,2] && ARRAY[2,4]`   | `true`      |
| **`||`**   | **拼接**（连接两个数组或向数组追加元素）  | `ARRAY[1,2] || ARRAY[3,4]`   | `{1,2,3,4}` |

💡 **技巧：** `@>` 和 `&&` 在实际业务中非常常用。例如查询“拥有‘Python’和‘Docker’标签的用户”，可以用 `tags @> ARRAY['Python', 'Docker']`。

## 核心数据类型 `ARRAY` | `<数据类型>[]`

PostgreSQL 默认提供了一个 **`ARRAY`** 以及 **`<数据类型>[]` | `<数据类型>[][]`** 写法来**创建与定义一个数组类型的 Column 列**。

### 数据定义

#### `<数据类型>[]` 创建数组类型字段

- **`<数据类型>[]`**：**在类型后直接加 `[]` / `[][]`**，可以创建整数、文本、日期、布尔...等类型的**一维、多维数组**。

```postgresql
CREATE TABLE <模式名>.<表名>
(
	<列字段名> <数据类型>[], -- 一维数组
	<列字段名> <数据类型>[][] -- 二维数组
);
```

核心作用：在 **`CREATE TABLE` 创建表**要定义**一个数组类型的 Column 列**。

> 💡 **注意：** 在 PostgreSQL 中，虽然可以在 `[]`方括号内指定长度（如 `integer[4]`），但 PostgreSQL 实际上会**忽略**这个长度限制，它不会强制约束数组的元素个数。因此，通常直接写 `[]` 即可。

示例：

```postgresql
CREATE TABLE items (
    id serial PRIMARY KEY,
    name text,
    -- 1. 文本数组（例如：存储商品的标签）
    tags text[], 
    -- 2. 整数数组（例如：存储每季度的销售额）
    quarterly_sales integer[],
    -- 3. 二维或多维数组（例如：存储坐标矩阵，语法上加多个 [] 即可）
    matrix integer[][]
);
```

#### 指定默认值

要想给数组类型的字段设定默认值，有 2 种写法：

##### `ARRAY[]` 空数组

```postgresql
CREATE TABLE <模式名>.<表名>
(
	<列字段名> <数据类型>[] DEFAULT ARRAY[]::<数据类型>[], -- 一维数组
	<列字段名> <数据类型>[][] DEFAULT ARRAY[][]::<数据类型>[][] -- 二维数组
);
```

##### `{element}` 显式赋值

> [!IMPORTANT]
>
> 注意点：PostgreSQL 中的**数组是以 `{}` 元组形式写入存储的**。

```postgresql
CREATE TABLE <模式名>.<表名>
(
	<列字段名> <数据类型>[] DEFAULT '{<默认值>}', -- 一维数组
	<列字段名> <数据类型>[][] DEFAULT '{{<默认值>}}' -- 二维数组
);
```

##### 示例

```postgresql
CREATE TABLE users (
    id serial PRIMARY KEY,
    username text,
    -- 默认值为空数组
    roles text[] DEFAULT ARRAY[]::text[],
    -- 默认包含 'guest' 标签的数组
    groups text[] DEFAULT '{guest}'
);
```

### 数据更新

#### `INSERT` 插入数据

##### `ARRAY[]` 数组方式（推荐）

这种方式直接使用 SQL 关键字，代码可读性好，不需要担心字符串转义问题。

```postgresql
INSERT INTO <模式名>.<表名> VALUES (ARRAY[<值1>, <值2>,...]); -- 一维数组


INSERT INTO <模式名>.<表名> VALUES (ARRAY[[<值1>, <值2>,...], [<值3>, <值4>],...]); -- 二维数组
```

示例：

```postgresql
CREATE TABLE array_table_test
(
	skills TEXT[] DEFAULT ARRAY[]::TEXT[], -- 一维文本数组，默认值是一个空数组，表示为 {}
	"position" NUMERIC(6,2)[][], -- 二维浮点数数组，经纬度位置
	"price" INTEGER[] DEFAULT '{0}' -- JSON 数组，默认值为 [0] -> {0}
);


-- INSERT 插入数据：ARRAY[] 方式
INSERT INTO array_table_test(skills, "position", price) 
VALUES (
	ARRAY['Python', 'Java', 'JavaScript'],
	ARRAY[[165.15,15.65],[498.15,31.47],[213.01,51.17]],
	ARRAY[1000,2000,3000,4000,5000]
);


SELECT * FROM array_test.array_table_test;
/*
          skills          |                    position                    |           price
--------------------------+------------------------------------------------+----------------------------
 {Python,Java,JavaScript} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1000,2000,3000,4000,5000}
*/
```

##### `{}` 元组字面量方式

> [!IMPORTANT]
>
> 注意点：PostgreSQL 中的**数组是以 `{}` 元组形式写入存储的**。
>
> - **字符串需使用 `""` 包裹**
>
> 这是 PostgreSQL 传统的数组表示法。**整个数组用单引号 `'` 包裹，内部使用大括号 `{}`，元素之间用逗号 `,` 分隔**。

```postgresql
INSERT INTO <模式名>.<表名> VALUES ('{<值1>, <值2>,...}'); -- 一维数组

INSERT INTO <模式名>.<表名> VALUES ('{{<值1>, <值2>,...}, {<值3>, <值4>},...},...'); -- 二维数组
```

示例：

```postgresql
CREATE TABLE array_table_test
(
	skills TEXT[] DEFAULT ARRAY[]::TEXT[], -- 一维文本数组，默认值是一个空数组，表示为 {}
	"position" NUMERIC(6,2)[][], -- 二维浮点数数组，经纬度位置
	"price" INTEGER[] DEFAULT '{0}' -- JSON 数组，默认值为 [0] -> {0}
);


-- INSERT 插入数据：{} 元组字面量方式
INSERT INTO array_table_test(skills, "position", price) VALUES (
	'{"Python", "Java", "JavaScript"}',
	'{{165.15,15.65},{498.15,31.47},{213.01,51.17}}',
	'{1000,2000,3000,4000,5000}'
);


SELECT * FROM array_test.array_table_test;
/*
          skills          |                    position                    |           price
--------------------------+------------------------------------------------+----------------------------
 {Python,Java,JavaScript} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1000,2000,3000,4000,5000}
 {Python,Java,JavaScript} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1000,2000,3000,4000,5000}
*/
```

#### `UPDATE` 更新数据

PostgreSQL 中数组的更新主要分为 **“覆盖”**、**“追加”、"删除"** 三种形式：

###### 完全覆盖

```postgresql
UPDATE <模式名>.<表名> SET <列> = ARRAY[<值>,...] | '{值,...}'; [WHERE <条件表达式>];
```

示例：

```postgresql
SELECT * FROM array_test.array_table_test;
/*
          skills          |                    position                    |           price
--------------------------+------------------------------------------------+----------------------------
 {Python,Java,JavaScript} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1000,2000,3000,4000,5000}
 {Python,Java,JavaScript} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1000,2000,3000,4000,5000}
*/

-- UPDATE 修改元素
-- 完全覆盖
UPDATE array_test.array_table_test SET price = ARRAY[1,2,3,4] WHERE skills @> ARRAY['Java'];
-- 将所有行记录中 skills 列数组包含有 {"Java"} 数组子元素的 price 列修改覆盖为 {1,2,3,4}

SELECT * FROM array_test.array_table_test;
/*
          skills          |                    position                    |   price
--------------------------+------------------------------------------------+-----------
 {Python,Java,JavaScript} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1,2,3,4}
 {Python,Java,JavaScript} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1,2,3,4}
*/
```

其余数组的操作基本都是通过PostgreSQL 内置提供的函数来实现：

##### 追加数组元素

###### `array_append(x[], y)` 函数（一维数组）

这是 PostgreSQL 提供的内置函数，专门用于**在一个一维数组 `x[]` 的末尾处追加一个元素**。

```postgresql
array_append(<一维数组列名 ARRAY[...] | '{...}'>, <追加的数组元素值>); -- 一维数组追加

-- 查看
SELECT array_append(ARRAY[1,2], 3);
-- 输出结果：{1,2,3}
```

示例：

```postgresql
-- 为所有行记录中 skills 列数组末尾处添加一个 {"PostgreSQL"} 子元素，前提是该 skills 列数组中没有该子元素
UPDATE array_test.array_table_test SET skills = array_append(skills, 'PostgreSQL')
WHERE NOT skills @> ARRAY['PostgreSQL'];  
-- skills @> ARRAY['PostgreSQL'] skills 数组是否包含 "PostgreSQL" 子元素，取反！


SELECT * FROM array_test.array_table_test;
/*
               skills                |                    position                    |   price
-------------------------------------+------------------------------------------------+-----------
 {Python,Java,JavaScript,PostgreSQL} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1,2,3,4}
 {Python,Java,JavaScript,PostgreSQL} | {{165.15,15.65},{498.15,31.47},{213.01,51.17}} | {1,2,3,4}
*/
```

###### `array_cat()` 数组合并（二维数组）

> [!IMPORTANT]
>
> ​	由于 **`array_append()` 只能用于一维数组**，所以若**想要给二位数组追加一个一维子数组元素**，需使用另一个 **`array_cat()` 函数**。

```postgresql
array_cat(<二维数组列名 ARRAY[...] | '{...}'>, ARRAY[[...]]::<数据类型>[]);

-- 查看
SELECT array_cat(ARRAY[[1],[2]], ARRAY[[3], [4]]::INTEGER[]);
-- 输出结果：{{1},{2},{3},{4}}
```

核心作用：**把第二个数组连接到第一个数组的末尾处**。

> 注意：要追加的元素外面**必须包裹两层方括号**，使其**保持二维结构**，**才能正确拼接到现有的二维数组中**。

示例：

```postgresql
-- 为所有行记录中 position 列 > 二维数组末尾处添加一个 {468.18,998.88} 子元素，前提是该 position 列数组中没有该子元素
UPDATE array_test.array_table_test 
SET "position" = array_cat("position", ARRAY[[468.18, 998.88]]::numeric[])
WHERE NOT "position" @> ARRAY[[468.18, 998.88]]::numeric[];

SELECT "position"::NUMERIC[][] FROM array_test.array_table_test; -- 更新之后
/*
                            position
----------------------------------------------------------------
 {{165.15,15.65},{498.15,31.47},{213.01,51.17},{468.18,998.88}}
 {{165.15,15.65},{498.15,31.47},{213.01,51.17},{468.18,998.88}}
*/
```

也可以直接使用 **`||` 拼接操作符**，它**更为直观**！！

###### `x || y` 数组拼接

核心作用：**将 `y` 数组拼接到 `x` 数组的末尾处，形成一个新的数组**。

> [!IMPORTANT]
>
> 注意：**千万不要使用 `'{}'` 与 另一个 `'{}'` 一起拼接**，否则**会被识别为字符串拼接**！
>
> - 建议**左右两边都使用 `ARRAY[]`！**

```postgresql
<ARRAY[...] | '{...}'> || <ARRAY[...] | '{...}'> -- 一维数组

<ARRAY[[...]] | '{{...}}'> || <ARRAY[[...]] | '{{...}}'>::<数据类型>[] -- 二维数组
-- 这里 ::<数据类型>[] 的作用是声明它是一个一维数组，用于被包含在左边的二维数组中。


-- 查看
-- 一维数组
SELECT ARRAY[1,2] || ARRAY[3,4]; -- {1,2,3,4}
SELECT  '{1,2}' || '{3,4}'; -- {1,2}{3,4} -- 注意：这个只是字符串拼接！！
SELECT ARRAY[1,2] || '{3,4}'; -- {1,2,3,4}

-- 二维数组
SELECT ARRAY[[1],[2]] || ARRAY[[3],[4]]::INTEGER[]; -- {{1},{2},{3},{4}}
```

示例：

- 一维数组：

  ```postgresql
  -- 为所有行记录的 price 列一维数组末尾处追加一个子数组拼接在一起，形成一个新数组
  UPDATE array_test.array_table_test SET price = price || ARRAY[5,6,7,8];
  
  SELECT price::INTEGER[] FROM array_test.array_table_test;
  /*
         price     
  +-------------------+
   {1,2,3,4,5,6,7,8}
   {1,2,3,4,5,6,7,8}
  */
  
  -- 将所有行记录的 price 列一维数组覆盖为一个新数组
  UPDATE array_test.array_table_test SET price = ARRAY[10,11,12,13] || ARRAY[20,21,22,23];
  
  SELECT price::INTEGER[] FROM array_test.array_table_test;
  /*
             price
  ---------------------------
   {10,11,12,13,20,21,22,23}
   {10,11,12,13,20,21,22,23}
  */
  ```

- 二维数组：

  注意：要追加的元素外面**必须包裹两层方括号**，使其保持二维结构，才能正确拼接到现有的二维数组中。

  ```postgresql
  -- 为所有行记录中 position 列 > 二维数组末尾处添加一个 {468.18,998.88} 子元素，前提是该 position 列数组中没有该子元素
  UPDATE array_test.array_table_test 
  SET "position" = "position" || ARRAY[[468.18, 998.88]]::numeric[]
  WHERE NOT "position" @> ARRAY[[468.18, 998.88]]::numeric[];
  
  SELECT "position"::NUMERIC[][] FROM array_test.array_table_test; -- 更新之后
  /*
                              position
  ----------------------------------------------------------------
   {{165.15,15.65},{498.15,31.47},{213.01,51.17},{468.18,998.88}}
   {{165.15,15.65},{498.15,31.47},{213.01,51.17},{468.18,998.88}}
  */
  ```

##### 删除数组元素

###### `array_remove(x,ele)` 函数

> 注：PostgreSQL 官方文档明确指出：**`array_remove` 目前只支持一维数组**。
>
> > 想要删除二维数组的子数组元素，最常用且有效的方法是：**将二维数组用 `unnest()` 展开为一维数组，过滤掉不要的元素，然后再重新组装回去**。

```postgresql
array_remove(<一维数组列名 ARRAY[...] | '{...}'>, <要移除的元素>);

-- 查看
SELECT array_remove(ARRAY[1,2,3], 3); -- 结果：{1,2}
```

核心作用：移除数组中**所有**匹配该值的元素。

示例：

```postgresql
SELECT skills::TEXT[] FROM array_test.array_table_test;
/*
               skills
-------------------------------------
 {Python,Java,JavaScript,PostgreSQL}
 {Python,Java,JavaScript,PostgreSQL}
*/

-- 移除所有行记录中 skills 列一维数组中的 "PostgreSQL" 数组子元素
UPDATE array_test.array_table_test SET skills = array_remove(skills, 'PostgreSQL');

SELECT skills::TEXT[] FROM array_test.array_table_test;
/*
               skills
-------------------------------------
 {Python,Java,JavaScript}
 {Python,Java,JavaScript}
*/
```

###### 移除二维数组元素

```postgresql
SELECT array_agg(ARRAY[x, y])
FROM (
    -- 假设你的二维数组是包含 x 和 y 坐标
    -- 这里用 unnest 展开时需要对应好你的维度数量
    SELECT arr[1] as x, arr[2] as y
    FROM unnest(ARRAY[[165.15,15.65],[498.15,31.47],[213.01,51.17]]) AS arr
) sub
WHERE NOT (x = 498.15 AND y = 31.47); -- 过滤掉指定的坐标

-- 结果: {{165.15,15.65},{213.01,51.17}}
```

### `SELECT` 数据查询

假设有一个用户表 `users`，其中包含一个文本数组类型的标签字段 `tags text[]`。

- 场景 A：查询拥有某一个特定标签的用户

  可以使用高级修改版的 `ANY` 关键字：

  ```postgresql
  SELECT * FROM users WHERE 'Java' = ANY(tags);
  ```

- 场景 B：查询同时拥有 'MySQL' 和 'Go' 两个标签的用户

  ```postgresql
  SELECT * FROM users WHERE tags @> ARRAY['MySQL', 'Go'];
  ```

- 测试：

  ```postgresql
  -- 创建一个包含 3 个整数的数组
  SELECT ARRAY[1, 2, 3] AS arr1;
  
  -- 使用大括号（注意整个数组需要用单引号括起来）
  SELECT '{1, 2, 3}'::int[] AS arr2;
  ```

- ...

#### 常用函数

PostgreSQL 提供了丰富的内置函数来处理数组。

##### `unnest` 数组展开

`unnest` 是最常用的数组函数，它可以将一个数组**展平（行转列）**，变成多行数据，非常适合用来做关联查询或统计。

```postgresql
SELECT unnest(ARRAY['apple', 'banana', 'orange']) AS fruit;
-- 结果会变成 3 行：
-- apple
-- banana
-- orange
```

##### 字符串-数组 转换

- **`string_to_array`**：将**字符串按分隔符切分成数组**。
- **`array_to_string`**：将**数组元素用分隔符拼接成字符串**。

```postgresql
-- 字符串转数组
SELECT string_to_array('a,b,c', ','); -- 结果: {a,b,c}

-- 数组转字符串
SELECT array_to_string(ARRAY['a', 'b', 'c'], '-'); -- 结果: 'a-b-c'
```

##### 获取数组长度

- **`array_length(array, dimension)`**：获取**指定维度的长度（一维数组传 `1`，二维数组传 `2`）**
- **`cardinality(array)`**：获取**数组中所有元素的总数**（最推荐，简单直观）

```postgresql
SELECT cardinality(ARRAY[1, 2, 3, 4]); -- 结果: 4
SELECT array_length(ARRAY[1, 2, 3, 4], 1); -- 结果: 4
```

## `GIN` 索引建立

如果建立数组字段是为了在之后**频繁进行“包含（`@>`）”、“是否有交集（`&&`）”等查询**，**强烈建议在建表后为该数组字段建立 GIN 索引**，否则随着数据量变大，查询性能会急剧下降：

```postgresql
-- 为 items 表的 tags 字段建立 GIN 索引
CREATE INDEX idx_items_tags ON items USING gin (tags);
```

### 示例

```postgresql
SET search_path TO array_test;
CREATE TABLE items
(
	item_arr INTEGER[]
);

-- 批量插入 一千万条 数组数据
DO $$
DECLARE
	v_i INTEGER := 0;
BEGIN
	FOR v_i IN 0..10000000 LOOP
		INSERT INTO array_test.items(item_arr) VALUES (ARRAY[v_i]);
	END LOOP;
END $$;

-- 查询数据
SELECT item_arr::INTEGER[] FROM array_test.items;
/*
  item_arr
------------
 {0}
 {1}
 {2}
 ...
 {10000000}
*/

-- 查询耗时测试：从一千万条数组数据中，找出数组元素值完全等于 {6094423} 的行记录
EXPLAIN ANALYSE SELECT item_arr::INTEGER[] FROM array_test.items WHERE item_arr = ARRAY[6094423];
-- 耗时：958.812 ms

-- 为 item_arr 数组字段建立一个 GIN 索引
CREATE INDEX idx_item_arr_gin ON items USING btree(item_arr);


-- 再次查询耗时测试：从一千万条数组数据中，找出数组元素值完全等于 {6094423} 的行记录
EXPLAIN ANALYSE SELECT item_arr::INTEGER[] FROM array_test.items WHERE item_arr = ARRAY[6094423];
-- 耗时：0.180 ms
```

可以看出，同一种针对于 `item_arr` 字段的 SQL 查询速度，从原先的 `958.812 ms` 提升到了 `0.180 ms`，加快了 4511 倍！
