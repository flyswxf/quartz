## 无条件等式

等式告诉Maude应该如何解析输入语句

### 基本语法

```maude
eq Term-1 = Term-2 [StatementAttributes] .
```

- 使用关键字 `eq`
- 左右两项必须具有相同的 kind
- 可选的语句属性用方括号包围
- 以` .`结束

## 条件等式/条件成员关系

只有当条件满足时, 才能解析
### 基本语法

**条件等式**：
```maude
ceq Term-1 = Term-2 
if EqCondition-1 /\ ... /\ EqCondition-k 
[StatementAttributes] .
```

**条件成员关系**：
```maude
cmb Term : Sort 
if EqCondition-1 /\ ... /\ EqCondition-k 
[StatementAttributes] .
```
- 条件使用[[逻辑符号]]表示