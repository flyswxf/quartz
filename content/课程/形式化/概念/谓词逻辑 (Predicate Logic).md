在谓词公式中存在两类“事物”：
1.  **对象 (Objects)**：如 Andy, Paul 等。函数符号也指向对象。这些由 **项 (terms)** 建模。
2.  **真值表达式 (Expressions)**：可以被赋予真值的表达式。这些由 **公式 (formulas)** 建模。

### 谓词词汇表
一个谓词词汇表包含三个集合：
*   **谓词符号 (Predicate symbols) $\mathcal{P}$**
*   **函数符号 (Function symbols) $\mathcal{F}$**
*   **常量 (Constants) $\mathcal{C}$**

> 每个谓词和函数符号都带有固定的 **元数 (arity)** (即参数的个数)。

#### 举例说明
*   **谓词符号 (Predicate Symbols)**: 表示对象之间的关系或属性，返回真值 (True/False)。
    *   例：`>(x, y)` (大于关系), `IsStudent(x)` (是否为学生), `=(x, y)` (相等关系)。
*   **函数符号 (Function Symbols)**: 表示对象到对象的映射，返回一个对象。
    *   例：`+(x, y)` (加法，返回数值对象), `MotherOf(x)` (x的母亲，返回人对象)。
*   **常量 (Constants)**: 特指域中的某个具体对象（**可视作0元函数**, 直接写到$\mathcal{F}$集合中）。
    *   例：`0`, `1`, `Alice`, `Bob`, `π`。

![[assets/谓词词汇表示例.png]]

## 翻译示例

句子："Every son of my father is my brother" (我父亲的每个儿子都是我的兄弟)

**建模**：
*   $S(x,y)$: $x$ 是 $y$ 的儿子
*   $F(x,y)$: $x$ 是 $y$ 的父亲
*   $B(x,y)$: $x$ 是 $y$ 的兄弟
*   $m$: 常量，表示 "myself" (我自己)

**翻译**：
$$ \forall x \forall y (F(x, m) \wedge S(y, x) \rightarrow B(y, m)) $$

## 自由变量与约束变量 (Free and Bound Variables)

**定义**：
令 $\Phi$ 为谓词逻辑中的一个公式。变量 $x$ 在 $\Phi$ 中的某次出现被称为 **自由的 (free)**，如果它是 $\Phi$ 解析树中的一个叶子节点，且从该节点向上的路径中不存在 $\forall x$ 或 $\exists x$ 节点。否则，该出现被称为 **约束的 (bound)**。
![[assets/自由约束变量解析树.png]]

对于 $\forall x \Phi$ (或 $\exists x \Phi$)，我们称 $\Phi$ 是量词 $\forall x$ (或 $\exists x$) 的 **作用域 (scope)**。
![[assets/量词作用域.png]]



## 替换 (Substitution)

变量是占位符，我们需要一种方法用更具体的信息来替换它们。

### 基本定义
给定变量 $x$，项 $t$ 和公式 $\Phi$，定义 $\Phi[t/x]$ 为通过将 $\Phi$ 中 **$x$ 的所有自由出现** 替换为 $t$ 而得到的公式。

示例：
$$ ((\forall x (P(x) \wedge Q(x))) \rightarrow (\neg P(x) \vee Q(y))) [f(x, y) / x] $$
结果为：
$$ (\forall x (P(x) \wedge Q(x))) \rightarrow (\neg P(f(x, y)) \vee Q(y)) $$
*(注意：量词 $\forall x$ 作用域内的 $x$ 是约束的，不被替换)*
![[assets/替换操作公式树.png]]

### 自由(Free)

**定义**：
给定项 $t$，变量 $x$ 和公式 $\Phi$，如果在 $\Phi$ 中 $x$ 的所有自由出现都不在任何 $\forall y$ 或 $\exists y$ 的作用域内（其中 $y$ 是 $t$ 中出现的任意变量），则称 **$t$ 对 $x$ 在 $\Phi$ 中是自由的 (free for $x$ in $\Phi$)**。

**备注**：
如果 $t$ 对 $x$ 在 $\Phi$ 中不是自由的，那么替换 $\Phi[t/x]$ 会导致 **变量捕获**，产生非预期的语义变化。(就是**防止重命名产生的歧义**)

**示例 (避免变量捕获)**：
原公式：$S(x) \wedge (\forall y (P(x) \rightarrow Q(y)))$
试图执行替换 $[y/x]$：
*   错误结果：$S(y) \wedge (\forall y (P(y) \rightarrow Q(y)))$
    *(这里 $t=y$，原公式中 $x$ 的自由出现位于 $\forall y$ 的作用域内，导致替换进去的 $y$ 被量词捕获)*

**解决方法**：
将原公式中的约束变量重命名 (例如 $y \to z$)：
$$ S(x) \wedge (\forall z (P(x) \rightarrow Q(z))) $$
然后再执行替换 $[y/x]$：
$$ S(y) \wedge (\forall z (P(y) \rightarrow Q(z))) $$

