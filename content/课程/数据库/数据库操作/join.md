## JOIN 操作（连接查询）

`JOIN` 用于根据两个或多个表之间的相关列，将这些表中的行结合起来。它是关系型数据库中最核心的操作之一。

### 1. 内连接（INNER JOIN）

**返回两个表中满足连接条件的交集。**
如果表中有至少一个匹配，则返回行。这是最常见的一种连接方式（通常省略 `INNER`，直接写 `JOIN`）。

```sql
SELECT columns
FROM table1
INNER JOIN table2
ON table1.column = table2.column;
```
- **等价的旧式写法**（隐式连接）：`SELECT ... FROM table1, table2 WHERE table1.column = table2.column;`

### 2. 外连接（OUTER JOIN）

外连接不仅返回满足条件的匹配行，还会保留某一张（或两张）表中的不匹配行，并将缺失的一侧填充为 `NULL`。

#### 左外连接（LEFT JOIN / LEFT OUTER JOIN）
**返回左表中的所有记录，以及右表中匹配的记录。** 若右表中没有匹配项，结果的右侧部分显示为 `NULL`。

```sql
SELECT columns
FROM table1
LEFT JOIN table2
ON table1.column = table2.column;
```

#### 右外连接（RIGHT JOIN / RIGHT OUTER JOIN）
**返回右表中的所有记录，以及左表中匹配的记录。** 若左表中没有匹配项，结果的左侧部分显示为 `NULL`。

```sql
SELECT columns
FROM table1
RIGHT JOIN table2
ON table1.column = table2.column;
```

#### 全外连接（FULL JOIN / FULL OUTER JOIN）
**返回左表和右表中的所有记录。** 只要其中某个表存在匹配，就返回行。相当于左外连接和右外连接的并集（不匹配的一侧均补 `NULL`）。

```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2
ON table1.column = table2.column;
```

### 3. 自然连接（NATURAL JOIN）

**自动寻找两个表中同名的列，并基于这些同名列进行等值连接。**
- 连接后，结果集中同名列只保留一列，不会重复出现。
- 可以结合外连接使用，如 `NATURAL LEFT JOIN`。

```sql
SELECT *
FROM instructor
NATURAL JOIN teaches;
```
> **注意**：如果两个表中有多个同名列，`NATURAL JOIN` 会要求所有同名列的值都必须相等。如果有些同名列在语义上无关，会导致错误的查询结果，因此在实际开发中需谨慎使用。

### 4. 交叉连接（CROSS JOIN / 笛卡尔积）

**返回两个表的笛卡尔积。** 即左表的每一行与右表的每一行进行组合。
如果左表有 $m$ 行，右表有 $n$ 行，结果将有 $m \times n$ 行。

```sql
SELECT columns
FROM table1
CROSS JOIN table2;
```
- 等价的写法：`SELECT columns FROM table1, table2;`

### 5. 附加条件（USING 子句）

除了使用 `ON` 子句指定具体的连接条件，如果两个表需要连接的列同名，可以使用 `USING(column_name)` 来简化书写。这类似于 `NATURAL JOIN`，但允许你明确指定用哪些同名列进行连接，且连接后的同名列在结果中只保留一列。

```sql
SELECT *
FROM table1
JOIN table2 USING (id);
```
