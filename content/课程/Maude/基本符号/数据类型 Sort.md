## 概述
在 Maude 中，Sort（数据类型）是代数规范的基础，用于声明数据的类型。Sort 可以通过子类型关系形成偏序结构。

## Sort 声明语法

### 基本语法规则
⚠️ **重要**：Sort 声明必须以句号 `.` 结尾，且句号前后必须有空格，否则会导致解析错误。

### 单个 Sort 声明
```maude
sort SortName .
```

**示例：**
```maude
sort Zero .
sort NzNat .
sort Nat .
```

### 多个 Sort 声明
```maude
sorts Sort-1 Sort-2 ... Sort-k .
```

**示例：**
```maude
sorts Zero Nat NzNat .
```

### 关键字说明
- `sort` 和 `sorts` 是同义词
- 可以用 `sort` 声明多个类型，也可以用 `sorts` 声明单个类型（不推荐）

## Sort 命名规范

### 命名限制
以下标识符**不能**用作 Sort 名称：
- `<`, `->`, `~>`
- 包含字符 `:`, `.`, `[`, `]` 的标识符
- `{`, `}`, `,` 只能在结构化 Sort 名称中使用

### 推荐命名风格
- 首字母大写：`Nat`, `Bool`
- 复合名称采用驼峰命名：`NzNat`（非零自然数）

## 结构化 Sort 名称

### 语法定义
```bnf
Sort ::= sort_identifier | Sort { SortList }
SortList ::= Sort | SortList , Sort
```

### 合法的结构化 Sort 名称
```maude
a{X}
a{X, Y}  
a{b, c{d}}{e}     // a{b, c{d}}作为{e}的前缀,满足Sort { SortList }结构
a{(}
```

### 非法的结构化 Sort 名称
```maude
{X}              // 缺少 sort 标识符前缀
a(X, Y)          // 逗号不在大括号内
a{b, {d}}{e}     // {d} 缺少 sort 标识符前缀
a({)             // { 没有对应的 }
```

### 等价的单标识符形式
使用反引号可以将结构化 Sort 名称写成单标识符形式：
```maude
a{b, c{d}}{e}  ≡  a`{b`,c`{d`}`}`{e`}
```

### 参数化模块中的应用
```maude
List{X}      // 参数化列表类型，参数为 X
List{Nat}    // 自然数列表的实例化
```



## 注意事项

### 常见错误
1. **缺少句号**：
   ```maude
   sorts A B sort C .  // 错误：会声明 A, B, sort, C 四个类型
   ```

2. **句号前后缺少空格**：
   ```maude
   sort A.  // 可能导致解析问题
   ```

### 最佳实践
- 始终在句号前后添加空格
- 使用清晰的命名约定
- 合理设计类型层次结构
- 避免过深的继承层次