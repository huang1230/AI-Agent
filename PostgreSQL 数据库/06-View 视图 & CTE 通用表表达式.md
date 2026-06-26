# CTE 通用表表达式

## 基本概念

​	**CTE（Common Table Expression，通用表表达式）** 是 SQL 中用来定义**临时结果集**的一种语法。可以理解为 CTE 是一个**只在当前 SQL 语句中有效**的**临时视图**，可以极大地提高复杂 SQL 查询的可读性和维护性。

### 为什么需要 CTE？（铺平SQL结构）

在没有 CTE 之前，面对复杂业务的 SQL 查询时，通常只能通过**嵌套子查询**的形式来定义：

```postgresql
SELECT * FROM (
    -- 主查询的数据源来自另一个子查询的临时虚拟表
    SELECT * FROM (
        -- 临时虚拟表的数据源又来自其他表...
        SELECT ... -- 这里的代码已经缩进到太平洋了
    ) AS inner_data
) AS outer_data;
```

像这样的 SQL 代码很冗余、结构太过复杂；可以使用 CTE 语法来优化复杂的 SQL 语句结构。

## 基本语法

```sql
WITH <临时变量名(子查询结果)> AS (
    -- 子查询会作为一张虚拟结果表 注入 到 WITH <临时变量名(子查询结果)> 中， <临时变量名(子查询结果)> 可以理解为是子查询在外部作用域的别名
    SELECT <查询结果> FROM ...
) 
-- 主查询（使用虚拟结果表的变量别名）
SELECT ... FROM <CTE 临时变量名(子查询结果)>
```

核心机制：可以把 CTE 看做是**把一段复杂的嵌套 SQL 子查询语句 “提取” 出来**，并**注入**到一个**临时变量（数据源）**中，在**紧随其后的 SQL 主查询语句**中**直接像对待普通表一样操作这个临时变量（数据源）**。

- 关键点：可以理解为 **`WITH <临时变量名(子查询结果)>` 变量**是 **`AS()` 中的 `SELECT` 子查询虚拟结果表**在**外部作用域**的**别名引用**。

<img src="media/CTE 的作用域概念.png" style="zoom:50%;" />

核心要点：CTE 最大的作用就是通过**由上至下**的执行方式来**扁平化**编写**复杂嵌套的 SQL 语句（由内往外）**，以此来提高代码可读性。

核心优势：

- 提高代码可读性与可维护性
- CTE 语句中**不仅可以写 `SELECT` 子查询**，还**可以写 `INSERT` 语句（通过 `RETURNING` 关键字返回插入的结果）**

> CTE 本质上是**将 `AS ()` 代码块中的 SQL 语句执行结果**通过 **`WITH xxx` 注入到一个临时变量**中，并**在后续的 SQL 代码中使用**。

### 示例

假设有如下表：

```postgresql
SET search_path TO high_school;

-- 1. 创建系别表 (Department)
CREATE TABLE department (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) NOT NULL
);
COMMENT ON TABLE department IS '系别表';
COMMENT ON COLUMN department.dept_id IS '系别ID';
COMMENT ON COLUMN department.dept_name IS '系别名';

-- 2. 创建班级表 (Class)
CREATE TABLE d_class (
    class_id INT PRIMARY KEY, -- 这里手动指定ID，方便与学生表关联
    class_name VARCHAR(50) NOT NULL,
    dept_id INT,
    CONSTRAINT fk_department FOREIGN KEY (dept_id) REFERENCES department(dept_id) ON DELETE SET NULL
);
COMMENT ON TABLE d_class IS '班级表';
COMMENT ON COLUMN d_class.class_id IS '班级ID';
COMMENT ON COLUMN d_class.class_name IS '班级名';
COMMENT ON COLUMN d_class.dept_id IS '所属系别ID';

-- 3. 创建学生表 (Student)
CREATE TABLE student (
    student_id SERIAL PRIMARY KEY, -- 使用 SERIAL 实现自增
    student_name VARCHAR(50) NOT NULL,
    gender VARCHAR(10),
    age INT,
    class_id INT,
    CONSTRAINT fk_class FOREIGN KEY (class_id) REFERENCES d_class(class_id) ON DELETE SET NULL
);
COMMENT ON TABLE student IS '学生表';
COMMENT ON COLUMN student.student_id IS '学生ID';
COMMENT ON COLUMN student.student_name IS '学生姓名';
COMMENT ON COLUMN student.gender IS '学生性别';
COMMENT ON COLUMN student.age IS '学生年龄';
COMMENT ON COLUMN student.class_id IS '学生所在班级ID';


-- 1. 插入2个系别数据 (让 SERIAL 自动生成 ID)
INSERT INTO department (dept_name) VALUES 
('理工系'),
('文学系');

-- 2. 插入4个班级数据 (1和2对应上面自动生成的系别ID)
INSERT INTO d_class (class_id, class_name, dept_id) VALUES 
(101, '2022-1701班', 1),
(102, '2022-1702班', 1),
(103, '2022-1703班', 2),
(104, '2022-1704班', 2);

-- 3. 插入10个学生数据 (student_id 会自动递增)
INSERT INTO student (student_name, gender, age, class_id) VALUES 
('张三', '男', 20, 101),
('李四', '男', 21, 101),
('王五', '女', 20, 101),
('赵六', '男', 19, 102),
('孙七', '女', 22, 102),
('周八', '男', 20, 103),
('吴九', '女', 21, 103),
('郑十', '女', 20, 104),
('刘一', '男', 19, 104),
('陈二', '女', 21, 104);
```

