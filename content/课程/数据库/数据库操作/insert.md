## INSERT 语句

`INSERT` 语句用于向关系表中插入新的数据行（记录）。

### 基本语法

#### 1. 插入单条数据（直接传参）

默认按照表结构的列顺序插入数据：

```sql
INSERT INTO table_name
VALUES (value1, value2, value3, ...);
```

#### 2. 插入单条数据（指定列名传参）

只为指定的列插入数据，未指定的列将填入默认值或 `NULL`：

```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

#### 3. 插入多条数据（通过 SELECT 语句）

通过 `SELECT` 语句查询出的结果集，直接插入到目标表中，代替 `VALUES()` 的位置：

```sql
INSERT INTO table_name (column1, column2, ...)
SELECT column1, column2, ...
FROM another_table
WHERE condition;
```

### 示例

```sql
-- 方法1：直接按列顺序插入
INSERT INTO course 
VALUES ('CS-001', 'Weekly Seminar', NULL, 0);

-- 方法2：带列名传参
INSERT INTO course (course_id, title, credits) 
VALUES ('CS-001', 'Weekly Seminar', 0);

-- 方法3：使用 SELECT 语句批量插入
INSERT INTO instructor (ID, name, dept_name, salary)
SELECT ID, name, dept_name, 18000
FROM student
WHERE tot_cred > 100;
```

#### 常量和列名放在一起

在 `SELECT` 语句中可以直接指定常量，这样插入时会为每一行固定填充该常量：
![[assets/插入语句示例.png]]
