## Hoare Triple (霍尔三元组)

需要能够表达以下陈述：“如果程序片段 $P$ 的执行开始于满足 $\Phi$ 的状态，那么 $P$ 的执行将结束于满足 $\Psi$ 的状态。” 将其表示为：

$$
(\!| \Phi |\!) P (\!| \Psi |\!)
$$

称这种结构为 **霍尔三元组 (Hoare triple)**。
*   $\Phi$ 被称为 **前置条件 (precondition)**。
*   $\Psi$ 被称为 **后置条件 (postcondition)**。

### 例子
假设程序 $P$ 的规范是“**计算一个平方小于 $x$ 的数**”。那么，以下断言应该成立：

$$
(\!| x > 0 |\!) P (\!| y \cdot y < x |\!)
$$

**这意味着**：如果从 $x > 0$ 的状态开始执行，那么 $P$ 的执行将结束于 $y^2 < x$ 的状态。

如果执行开始时 $x \le 0$ 会发生什么？不知道！

## Partial vs Total Correctness (部分正确性 vs 完全正确性)

*   **部分正确性 (Partial correctness)**：**不要求**程序终止。
*   **完全正确性 (Total correctness)**：**要求**程序终止。

### 定义 (部分正确性)
如果对于所有满足 $\Phi$ 的状态，由 $P$ 执行产生的状态都满足后置条件 $\Psi$，**前提是 $P$ 确实终止**，就说三元组 $(\!| \Phi |\!) P (\!| \Psi |\!)$ 满足部分正确性。
在这种情况下，记作：

$$
\models_{par} (\!| \Phi |\!) P (\!| \Psi |\!)
$$

### 定义 (完全正确性)
如果对于所有执行 $P$ 且满足前置条件 $\Phi$ 的状态，$P$ **保证终止**，并且由 $P$ 执行产生的状态满足后置条件 $\Psi$，就说三元组 $(\!| \Phi |\!) P (\!| \Psi |\!)$ 满足完全正确性。
在这种情况下，记作：

$$
\models_{tot} (\!| \Phi |\!) P (\!| \Psi |\!)
$$

## Program Variables and Logical Variables (程序变量与逻辑变量)

考虑以下例子：

**Fac2:**
```c
y = 1;
while (x != 0) {
    y = y * x;
    x = x - 1;
}
```

**Sum:**
```c
z = 0;
while (x > 0) {
    z = z + x;
    x = x - 1;
}
```

$y$ 和 $z$ 的值是 $x$ 的**初始**值的函数。该值在程序结束时不再作为程序变量可用。
引入逻辑变量来处理这种情况。

$$
\models_{tot} (\!| x = x_0 \land x \ge 0 |\!) \text{ Fac2 } (\!| y = x_0! |\!)
$$

$$
\models_{tot} (\!| x = x_0 \land x > 0 |\!) \text{ stm } \left(\!| z = \frac{x_0(x_0+1)}{2} |\!\right)
$$

## Proof Calculus for Partial Correctness (部分正确性的证明演算)



*   **Composition (顺序组合)**
    $$
    \frac{(\!| \phi |\!) C_1 (\!| \eta |\!) \quad (\!| \eta |\!) C_2 (\!| \psi |\!)}{(\!| \phi |\!) C_1; C_2 (\!| \psi |\!)}
    $$
    **意义**：如果要证明程序 $C_1; C_2$ 从前置条件 $\phi$ 执行后满足后置条件 $\psi$，需要找到一个中间断言 $\eta$。$C_1$ 将状态从 $\phi$ 转换到 $\eta$，而 $C_2$ 将状态从 $\eta$ 转换到 $\psi$。

*   **Assignment (赋值)**
    $$
    \overline{(\!| \psi[E/x] |\!) x = E (\!| \psi |\!)}
    $$
    **意义**：赋值公理。为了使赋值语句 $x = E$ 执行后满足 $\psi$，其前置条件必须是 $\psi$ 中所有 $x$ 出现的地方都被表达式 $E$ 替换后的形式。这是逆向推理的核心。

*   **Implied (Consequence) (蕴含/推论)**
    $$
    \frac{\vdash \phi' \rightarrow \phi \quad (\!| \phi |\!) C (\!| \psi |\!) \quad \vdash \psi \rightarrow \psi'}{(\!| \phi' |\!) C (\!| \psi' |\!)}
    $$
    **意义**：允许加强前置条件（$\phi' \rightarrow \phi$）或减弱后置条件（$\psi \rightarrow \psi'$）。这在逻辑推导和实际程序验证之间架起了桥梁，使得我们可以使用数学逻辑来调整断言以匹配证明规则的要求。

*   **If-statement (条件语句)**
    $$
    \frac{(\!| \phi \land B |\!) C_1 (\!| \psi |\!) \quad (\!| \phi \land \neg B |\!) C_2 (\!| \psi |\!)}{(\!| \phi |\!) \text{if } B \{C_1\} \text{ else } \{C_2\} (\!| \psi |\!)}
    $$
    **意义**：要证明条件语句正确，需要分别证明两个分支的正确性。在 `then` 分支中，假设条件 $B$ 为真；在 `else` 分支中，假设条件 $B$ 为假。两个分支执行完后都必须满足相同的后置条件 $\psi$。

