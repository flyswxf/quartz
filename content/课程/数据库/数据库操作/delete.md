## DELETE 语句

`DELETE` 语句用于从关系表中删除现有的数据行（记录）。

### 基本语法

```sql
DELETE FROM table_name
WHERE condition;
```

### 参数说明

- `FROM table_name`：指定要从中删除数据的目标表。
- `WHERE condition`：指定删除的条件。只有满足条件的行才会被删除。

### 注意事项

- 如果省略 `WHERE` 子句，将**删除表中的所有数据**，请谨慎操作。
- 建议在执行 `DELETE` 之前，先使用相同的 `WHERE` 条件执行 `SELECT` 语句，以确认将要删除的数据是否正确。

### 示例

```sql
-- 删除 course 表中 id 为 'CS-001' 的记录
DELETE FROM course
WHERE id = 'CS-001';

-- 删除所有工资低于 50000 的讲师记录
DELETE FROM instructor
WHERE salary < 50000;
```
