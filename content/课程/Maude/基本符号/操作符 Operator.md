## 概述
在 Maude 模块中，操作符用于定义对数据类型的操作。操作符声明指定了操作符的名称、参数类型（定义域）和返回类型（值域）。

## 基本语法结构

### 标准语法
```maude
op OpName : Sort-1 ... Sort-k -> Sort [OperatorAttributes] .
```

**语法组成部分：**
- `op`：关键字
- `OpName`：操作符名称
- `:`：分隔符
- `Sort-1 ... Sort-k`：**定义域**（参数类型列表，操作符的元数/arity）
- `->`：箭头分隔符
- `Sort`：**值域**（返回类型，操作符的余元数/coarity）
- `[OperatorAttributes]`：可选的操作符属性
- `.`：结束符（前后必须有空格）

### 基本示例

^eff35b

```maude
op zero : -> Zero .                    // 常量（无参数）
op s_ : Nat -> NzNat .                 // 一元操作符
op sd : Nat Nat -> Nat .               // 二元前缀操作符
ops _+_ _*_ : Nat Nat -> Nat .          // 多个操作符同时声明
```

## 操作符形式分类

### 1. 常量 (Constants)
参数列表为空的操作符称为常量。

```maude
op zero : -> Zero .
op true : -> Bool .
op false : -> Bool .
```

### 2. 前缀形式 (Prefix Form)

^d1eec5

操作符名称中**不包含下划线**的操作符为前缀形式。

```maude
op sd : Nat Nat -> Nat .               // 前缀二元操作符
op not : Bool -> Bool .                // 前缀一元操作符
```

**使用方式：**
```maude
sd(3, 5)        // 前缀调用
not(true)       // 前缀调用
```

### 3. 混合形式 (Mixfix Form)
操作符名称中**包含下划线**的操作符为混合形式。

#### 重要规则
- 下划线数量必须等于参数数量
- 第 n 个下划线表示第 n 个参数的位置
- 常量名称不能包含下划线

#### 混合形式示例
```maude
op s_ : Nat -> NzNat .                 // 后缀一元操作符
op _+_ : Nat Nat -> Nat .              // 中缀二元操作符
op _*_ : Nat Nat -> Nat .              // 中缀二元操作符
op if_then_else_fi : Bool Nat Nat -> Nat . // 三元混合操作符
```

**使用方式：**
```maude
s 5             // 后缀调用：s_(5)
3 + 4           // 中缀调用：_+_(3, 4)
if true then 1 else 0    // 混合调用
```

## 特殊语法形式

### 空语法 (Empty Syntax)
操作符可以声明为只有下划线，没有其他字符。

```maude
sort NatSeq .
subsort Nat < NatSeq .
op __ : NatSeq NatSeq -> NatSeq [assoc] .
```

**使用效果：**
```maude
zero (s zero) (s s zero)    // 序列连接，无显式操作符
```

### 多操作符声明
使用 `ops` 关键字可以同时声明多个具有相同元数和余元数的操作符。

```maude
ops _+_ _*_ _-_ : Nat Nat -> Nat .
ops _and_ _or_ : Bool Bool -> Bool .
```

## 完整示例

### NUMBERS 模块操作符声明
```maude
fmod NUMBERS is
  sorts Zero NzNat Nat .
  
  subsorts Zero NzNat < Nat .
  
  op zero : -> Zero .                   // 常量
  op s_ : Nat -> NzNat .               // 后缀一元操作符
  op sd : Nat Nat -> Nat .             // 前缀二元操作符
  ops _+_ _*_ : Nat Nat -> Nat .       // 中缀二元操作符
  
endfm
```

### 使用示例
```maude
zero                    // 常量
s zero                  // 后缀：s_(zero)
s s zero               // 嵌套：s_(s_(zero))
sd(s zero, zero)       // 前缀调用
(s zero) + (s s zero)  // 中缀调用
```