#### 嵌套子查询的优化

- 复杂嵌套 SQL：

  ```sql
  -- 查看理工系中年龄超过 20 岁的学生信息
  SELECT s_c.student_name AS "学生姓名", s_c.class_name AS "所在班级", d.dept_name AS "所属系别" 
  FROM (
  	-- 子查询数据源：返回所有系别中，年龄超过 20 岁的学生信息（姓名、所在班级名称、班级所属系别ID）
  	SELECT s.student_name, d_c.class_name, d_c.dept_id
  	FROM student s 
  	LEFT JOIN d_class d_c ON d_c.class_id = s.class_id
  	WHERE s.age > 20
  ) s_c -- 数据源别名（临时虚拟表）
  JOIN department d
  ON d.dept_id = s_c.dept_id
  WHERE d.dept_name = '理工系';
  /*
   学生姓名 | 所在班级 |  所属系别  
  --------+----------+----------+ 
    李四	2022-1701班	理工系
    孙七	2022-1702班	理工系
    吴九	2022-1703班	文学系
    陈二	2022-1704班	文学系
  */
  ```

- CTE 优化后：

  ```sql
  -- CTE 优化
  WITH student_info -- AS (SELECT ...) 子查询的虚拟结果表会注入到 student_info 变量中
  AS (
  	SELECT s.student_name, d_c.class_name, d_c.dept_id
  	FROM student s 
  	LEFT JOIN d_class d_c ON d_c.class_id = s.class_id
  	WHERE s.age > 20
  )
  -- 主查询
  SELECT info.student_name AS "学生姓名", info.class_name AS "所在班级", d.dept_name AS "所属系别" 
  FROM student_info info 
  -- info 里存储的就是子查询结果：所有系别中，年龄超过 20 岁的学生信息（姓名、所在班级名称、班级所属系别ID）
  JOIN department d
  ON d.dept_id = info.dept_id;
  /*
   学生姓名 | 所在班级 |  所属系别  
  --------+----------+----------+ 
    李四	2022-1701班	理工系
    孙七	2022-1702班	理工系
    吴九	2022-1703班	文学系
    陈二	2022-1704班	文学系
  */
  ```

## 级联定义

- 可以**在同一行（`;` 之前）代码**中**定义多个 CTE 语句**，**后面的 CTE 语句** 可以直接**引用前面的 CTE 语句返回的临时变量**并操作。

### 基本语法

```sql
WITH 
-- CTE_1（本质上是一张虚拟结果表）
<临时变量名1> AS (
    -- 子查询1
    SELECT <查询结果> FROM ...
),
-- CTE_2
<临时变量名2> AS (
    -- 子查询2
    SELECT <查询结果> FROM <CTE_1 临时变量1(子查询结果)> -- （只能引用前面一个的临时变量）
),
-- 主查询
SELECT ... FROM <CTE_2 临时变量2(子查询结果)> -- （只能引用前面一个的临时变量）
```

