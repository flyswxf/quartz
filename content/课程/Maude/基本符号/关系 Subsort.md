### 基本概念
Subsort 关系类似于集合的子集关系，在 Sort 之间建立偏序关系。

### 单个 Subsort 声明
```maude
subsort Sort-1 < Sort-2 .
```
- 含义: Sort-1这一类型的所有元素属于Sort-2这一类型

**示例：**
```maude
subsort Zero < Nat .
subsort NzNat < Nat .
```

### 多个 Subsort 声明
```maude
subsorts Sort-1 ... Sort-j < ... < Sort-k ... Sort-l .
```

**示例：**
```maude
subsorts Zero NzNat < Nat .
subsorts NzNat < NzInt Nat < Int .
```

### 偏序要求
- Subsort 声明必须定义偏序关系
- **禁止循环**：如果 A < B，则不能有 B < A
- 偏序将 Sort 集合划分为连通分量

## 连通分量示例

### 数值类型层次结构
```
        Int
       /   \
     Nat   NzInt
    /  \   /
 Zero  NzNat
```

### 逻辑类型层次结构
```
 Prop
  |
 Bool
```

### 完整示例
```maude
sorts Zero NzNat Nat NzInt Int Bool Prop .

subsorts Zero NzNat < Nat .
subsorts NzNat < NzInt .
subsorts Nat < Int .
subsorts Bool < Prop .
```


### 常见错误
1. **循环依赖**：
   ```maude
   subsort A < B .
   subsort B < A .  // 错误：形成循环
   ```