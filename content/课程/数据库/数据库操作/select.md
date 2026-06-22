## SELECT 语句

`SELECT` 语句用于从数据库中查询数据，并返回一个结果集（Relation）。

### 基本语法与执行顺序

一个完整的 `SELECT` 查询执行过程如下：

1. `FROM`：进行笛卡尔积或连接，获得临时表。
2. `WHERE`：对每一行施加约束，选出满足条件的项。
3. `GROUP BY`：分组，如果没有 `GROUP BY` 语句，则视为整体为1组。
4. `HAVING`：对每一组施加约束，选出满足条件的组。
5. `SELECT`：选择要返回的列，计算投影。
6. `ORDER BY`：对最终结果进行排序。

### 关键字说明

- `FROM r1, r2, ...`：对多个关系表做笛卡尔积。如果不同表中存在同名字段，需要用 `r1.id`, `r2.id` 区分。
- `WHERE condition`：选择符合条件的项。
	- 等值匹配：`WHERE course.id = takes.id`
	- 子查询匹配：`WHERE course.id IN (SELECT ID FROM teaches WHERE year = 2024)`
	- 多列匹配：`WHERE (course_id, sec_id, semester, year) IN (SELECT course_id, sec_id, semester, year FROM teaches WHERE teaches.ID = '10101')`
	- 模糊匹配：`WHERE LOWER(title) LIKE '%advanced%'`
- `AS alias`：重命名列或表。
	- 列重命名：`SELECT AVG(salary) AS avg_salary`（[[聚合函数]]没有默认列名，通常需要重命名）。
	- 表重命名：`SELECT A FROM R1 AS R`。
- `ORDER BY column`：对结果进行排序，默认递增（`ASC`）。
	- 递减排序使用 `DESC`：`ORDER BY salary DESC, name ASC`。
- `GROUP BY column`：对结果按指定列进行分组。
	- 常配合[[聚合函数]]使用
- `HAVING condition`：选择满足条件的组，必须与 `GROUP BY` 配合使用。


- **去重操作**：`SELECT` 默认**不去重**，需要去重时添加 `DISTINCT` 关键字，例如 `SELECT DISTINCT name FROM instructor`。详见 [[去重]]。

### 常见 SELECT 查询模式

#### 1. 连接（Join）的表示
显式指定连接条件来替代纯笛卡尔积（详见 [[join]]）：
```sql
SELECT A FROM r1, r2 WHERE r1.id = r2.id;
```

#### 2. 自身比较
将表与自身做笛卡尔积并比较：
```sql
SELECT A FROM R1 AS S, R1 AS T WHERE S.SCORE > T.SCORE;
```

#### 3. 关系代数中的除法（包含全部）
```sql
NOT EXISTS (SELECT * FROM ...) EXCEPT (SELECT ... FROM ... WHERE ...)
```

#### 4. 一个都没有（空集判断）
```sql
(SELECT ... FROM ...) EXCEPT (SELECT ... FROM ... WHERE ...)
```

#### 5. 最值查询
```sql
SELECT ... FROM r WHERE A = (SELECT MIN(A) FROM r);
```
![[assets/最小薪资公司查询.png]]