### 示例

#### `INSERT` 同时插入 n 条新数据

- **`DO` 匿名块**：

  ```postgresql
  -- 同时插入3条数据
  DO $$
  DECLARE -- 定义变量
  	v_dept_id INTEGER; -- 当前新插入的系别ID
  	v_class_id INTEGER; -- 当前新插入的班级ID
  BEGIN
  	-- 插入一条系别数据
  	INSERT INTO department (dept_name) VALUES ('金融系') RETURNING dept_id INTO v_dept_id; 
  	-- 注入到 v_dept_id 变量中，给后续SQL代码引用
  	
  	-- 根据新插入的系别ID，插入 n 条班级数据
  	INSERT INTO d_class (class_id, class_name, dept_id) VALUES (1010, '2022-1701班', v_dept_id) RETURNING class_id INTO v_class_id;
  	 -- 注入到 v_class_id 变量中，给后续SQL代码引用
  	
  	-- 根据所新插入的班级ID，插入 n 条学生数据
  	INSERT INTO student (student_name, gender, age, class_id) VALUES ('王大志', '男', 22, v_class_id);
  	INSERT INTO student (student_name, gender, age, class_id) VALUES ('海龙', '男', 20, v_class_id);
  END $$;
  ```

- CTE 语法：

  ```postgresql
  WITH 
  -- 1. 插入系别，并返回新生成的 dept_id
  v_dept AS (
      INSERT INTO department (dept_name) 
      VALUES ('机械系') 
      RETURNING dept_id;
  ),
  -- 2. 传入系别 ID，插入班级，并返回新生成的 class_id
  v_class AS (
  		-- v_dept 变量返回的是一张虚拟结果表，可以把它的内容通过 SELECT 提取出来，并通过 INTO 插入到 d_class 表中
      INSERT INTO d_class (class_id, class_name, dept_id) SELECT 1011, '2022-1705班', dept_id FROM v_dept
      RETURNING class_id
  )
  -- 3. 最终主语句：传入班级 ID，批量插入多条学生数据
  INSERT INTO student (student_name, gender, age, class_id) 
  SELECT '王晓慧', '女', 22, class_id FROM v_class
  UNION ALL
  SELECT '利奇马', '男', 20, class_id FROM v_class;
  ```

## 避坑指南与性能考量

1. **生命周期：** CTE 的生命周期**仅限于当前的单条 SQL 语句**。主查询结束后，CTE 就会被销毁，无法在下一条独立的 SQL 中引用（如果需要跨语句复用，请考虑临时表 `#TempTable` 或视图 `View`）。
2. **性能物化（Materialization）：**
   - 在早期的数据库版本中，CTE 往往会被**物化（Materialized）**，即数据库会把 CTE 的结果先算出来存在内存/磁盘中，这有时会导致无法利用索引，性能不如直接内联的子查询。
   - 现代数据库（如 **PostgreSQL 12+**, MySQL 8.0+, SQL Server 等）的优化器已经非常智能。它们通常会把 CTE 自动“展开”并与主查询一起优化（视为 Inline）。
   - *注：在 PostgreSQL 中，如果你确定不需要物化，或者想强制物化，可以使用 `WITH cte_name AS NOT MATERIALIZED (...)` 或 `AS MATERIALIZED` 来手动控制。*
3. **不能滥用递归：** 编写递归 CTE 时，务必确保有明确的终止条件（通常在 `JOIN` 条件中自然终止），否则一旦数据存在环路（A 的经理是 B，B 的经理是 A），会导致**死循环**，直到内存溢出或达到系统最大递归深度限制。

# View 视图

## 基本概念

​	**`VIEW` 视图**本质上就是一张 **“虚拟表”**，它**不存储实际数据，只存储一条 SQL 语句**；当**使用 `SELECT` 查询视图**时，会**自动跑一遍背后的 SQL 语句**。

核心特点：

1. **`VIEW` 视图是一张 “虚拟表”**，是**从一个或多个 Table 基本表（或 `VIEW` 视图）**的**源数据**中**实时**导出来的**结果集**

2. 数据库**只存放 `VIEW` 视图的定义结构**，而不存储视图中查询的对应数据

