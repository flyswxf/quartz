包含多条执行语句

### 例子
事务A:
1. read(A)
2. A := A – 50
3. write(A)
4. read(B)
5. B := B + 50
6. write(B)

### 性质
- 事务需要满足[[ACID Properties]]
- 当事务执行完毕时, 会**commit**
- 当事务执行出错时, 会**abort**
