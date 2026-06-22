规范和编程的基本单元

## 结构
```python
fmod/mod MODULE-NAME is
	--- 此处填入以下内容
endfm/endm
```
- [[模块引入|子模块导入]]
- [[数据类型 Sort]] 声明
- [[关系 Subsort]] 声明  
- [[操作符 Operator]] 声明
- [[变量 Variables]] 声明
- [[等式 Equation]] 声明
- [[规则 Rule]] 声明 (仅系统模块)

### 函数模块 (Functional Modules)
表示可计算的数据类型与函数的等式理论。用等式和成员关系做“值计算/归约”，保证规范形唯一（偏向确定性、函数式定义）。

```python
fmod NUMBERS is
  sorts Nat .
  op 0 : -> Nat .
  op s : Nat -> Nat .
  op _+_ : Nat Nat -> Nat .
  
  vars M N : Nat .
  eq 0 + N = N .
  eq s(M) + N = s(M + N) .
endfm
```

### 系统模块 (System Modules)
不仅包含确定性的[[等式 Equation|等式]]，还包含描述状态转换和并发行为的 **[[规则 Rule|重写规则]]**。

```python
mod VENDING-MACHINE is
  sorts Coin Item State .
  
  ops quarter dime : -> Coin .
  ops apple candy : -> Item .
  
  op [_] : Nat -> State .
  op insert_in_ : Coin State -> State .
  
  vars N : Nat .
  rl [insert quarter in [N]] => [N + 25] .
  rl [insert dime in [N]] => [N + 10] .
endm
```