3. `VIEW` 视图中**涉及到的 Table 原表发生变化**时，**`VIEW` 视图查询的结果**也会**随之实时更新**

4. 可以**手动定义 `VIEW` 视图查询的输出结果列**，用来**一一对应关联 `VIEW` 视图中 `SELECT` 子查询投影的列**；

   核心机制：**要么全部省略（完全继承自 Table 原表的列命名），要么全部指定（自定义投影结果列的命名）**

核心优势：

1. **代码复用**：**`VIEW` 视图会作为一个对象持久化存储**在数据库中，**想用**时**直接 `SELECT` 查询**即可；可以把一些经常要用的复杂 SQL 语句（复杂查询、报表统计...）封装成一个 `VIEW` 视图，**随时取用**。

2. **权限控制、隐藏 Table 表敏感字段**：

   ​	如果不想让普通用户直接看到一些 Tabie 表的敏感字段数据，可以通过将 Table 表中的敏感字段挑选出来，并将**整个 SQL 语句封装成一个 `VIEW` 视图授权给一些普通用户**，这样**普通用户只能查看某个表上的 `VIEW` 视图数据，不能直接查看原表的敏感信息**。

3. **动态更改**：支持使用 **`WITH CHECK OPTION`** 命令**对 `VIEW` 视图中已定义的查询条件进行实时更新**，提高灵活性

## `CREATE VIEW` 创建视图

*注：强烈建议 `VIEW` 视图是一个**只读**的，别使用 `UPDATE`、`INSERT`、`DELETE` 去更新视图的查询数据，这会造成很多麻烦。*

### 基本语法

注意点：**`VIEW` 视图与 `TABLE` 表是同一层级的，属于某个 Schema 模式下归属管理**。

```postgresql
-- 授权视图的 SELECT 查看权限给某个用户
GRANT SELECT ON <模式名>.<视图名> TO <用户名>;
```

#### `VIEW` **简单视图**

注：建立加上 **`IF NOT EXISTS`（如果不存在）**;

```postgresql
CREATE VIEW [IF NOT EXISTS] <模式名>.<视图名> AS (<SELECT 子查询语句...>); -- 建立使用 () 包裹子查询，看起来结构更清晰
```

##### 示例

假设有如下表：

```postgresql
SET search_path TO company;

CREATE TABLE emp (
    emp_id INT PRIMARY KEY, -- 员工ID
    emp_name VARCHAR(50) NOT NULL, -- 员工姓名
    salary DECIMAL(10, 2), -- 薪资
    dept_id INT --所属部门ID
);
```

- **简单视图**：

  ```postgresql
  -- 展示原表的所有结果列
  CREATE VIEW emp_view AS (SELECT * FROM emp);
  
  -- 查看视图
  SELECT * FROM emp_view;
  /*
   emp_id | emp_name |  salary  | dept_id
  --------+----------+----------+---------
     1001 | 张三     | 12000.00 |      10
     1002 | 李四     | 15000.00 |      10
     1003 | 王五     |  8000.00 |      20
     1004 | 赵六     |  8000.00 |      20
     1005 | 钱七     |  9500.00 |      30
     1006 | 孙八     | 11000.00 |
  */
  ```

  ```postgresql
  -- 只展示原表的 n 个结果列
  CREATE VIEW emp_view_as_name AS (SELECT emp_name, salary FROM emp WHERE emp_name = '张三');
  
  -- 查看视图
  SELECT * FROM emp_view_as_name;
  /*
   emp_name |  salary
  ----------+----------
   张三     | 12000.00
  */
  ```

- **基于视图创建一个视图**：

  ```postgresql
  -- 展示原视图的所有结果列
  CREATE VIEW emp_view_exc AS (SELECT * FROM emp_view);
  
  -- 查看视图
  SELECT * FROM company.emp_view;
  /*
   emp_id | emp_name |  salary  | dept_id
  --------+----------+----------+---------
     1001 | 张三     | 12000.00 |      10
     1002 | 李四     | 15000.00 |      10
     1003 | 王五     |  8000.00 |      20
     1004 | 赵六     |  8000.00 |      20
     1005 | 钱七     |  9500.00 |      30
     1006 | 孙八     | 11000.00 |
  */
  ```

  ```postgresql
  -- 只展示原视图的 n 个结果列
  CREATE VIEW emp_view_exc_name AS SELECT * FROM emp_view WHERE emp_name = '张三';
  
  -- 查看视图
  SELECT * FROM company.emp_view_exc_name;
  /*
   emp_id | emp_name |  salary  | dept_id
  --------+----------+----------+---------
     1001 | 张三     | 12000.00 |      10
  */
  ```

