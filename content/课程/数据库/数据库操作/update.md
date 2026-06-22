## UPDATE 语句

`UPDATE` 语句用于修改关系表中已存在的数据行（记录）。

### 基本语法

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

### 参数说明

- `table_name`：要更新的表名。
- `SET column = value`：指定要更新的列及其新值。可以同时更新多个列，用逗号分隔。
- `WHERE condition`：指定更新的条件。只有满足条件的行才会被更新。

### 注意事项

- 如果省略 `WHERE` 子句，将**更新表中的所有行**，请务必谨慎操作。

### 示例

```sql
-- 将名为 '菜鸟教程' 的网站记录的 alexa 排名更新为 5000，国家更新为 'USA'
UPDATE Websites 
SET alexa = '5000', country = 'USA' 
WHERE name = '菜鸟教程';

-- 给所有工资低于 70000 的讲师涨薪 5%
UPDATE instructor
SET salary = salary * 1.05
WHERE salary < 70000;
```
