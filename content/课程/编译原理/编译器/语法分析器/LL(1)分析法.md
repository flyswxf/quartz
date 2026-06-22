LL(1) 分析法是一种**自顶向下**的语法分析方法，属于**确定的自顶向下分析**。

* **L**：从左到右扫描输入串 (Left-to-right scan)。
* **L**：产生最左推导 (Leftmost derivation)。
* **1**：向前查看 1 个输入符号 (Lookahead 1 symbol)。

---

## 1. 核心概念

要判断和构造 LL(1) 分析器，需要依赖三个核心集合：`FIRST` 集、`FOLLOW` 集和 `SELECT` 集。

### 1.0 符号含义
* $\alpha, \beta$：通常表示**符号串**（由终结符和非终结符混合组成）。
* $S$：**文法开始符号**。也就是推导的起点。
* $\$$：**结束符**。代表输入串的结尾，用来标记句型的最右端。
* $\Rightarrow^*$：表示**经过零步或多步推导**。例如 $\alpha \Rightarrow^* a \dots$ 意思是符号串 $\alpha$ 经过若干步替换后，可以变成以终结符 $a$ 开头的串。
* $A \rightarrow \alpha$：**产生式规则**。
* $\epsilon$：表示**空串**（长度为0的字符串）。
* $V_T$：**终结符集合**（Terminal）。终结符是不能再继续向下推导的叶子节点，如 `+`、`*`、`id`、`a` 等小写字母或符号。
* $V_N$：**非终结符集合**（Non-terminal）。非终结符是可以继续推导的节点，通常用 $A$、$B$、$X$、$Y$ 等大写字母表示。
* $\setminus \{\epsilon\}$：集合操作，表示从集合中**剔除空串 $\epsilon$**。

### 1.1 FIRST 集 (首符号集)

> 一个符号（或者一串符号）在不断展开后，它最终形成的字符串，第一个字符可能是什么？

**定义**：设 $\alpha$ 是一符号串，则 $FIRST(\alpha)$ 是可以从 $\alpha$ 推导出的所有串的**首终结符**的集合。如果 $\alpha \Rightarrow^* \epsilon$，则 $\epsilon \in FIRST(\alpha)$。