#### `VIEW(...)` 自定义结果列的视图

```postgresql
CREATE VIEW <模式名>.<视图名>(<结果列1>, <结果列2>,...) AS (<SELECT [* | <列1>, <列2>,...] FROM <表名>>);
```

核心要点：

- **声明并限制 `VIEW` 视图执行查询后返回的结果列**，**必须与 `AS SELECT` 子查询返回的结果列投影顺序、个数一一对应**

  相当于是**给 `AS SELECT` 子查询返回的结果列**起了一个**别名**

<img src="media/VIEW 视图-限制结果列.png" style="zoom:50%;" />

> [!IMPORTANT]
>
> ⚠️注意点：
>
> - 如果 **`VIEW` 视图的结果列别名个数**（**少于**） **`AS SELECT` 投影返回的结果列个数**，则**视图结果中会显示原表的原结果列名**
>
>   
>
> - 如果 **`VIEW` 视图的结果列别名个数**（**多于**） **`AS SELECT` 投影返回的结果列个数**，则会**报错**，因为**原表并没有那么多列**
>
>   <img src="media/VIEW 视图-限制结果列-视图个数多于子查询列个数.png" style="zoom:50%;" />

##### 示例

假设有如下表：

```postgresql
SET search_path TO company;

CREATE TABLE emp (
    emp_id INT PRIMARY KEY, -- 员工ID
    emp_name VARCHAR(50) NOT NULL, -- 员工姓名
    salary DECIMAL(10, 2), -- 薪资
    dept_id INT --所属部门ID
);
```

- **全部起别名**：

  ```postgresql
  CREATE VIEW view_emp_all_column (v_emp_id, v_emp_name, v_salary, v_dept_id) AS (SELECT * FROM emp);
  
  -- 查看视图
  SELECT * FROM company.view_emp_all_column;
  /*
   v_emp_id | v_emp_name | v_salary | v_dept_id
  ----------+------------+----------+-----------
       1001 | 张三       | 12000.00 |        10
       1002 | 李四       | 15000.00 |        10
       1003 | 王五       |  8000.00 |        20
       1004 | 赵六       |  8000.00 |        20
       1005 | 钱七       |  9500.00 |        30
       1006 | 孙八       | 11000.00 |
  */
  ```

- **限制视图结果列个数 < 原表的投影结果列个数**：

  ```postgresql
  -- 限制视图结果列个数 < 原表的投影结果列个数
  CREATE VIEW view_emp_part_column (v_emp_id, v_emp_name) AS (SELECT * FROM emp);
  
  -- 查看视图
  SELECT * FROM view_emp_part_column;
  /*
  (视图自定义的别名结果列)  - (原表结果列名)
   v_emp_id | v_emp_name |  salary  | dept_id
  ----------+------------+----------+---------
       1001 | 张三       | 12000.00 |      10
       1002 | 李四       | 15000.00 |      10
       1003 | 王五       |  8000.00 |      20
       1004 | 赵六       |  8000.00 |      20
       1005 | 钱七       |  9500.00 |      30
       1006 | 孙八       | 11000.00 |
  */
  
  
  -- 限制视图结果列个数 < 原表的投影结果列个数
  CREATE VIEW view_emp_part_column3 (v_emp_name) AS (SELECT emp_name, salary FROM emp);
  
  
  -- 查看视图
  SELECT * FROM company.view_emp_part_column3;
  /*
   v_emp_name |  salary
  ------------+----------
   张三       | 12000.00
   李四       | 15000.00
   王五       |  8000.00
   赵六       |  8000.00
   钱七       |  9500.00
   孙八       | 11000.00
  */
  ```

- ...

## `SELECT` 查询视图

可以像表一样操作视图的查询结果：

```postgresql
SELECT <* | 指定结果列,...> FROM <视图名> [WHERE <条件>] [GROUP BY <列>];
```

示例：