*   **Partial-while (While 循环)**
    $$
    \frac{(\!| \psi \land B |\!) C (\!| \psi |\!)}{(\!| \psi |\!) \text{while } B \{C\} (\!| \psi \land \neg B |\!)}
    $$
    - 循环规则依赖于**循环不变量 (loop invariant)** $\psi$。
	    - $\psi$ 在循环体 $C$ 执行前成立 (通过之前的代码证明得到)
	    - 进入循环之后, $\psi$ 成立, 且循环条件 $B$ 为真 (invariant hyp $\land$ guard)
	    - 循环最后,  $\psi$ 仍然成立 (通过循环中的代码证明得到)
	    - 循环结束后，不变量 $\psi$ 仍然成立，且循环条件 $B$ 为假 (partial-while)

*   **Total-while (Total Correctness)**
    $$
    \frac{(\!| \eta \land B \land 0 \le E = E_0 |\!) C (\!| \eta \land 0 \le E < E_0 |\!)}{(\!| \eta \land 0 \le E |\!) \text{while } B \{C\} (\!| \eta \land \neg B |\!)}
    $$
    **意义**：完全正确性不仅要求循环不变量 $\eta$ 保持成立，还要求证明循环会终止。这通过引入一个**变体 (variant)** 表达式 $E$ 来实现。
    - $E$ 必须是非负整数 ($0 \le E$)。
    - 每次循环迭代后，$E$ 的值必须严格减小 ($E < E_0$)。
    - 因为 $E$ 是非负整数且每次递减，循环必然在有限步内终止。

## Proof Tableaux (证明表格)

顺序组合规则表明了一种更方便的展示程序逻辑证明的方法：**证明表格 (proof tableaux)**。可以将核心编程语言的任何程序视为一个序列。

对应的表格：

```text
       (| Φ0 |)
C1;
       (| Φ1 |)    理由 (justification)
C2;
       (| Φ2 |)    理由 (justification)
...
       (| Φn-1 |)  理由 (justification)
Cn
       (| Φn |)    理由 (justification)
```

每一个转换
$$
(\!| \Phi_i |\!) C_{i+1} (\!| \Phi_{i+1} |\!)
$$
都诉诸于其中一个证明规则。

## Examples (示例)

作业例子见[[王宇飞-10235101413-作业7]]

### Assignment (赋值语句)

**示例 1**: 证明 $\vdash_{par} (\!| y = 5 |\!) x = y + 1 (\!| x = 6 |\!)$

```text
       (| y = 5 |)
       (| y + 1 = 6 |)    Implied (由 y=5 推导 5+1=6)
x = y + 1;
       (| x = 6 |)        Assignment
```

**示例 2**: 证明 $\vdash_{par} (\!| y < 3 |\!) y = y + 1 (\!| y < 4 |\!)$

```text
       (| y < 3 |)
       (| y + 1 < 4 |)    Implied (由 y<3 推导 y+1 < 4)
y = y + 1;
       (| y < 4 |)        Assignment
```

### If Statement (条件语句)

**示例**:
```text
       (| T |)
       (| (x + 1 - 1 = 0 -> 1 = x + 1) /\ (~(x + 1 - 1 = 0) -> x + 1 = x + 1) |)   Implied
a = x + 1;
       (| (a - 1 = 0 -> 1 = x + 1) /\ (~(a - 1 = 0) -> a = x + 1) |)               Assignment
if (a - 1 == 0) {
       (| 1 = x + 1 |)
    y = 1;
       (| y = x + 1 |)    Assignment
} else {
       (| a = x + 1 |)
    y = a;
       (| y = x + 1 |)    Assignment
}
       (| y = x + 1 |)    If-Statement
```

### Loop (循环 - Partial Correctness)

**示例**: 阶乘计算
```text
       (| T |)
       (| 1 = 0! |)             Implied (1 等于 0 的阶乘)
y = 1;
       (| y = 0! |)             Assignment
z = 0;
       (| y = z! |)             Assignment
while (z != x) {
       (| y = z! /\ z != x |)   Invariant Hyp. /\ guard
       (| y * (z + 1) = (z + 1)! |) Implied
    z = z + 1;
       (| y * z = z! |)         Assignment
    y = y * z;
       (| y = z! |)             Assignment
}
       (| y = z! /\ ~(z != x) |) Partial-while
       (| y = x! |)             Implied
```

### Loop (循环 - Total Correctness)

**示例**: 阶乘计算 (证明终止性)

我们选择变体 $E = x - z$。

```text
       (| x >= 0 |)
       (| 1 = 0! /\ 0 <= x - 0 |)            Implied
y = 1;
       (| y = 0! /\ 0 <= x - 0 |)            Assignment
z = 0;
       (| y = z! /\ 0 <= x - z |)            Assignment
while (x != z) {
       (| y = z! /\ x != z /\ 0 <= x - z = E0 |)      Invariant Hyp. /\ guard
       (| y * (z + 1) = (z + 1)! /\ 0 <= x - (z + 1) < E0 |) Implied (变体 E = x-z 减小)
    z = z + 1;
       (| y * z = z! /\ 0 <= x - z < E0 |)            Assignment
    y = y * z;
       (| y = z! /\ 0 <= x - z < E0 |)                Assignment
}
       (| y = z! /\ x = z |)                 Total-while
       (| y = x! |)                          Implied
```
