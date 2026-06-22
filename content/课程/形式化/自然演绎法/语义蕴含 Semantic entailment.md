若对每个语义解释 $S$，只要对所有 $i=1,\ldots,n$ 都有 $S(\Phi_i)=\mathrm{T}$，就必然有 $S(\Psi)=\mathrm{T}$，则记为：

$$\Phi_1,\ \Phi_2,\ \ldots,\ \Phi_n \vDash \Psi$$

该关系称为“语义蕴含”。它表达的是：前提在任意解释下为真时，结论也为真。

### 示例：
$$p,\ q \vDash p \land (q \lor \lnot r)$$。
解释：当 $p=\mathrm{T}$ 且 $q=\mathrm{T}$ 时，式 $p \land (q \lor \lnot r)$ 也为真，不论 $r$ 的取值。

### 与语法可证明性对比
语义蕴含记作 $\vDash$，基于“在所有解释下为真”；语法可证明性记作 $\vdash$，基于推演规则与证明系统。

## 语义模型与解释（形象版）

### 模型的组成
- 一个非空集合 $A$（也叫“域”），表示“具体值的宇宙”。
- 每个函数符号 $f\in\mathcal{F}$ 都由一个具体函数 $f^{\mathcal{M}}:A^n\to A$ 来解释。
- 每个谓词符号 $P\in\mathcal{P}$ 都由一个具体子集 $P^{\mathcal{M}}\subseteq A^n$（或等价的真值函数）来解释。
- 要点：$f$、$P$ 是“符号”，而 $f^{\mathcal{M}}$、$P^{\mathcal{M}}$ 是在模型中的“具体含义”。

### 直观例子1：实数模型
- 取 $A=\mathbb{R}$；函数符号集 $\mathcal{F}=\{+,\times, -\}$；谓词符号集 $\mathcal{P}=\{=,\le,<,\text{zero}\}$。
- 解释：$+^{\mathcal{M}},\ *^{\mathcal{M}},\ -^{\mathcal{M}}$ 分别是实数的加、乘、减；$=^{\mathcal{M}},\ \le^{\mathcal{M}},\ <^{\mathcal{M}}$ 是相等、大小关系；$\text{zero}^{\mathcal{M}}(r)$ 当且仅当 $r=0$。
- 示例公式：$\forall x\forall y\big(\text{zero}(y)\to x\times y=y\big)$。这句话的意思是：“任意 $x,y$，如果 $y$ 是 $0$，则 $x\times y=y$”，在实数模型中为真。

### 直观例子2：二进制字符串模型
- 取 $A$ 为所有二进制串的集合（含空串 $\varepsilon$）。
- 函数符号集 $\mathcal{F}=\{e,\ \cdot\}$，其中常元 $e$ 解释为 $\varepsilon$，二元函数 $\cdot$ 解释为“连接字符串”。
- 谓词符号集 $\mathcal{P}=\{\le\}$，其中 $\le^{\mathcal{M}}$ 表示“前缀关系”：$s_1\le s_2$ 当且仅当 $s_1$ 是 $s_2$ 的前缀。
- 公式转中文并判断：
  - $\forall x\big((x\le x\cdot e)\ \land\ (x\cdot e\le x)\big)$：每个串都与自身连接空串互为前缀（为真）。
  - $\exists y\forall x(y\le x)$：存在一个串是所有串的前缀（确有其事，$y=e$）。
  - $\forall x\exists y(y\le x)$：每个串都有一个前缀（为真，取 $y=x$ 或 $y=e$）。
  - $\forall x\forall y\forall z\big((x\le y)\to(x\cdot z\le y\cdot z)\big)$：若 $x$ 是 $y$ 的前缀，则在右侧共同连接同一个 $z$，前缀关系保持（一般不成立；反例：$x=\varepsilon,\ y=0,\ z=1$，则 $x\cdot z=1$, $y\cdot z=01$，$1\nleq 01$）。
  - $\neg\exists x\forall y\big((x\le y)\to(y\le x)\big)$：不存在一个串，一旦它是任何串的前缀，该串也都是它的前缀（为真；因为非空串不可能是所有串的前缀，空串也不是所有非空串的后缀）。

### 环境（变量赋值表）
- 量词的语义依赖一个“查表”环境 $I:\text{var}\mapsto A$，把变量映到具体值。
- 更新写作 $I[x\mapsto a]$：除把 $x$ 映到 $a$ 外，其他变量按原来 $I$ 保持不变。
- 直观理解：评价 $\forall x\,\Phi$ 时，要“遍历”域中每个 $a\in A$，用 $I[x\mapsto a]$ 去判断 $\Phi$ 是否为真；评价 $\exists x\,\Phi$ 时，找一个合适的 $a$ 使得在 $I[x\mapsto a]$ 下 $\Phi$ 为真。

### 两个常见等价与非等价
- 否定与量词的交换：$\neg\forall x\,P(x)\equiv\exists x\,\neg P(x)$。语义上：并非对所有值都真，等价于存在一个值使其为假。
- 量词顺序的差异：$\exists x\,\forall y\,P(x,y)\not\equiv\forall y\,\exists x\,P(x,y)$。在字符串前缀模型中，后者常真（每个 $y$ 自己就是它的前缀），但前者要求“一个统一的 $x$ 适配所有 $y$”，通常做不到。