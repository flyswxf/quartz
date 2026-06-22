## WITH 语句（公共表表达式 CTE）

`WITH` 语句用于定义一个或多个临时的结果集，这些结果集被称为公共表表达式（Common Table Expressions, CTE）。这些临时表在接下来的查询中可以被多次引用，使得复杂的 SQL 语句更加简洁易读。

### 基本语法

```sql
WITH temp_table_name AS (
    SELECT ...
),
temp_table_name_2 AS (
    SELECT ...
)
SELECT ...
FROM temp_table_name, temp_table_name_2
WHERE ...;
```

### 作用与特点

- 相当于定义了一个临时的关系表。数据库执行带 `WITH` 的语句时，会将临时表名替换为对应的 `SELECT` 查询结果。
- 提高了复杂查询的可读性和可维护性。
- 常用于替代复杂的嵌套子查询。

### 示例

```sql
-- 找出总工资最高的系
WITH dept_total AS (
    SELECT dept_name, SUM(salary) AS value
    FROM instructor
    GROUP BY dept_name
),
dept_total_avg AS (
    SELECT AVG(value) AS value
    FROM dept_total
)
SELECT dept_name
FROM dept_total, dept_total_avg
WHERE dept_total.value >= dept_total_avg.value;
```