$$ FIRST(\alpha) = \{ a \mid \alpha \Rightarrow^* a \dots, a \in V_T \} \cup \{ \epsilon \mid \alpha \Rightarrow^* \epsilon \} $$
符号含义参考[[LL(1)分析法#1.0 符号含义|符号含义]]

**计算规则**：
1. 若 $X \in V_T$（终结符），则 $FIRST(X) = \{X\}$。
2. 若 $X \in V_N$（非终结符）
	1. 若有产生式 $X \to a\alpha$，则 $a \in FIRST(X)$。
	2. 若有 $X \to \epsilon$，则 $\epsilon \in FIRST(X)$。
3. 若 $X \to Y_1Y_2\dots Y_k$，则将 $FIRST(Y_1) \setminus \{\epsilon\}$ 加入 $FIRST(X)$。
	1. 若 $Y_1 \dots Y_{i-1}$ 都能推导出 $\epsilon$，则将 $FIRST(Y_i) \setminus \{\epsilon\}$ 加入 $FIRST(X)$。
	2. 若所有 $Y_i$ 都能推导出 $\epsilon$，则 $\epsilon \in FIRST(X)$。

>[!tip]- **通俗解释**
>* **规则 1**：如果它本身就是个终结符（比如 `+`），那它展开后的第一个字符肯定就是 `+`。
>* **规则 2**：如果它是个非终结符（比如 $A$），并且有规则说 $A \to a \dots$，那 $a$ 肯定是首符号之一。如果有 $A \to \epsilon$，那空串也是它的可能结果。
>* **规则 3**：如果 $X \to Y_1 Y_2 Y_3$。
>	* 那么 $X$ 的首符号，首先取决于 $Y_1$ 的首符号。所以要把 $FIRST(Y_1)$ 加进来。
>	* 但是！如果 $Y_1$ 有可能推导为空（$\epsilon$），$Y_2$ 的首符号就会顶上来成为 $X$ 的首符号。此时就要把 $FIRST(Y_2)$ 也加进来。
>	* 如果 $Y_2$ 也能为空，就继续看 $Y_3$，以此类推。如果全部都能为空，那整个 $X$ 就能为空，最后把 $\epsilon$ 加入 $FIRST(X)$。

>[!tip]- **示例**：
>假设有以下文法：
>1. $S \to AB$
>2. $A \to a \mid \epsilon$
>3. $B \to b \mid c$
>
>* **求 $FIRST(A)$**：根据规则 2，因为有 $A \to a$ 和 $A \to \epsilon$，所以 $FIRST(A) = \{a, \epsilon\}$。
>* **求 $FIRST(B)$**：根据规则 2，因为有 $B \to b$ 和 $B \to c$，所以 $FIRST(B) = \{b, c\}$。
>* **求 $FIRST(S)$**：根据规则 3，看产生式 $S \to AB$。
>	* 首先看 $A$，把 $FIRST(A)$ 中的非空元素加入 $FIRST(S)$，即加入 $a$。
>	* 因为 $A$ 可以推导出 $\epsilon$（即 $A$ 可能“消失”），所以必须继续往后看 $B$ 的首符号。
>	* 把 $FIRST(B)$ 中的非空元素加入 $FIRST(S)$，即加入 $b, c$。
>	* 因为 $B$ 不能推导出 $\epsilon$，所以推导到此结束。
>	* 最终结果：$FIRST(S) = \{a, b, c\}$。

### 1.2 FOLLOW 集 (后跟符号集)
> 在整个推导过程中，谁有可能紧跟在 $B$ 的后面？

**定义**：对于非终结符 $A$，$FOLLOW(A)$ 是在所有句型中紧跟在 $A$ 后面的**终结符**的集合。如果 $A$ 可能是某个句型的最右符号，则结束符 $\$$ 属于 $FOLLOW(A)$。

$$ FOLLOW(A) = \{ a \mid S \Rightarrow^* \dots Aa \dots, a \in V_T \} $$
符号含义参考[[LL(1)分析法#1.0 符号含义|符号含义]]

注意：FOLLOW 集合里**绝对不能包含空串 $\epsilon$**。

**计算规则**：
1. 对于文法开始符号 $S$，将结束符 $\$$ 加入 $FOLLOW(S)$。
2. 对于产生式 $A \to \alpha B \beta$，将 $FIRST(\beta) \setminus \{\epsilon\}$ 加入 $FOLLOW(B)$。
3. 对于产生式 $A \to \alpha B$ 或 $A \to \alpha B \beta$（且 $\beta \Rightarrow^* \epsilon$），将 $FOLLOW(A)$ 加入 $FOLLOW(B)$。

>[!tip]- **通俗解析**：
>* **规则 1（开局自带 $\$$）**：如果是文法的开始符号 $S$，那它的背后可能什么都没有了，所以默认给它后面跟一个结束符 $\$$。
>* **规则 2（看右邻居）**：如果在某个规则里，长成 $\dots B \beta \dots$ 这样（$B$ 后面跟着 $\beta$）。那 $B$ 后面的符号，自然就是 $\beta$ 的首符号。所以要把 $FIRST(\beta)$ 中除了空串以外的元素加到 $FOLLOW(B)$ 里。
>* **规则 3（继承左部）**：这是最容易晕的一点。如果长成 $A \to \dots B$（$B$ 在最末尾），或者 $A \to \dots B \beta$（虽然有 $\beta$，但 $\beta$ 会消失变成 $\epsilon$）。这个时候 $B$ 后面没东西了，怎么办？**$B$ 就会暴露在 $A$ 的最右边，所以谁跟在 $A$ 后面，谁就自然跟在 $B$ 后面。** 这时要把 $FOLLOW(A)$ 整个倒进 $FOLLOW(B)$ 里。

>[!tip]- **简单示例**：
>假设有以下文法，开始符号是 $S$：
>1. $S \to aAB$
>2. $A \to c \mid \epsilon$
>3. $B \to d$
>
>已知 FIRST 集：$FIRST(A) = \{c, \epsilon\}$, $FIRST(B) = \{d\}$
>
>* **求 $FOLLOW(S)$**：根据规则 1，$S$ 是开始符号，加入 $\$$。所以 $FOLLOW(S) = \{\$\}$。
>* **求 $FOLLOW(A)$**：找 $A$ 在产生式右部出现的位置，发现 $S \to aAB$。
>	* $A$ 后面跟着 $B$。根据规则 2，要把 $FIRST(B)$ 加入 $FOLLOW(A)$。
>	* 因为 $FIRST(B) = \{d\}$ 且不包含 $\epsilon$（$B$ 不会消失），所以 $FOLLOW(A) = \{d\}$。
>* **求 $FOLLOW(B)$**：找 $B$ 在产生式右部出现的位置，发现 $S \to aAB$。
>	* $B$ 后面没东西了（它在产生式的最末尾）。
>	* 根据规则 3，谁跟在 $S$ 后面，谁就跟在 $B$ 后面。所以要把 $FOLLOW(S)$ 加入 $FOLLOW(B)$。
>	* 最终 $FOLLOW(B) = \{\$\}$。

### 1.3 SELECT 集 (可选符号集)
> 当手头拿着非终结符 $A$，并且偷看了一眼后面的输入字符是 $a$ 时，到底该不该用 $A \to \alpha$ 这条规则来替换 $A$？

**定义**：对于产生式 $A \to \alpha$，$SELECT(A \to \alpha)$ 表示在面对特定的输入符号时，可以选择该产生式进行推导的符号集合。

**计算规则**：
* 如果 $\alpha$ 不能推导出 $\epsilon$：
  $$ SELECT(A \to \alpha) = FIRST(\alpha) $$
* 如果 $\alpha$ 能推导出 $\epsilon$：
  $$ SELECT(A \to \alpha) = (FIRST(\alpha) \setminus \{\epsilon\}) \cup FOLLOW(A) $$

符号含义参考[[LL(1)分析法#1.0 符号含义|符号含义]]

>[!tip]- **通俗解析**：
>* **情况 1（右边不会消失）**：如果 $\alpha$ 绝对不可能变成空串，那么只要我接下来看到的字符在 $FIRST(\alpha)$ 里，我就大胆地用这条规则展开。
>* **情况 2（右边会消失）**：如果 $\alpha$ 有可能变成空串（比如本身就是 $A \to \epsilon$，或者 $\alpha$ 里的非终结符都能变空）。这意味着什么？这意味着如果我用了这条规则，$A$ 就凭空消失了！既然 $A$ 消失了，那接下来应该匹配的字符，就应该是原本排在 $A$ 后面的字符。所以，除了看 $FIRST(\alpha)$，我还得看 $FOLLOW(A)$。只要下一个字符在 $FIRST(\alpha)$（除去 $\epsilon$）或者 $FOLLOW(A)$ 里，我都可以用这条规则。

>[!tip]- **简单示例**：
>假设文法：
>1. $A \to aB$
>2. $A \to \epsilon$
>
>已知：$FIRST(aB) = \{a\}$, $FIRST(\epsilon) = \{\epsilon\}$, $FOLLOW(A) = \{b, c\}$
>
>* **求 $SELECT(A \to aB)$**：
>	* 因为 $aB$ 不能推导出 $\epsilon$（属于情况 1）。
>	* 所以 $SELECT(A \to aB) = FIRST(aB) = \{a\}$。
>	* **含义**：如果当前是非终结符 $A$，且下一个输入字符是 $a$，那就用 $A \to aB$ 规则。
>* **求 $SELECT(A \to \epsilon)$**：
>	* 因为右部直接就是 $\epsilon$（属于情况 2）。
>	* 所以 $SELECT(A \to \epsilon) = (FIRST(\epsilon) \setminus \{\epsilon\}) \cup FOLLOW(A) = \emptyset \cup \{b, c\} = \{b, c\}$。
>	* *含义**：如果当前是非终结符 $A$，且下一个输入字符是 $b$ 或 $c$（也就是 $A$ 后面的符号），说明 $A$ 在这里必须“让位”消失掉，所以应该用 $A \to \epsilon$ 规则。

---

## 2. LL(1) 文法的定义

一个文法是 LL(1) 文法，当且仅当对于任意非终结符 $A$ 的两个不同产生式 $A \to \alpha$ 和 $A \to \beta$，满足：

$$ SELECT(A \to \alpha) \cap SELECT(A \to \beta) = \emptyset $$

即：在任何情况下，给定当前非终结符和向前查看的 1 个输入符号，都可以**唯一确定**选用哪个产生式，不会产生冲突。

---

## 3. LL(1) 分析器的构造

LL(1) 分析器由**分析栈**、**分析表**和**控制程序**组成。

### 3.1 构造 LL(1) 分析表 (Parsing Table)

分析表 $M[A, a]$ 是一个二维数组，行表示非终结符 $A$，列表示终结符 $a$（包括结束符 $\$$）。

**构造算法**：
对于文法中的每一个产生式 $A \to \alpha$：
1. 对于 $SELECT(A \to \alpha)$ 中的每个终结符 $a$，将 $A \to \alpha$ 填入 $M[A, a]$。
2. **如果某个单元格中有多个产生式，说明该文法不是 LL(1) 文法。** 
   即违反[[LL(1)分析法#2. LL(1) 文法的定义|LL(1) 文法的定义]], 使$$ SELECT(A \to \alpha) \cap SELECT(A \to \beta) \ne \emptyset $$

**判断一个文法是否是 LL(1) 文法**
1. 对文法[[消除左递归]] 
2. 对文法[[提取公共左因子]]
3. 构造 LL(1) 分析表, 分析是否存在产生式冲突
### 3.2 LL(1) 分析过程

1. 初始化：将结束符 $\$$ 和文法开始符号 $S$ 压入分析栈，栈顶为 $S$。读入第一个输入符号。
2. 循环执行以下操作，直到栈为空或发生错误：
   * 若栈顶符号 $X$ 是终结符：
     * 若 $X$ 等于当前输入符号 $a$，则匹配成功，弹出栈顶符号，读取下一个输入符号。
     * 若 $X \neq a$，则报错。
   * 若栈顶符号 $X$ 是非终结符：
     * 查分析表 $M[X, a]$。
     * 若表项为产生式 $X \to Y_1Y_2\dots Y_k$，则弹出栈顶符号 $X$，并将 $Y_k, \dots, Y_2, Y_1$ 依次压入栈中（逆序压栈，保证 $Y_1$ 在栈顶）。
     * 若表项为空，则报错。
   * 若栈顶符号为 $\$$，且当前输入符号也为 $\$$，则分析成功结束。

---

## 4. 示例分析

**给定文法：**
1. $E \to TE'$
2. $E' \to +TE' \mid \epsilon$
3. $T \to FT'$
4. $T' \to *FT' \mid \epsilon$
5. $F \to (E) \mid \text{id}$

**步骤一：求 FIRST 集**
* $FIRST(F) = \{ (, \text{id} \}$
* $FIRST(T') = \{ *, \epsilon \}$
* $FIRST(T) = FIRST(F) = \{ (, \text{id} \}$
* $FIRST(E') = \{ +, \epsilon \}$
* $FIRST(E) = FIRST(T) = \{ (, \text{id} \}$

**步骤二：求 FOLLOW 集**
* $FOLLOW(E) = \{ \$, ) \}$
* $FOLLOW(E') = FOLLOW(E) = \{ \$, ) \}$
* $FOLLOW(T) = (FIRST(E') \setminus \{\epsilon\}) \cup FOLLOW(E) \cup FOLLOW(E') = \{ +, \$, ) \}$
* $FOLLOW(T') = FOLLOW(T) = \{ +, \$, ) \}$
* $FOLLOW(F) = (FIRST(T') \setminus \{\epsilon\}) \cup FOLLOW(T) \cup FOLLOW(T') = \{ *, +, \$, ) \}$

**步骤三：构造预测分析表**

| 非终结符 | id                | +                 | *             | (           | )                 | $                 |
| :------- | :---------------- | :---------------- | :------------ | :---------- | :---------------- | :---------------- |
| **E**    | $E \to TE'$       |                   |               | $E \to TE'$ |                   |                   |
| **E'**   |                   | $E' \to +TE'$     |               |             | $E' \to \epsilon$ | $E' \to \epsilon$ |
| **T**    | $T \to FT'$       |                   |               | $T \to FT'$ |                   |                   |
| **T'**   |                   | $T' \to \epsilon$ | $T' \to *FT'$ |             | $T' \to \epsilon$ | $T' \to \epsilon$ |
| **F**    | $F \to \text{id}$ |                   |               | $F \to (E)$ |                   |                   |

表中没有冲突项，因此该文法是 LL(1) 文法。

**步骤四：输入串 `id + id * id` 的分析过程**

| 栈         | 输入串           | 动作                                           |
| :--------- | :--------------- | :--------------------------------------------- |
| $\$$E      | id + id * id$\$$ | 查表 $M[E, \text{id}]$，使用 $E \to TE'$       |
| $\$$E'T    | id + id * id$\$$ | 查表 $M[T, \text{id}]$，使用 $T \to FT'$       |
| $\$$E'T'F  | id + id * id$\$$ | 查表 $M[F, \text{id}]$，使用 $F \to \text{id}$ |
| $\$$E'T'id | id + id * id$\$$ | 匹配 id，出栈，读下一个字符                    |
| $\$$E'T'   | + id * id$\$$    | 查表 $M[T', +]$，使用 $T' \to \epsilon$        |
| $\$$E'     | + id * id$\$$    | 查表 $M[E', +]$，使用 $E' \to +TE'$            |
| $\$$E'T+   | + id * id$\$$    | 匹配 +，出栈，读下一个字符                     |
| $\$$E'T    | id * id$\$$      | 查表 $M[T, \text{id}]$，使用 $T \to FT'$       |
| $\$$E'T'F  | id * id$\$$      | 查表 $M[F, \text{id}]$，使用 $F \to \text{id}$ |
| $\$$E'T'id | id * id$\$$      | 匹配 id，出栈，读下一个字符                    |
| $\$$E'T'   | * id$\$$         | 查表 $M[T', *]$，使用 $T' \to *FT'$            |
| $\$$E'T'F* | * id$\$$         | 匹配 *，出栈，读下一个字符                     |
| $\$$E'T'F  | id$\$$           | 查表 $M[F, \text{id}]$，使用 $F \to \text{id}$ |
| $\$$E'T'id | id$\$$           | 匹配 id，出栈，读下一个字符                    |
| $\$$E'T'   | $\$$             | 查表 $M[T', \$]$，使用 $T' \to \epsilon$       |
| $\$$E'     | $\$$             | 查表 $M[E', \$]$，使用 $E' \to \epsilon$       |
| $\$$       | $\$$             | 接受 (Accept)                                  |
