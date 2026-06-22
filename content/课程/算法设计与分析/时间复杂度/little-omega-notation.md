# ω-notation（小 ω 记号）

## 定义
$$ \omega(g(n)) = \{\, f(n)\mid \forall\, c>0\;\exists\, n_0\ \text{s.t.}\ \forall\, n\ge n_0:\ 0\le c\,g(n)\le f(n)\,\} $$
- 直观理解：$f(n)=\omega(g(n))$ 表示 $f$ 相对 $g$ 的增长“严格更快”（严格大于），即任意常数倍的 $g$ 最终都被 $f$ 超过。若 $f, g$ 渐近正，等价于
  $$ \lim_{n\to\infty} \frac{f(n)}{g(n)} = \infty. $$

## 与其它记号的关系
- $f=\omega(g) \Rightarrow f=\Omega(g)$（严格大于蕴含不低于）。但反过来不一定成立。
- 转置对称：$f=\omega(g) \iff g=o(f)$。
- 传递性：若 $f=\omega(g)$ 且 $g=\omega(h)$，则 $f=\omega(h)$。
- 与 $\Theta$ 的关系：若 $f=\omega(g)$，则 $f\ne \Theta(g)$；严格大于不可能同阶。

## 如何证明/否定一个小 ω 陈述
- 直接按定义：对任意常数 $c>0$，给出阈值 $n_0$ 使 $c\,g(n)\le f(n)$ 当 $n\ge n_0$。
- 极限法：在 $f,g$ 渐近正时，证明 $\lim f/g=\infty$ 即得；或用反证：若存在上界常数倍则比值有界，与趋于无穷矛盾。

## 例子
1) $n^2 = \omega(n)$：当 $n\ge 1$，$c\,n \le n^2$ 取 $c=1$ 即可；或比值 $n\to\infty$。
2) $n\log n = \omega(n)$：比值 $\log n\to\infty$。
3) $n^{\varepsilon} = \omega((\log n)^k)$（任意常数 $\varepsilon>0,\ k>0$）。
4) $2^n = \omega(n^k)$（任意常数 $k$）。
5) 反例：$n \ne \omega(n)$、$\log n \ne \omega(n)$（比值不趋于无穷）。

## 技巧与注意
- 与小 o 相对，小 ω 的阈值也可以依赖给定的常数 $c$。
- 若函数可能出现负值，需说明“渐近正”的前提或仅讨论绝对值，以避免定义中的不等式失效。