```postgresql
-- 查看视图
SELECT * FROM emp_view;

SELECT emp_name, salary FROM emp_view;

SELECT * FROM emp_view WHERE emp_name = '张三';

SELECT dept_id, COUNT(*) FROM emp GROUP BY dept_id;
```

## `ALTER / REPLACE` 更新视图

如果需要修改视图的内部查询逻辑，通常有两种方式：

### `CREATE OR REPLACE` 存在则覆盖

核心作用：**如果 `VIEW` 视图已存在则覆盖，不存在则创建**。

```postgresql
CREATE OR REPLACE VIEW <模式名>.<视图名> AS (<SELECT 子查询语句...>);
```

> [!IMPORTANT]
>
> 在 PostgreSQL 等数据库中，`CREATE OR REPLACE VIEW` 的行为有严格的**底层限制**：
>
> - 它的“替换”功能，**只能在现有视图的末尾追加（Append）新列**
> - 它**绝对不允许**删除（Drop）已经存在的列，也**不允许改变现有列的数据类型或顺序**
>
> 简单来说，**`CREATE OR REPLACE` 创建的新视图结构，必须是基于原视图的字段结构之上进行追加字段展示，不可减少。**

示例：

```postgresql
SET search_path TO company;

-- 原视图
CREATE VIEW view_emp AS (SELECT emp_id, emp_name FROM emp WHERE emp_name = '张三');

SELECT * FROM view_emp;
/*
 emp_id | emp_name
--------+----------
   1001 | 张三
*/

-- 基于原始图的结构之上覆盖，只能追加字段展示，不可减少
CREATE OR REPLACE VIEW view_emp AS (SELECT emp_id, emp_name, salary, dept_id FROM emp);

SELECT * FROM view_emp;
/*
 emp_id | emp_name |  salary  | dept_id
--------+----------+----------+---------
   1001 | 张三     | 12000.00 |      10
   1002 | 李四     | 15000.00 |      10
   1003 | 王五     |  8000.00 |      20
   1004 | 赵六     |  8000.00 |      20
   1005 | 钱七     |  9500.00 |      30
   1006 | 孙八     | 11000.00 |
*/
```

### `ALTER VIEW` 修改视图的元属性

核心作用：**明确强制修改（覆盖） `VIEW` 视图的内部查询逻辑**。

```postgresql
ALTER VIEW <模式名>.<视图名> [RENAME TO | OWNER TO | SET SCHEMA] <视图名 | 用户名 | 模式名>;
```

示例：

```postgresql
-- 1. 给视图重命名
ALTER VIEW view_emp RENAME TO v_employee_all;

-- 2. 修改视图的所有者 (Owner)
ALTER VIEW view_emp OWNER TO admin_user;

-- 3. 改变视图所属的 Schema
ALTER VIEW view_emp SET SCHEMA public;
```

#### 不同数据库的差异

| **数据库**     | **支持 ALTER VIEW view_name AS SELECT ... 吗？** |
| -------------- | ------------------------------------------------ |
| **MySQL**      | **支持**（可以用它来改内部的 SQL 语句）          |
| **SQL Server** | **支持**（标准的修改视图语句做法）               |
| **PostgreSQL** | **绝对不支持** ❌（会直接报 `syntax error`）      |
| **Oracle**     | **绝对不支持** ❌（只能用来重新编译或改属性）     |

## `DROP` 删除视图

注：建立加上 **`IF EXISTS`（如果存在）**;

```postgresql
DROP VIEW [IF EXISTS] <模式名>.<视图名>;
```

示例：
```postgresql
DROP VIEW IF EXISTS emp_view; -- 如果 emp_view 视图存在就删除它
```

# `VIEW` 视图 vs CTE

| **特性**     | **CTE (WITH)**                   | **视图 (View)**                                  |
| ------------ | -------------------------------- | ------------------------------------------------ |
| **生命周期** | **只在当前这条 SQL 里有效**      | **永久保存在数据库里**                           |
| **类比**     | 局部变量                         | 全局公共函数                                     |
| **适用场景** | 仅仅为了让这一行复杂长查询变好看 | 经常要用的复杂查询、报表统计                     |
| **权限控制** | 无                               | **非常强大**（可以给用户看视图，但不给他看原表） |

