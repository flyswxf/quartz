# Ω-notation（大 Ω 记号）

## 定义
$$ \Omega(g(n)) = \{\, f(n)\mid \exists\, c>0,\ n_0\ \text{s.t.}\ \forall\, n\ge n_0:\ 0\le c\,g(n)\le f(n)\,\} $$
- 直观理解：$\Omega(g(n))$ 是所有“增长不低于 $g(n)$ 的同阶或更高阶”的函数集合；亦即 $g(n)$ 是 $f(n)$ 的渐近下界（asymptotic lower bound）。
- 常见前提：当 $n$ 足够大时 $f(n),\ g(n)\ge 0$（渐近正）。

## 如何证明/否定一个 Ω 陈述
- 证明 $f(n)=\Omega(g(n))$：给出常数 $c>0$ 与阈值 $n_0$，使得对所有 $n\ge n_0$ 成立 $c\,g(n)\le f(n)$。
- 否定 $f(n)=\Omega(g(n))$：反证，若假设存在 $c,n_0$ 满足定义，则构造足够大的 $n$ 使 $f(n)<c\,g(n)$，与假设矛盾。
- 关系：$f(n)=\Omega(g(n))\iff g(n)=O(f(n))$（互为对偶）。

## 例子
1) $5n^2 = \Omega(n)$
   $$ c\,n \le 5n^2 \text{ 当 } n\ge 1. $$ 取 $c=1$、$n_0=1$ 即可。

2) $100n+5 \ne \Omega(n^2)$（反证法）
   假设存在 $c>0, n_0$ 使得 $\forall n\ge n_0$, 有 $c\,n^2 \le 100n+5$。当 $n\ge 1$，$100n+5\le 105n$，于是
   $$ c\,n^2 \le 105n \ \Rightarrow\ n(cn-105)\le 0. $$
   由于 $n>0$，得 $cn-105\le 0$，即 $n\le 105/c$，与“对所有足够大的 $n$”矛盾，故不成立。

3) 进一步例子：
   - $n = \Omega(2n)$：取 $c=\tfrac{1}{2}$，则 $c\cdot 2n = n \le n$（恒成立）。
   - $n^3 = \Omega(n^2)$：取 $c=1$、$n_0=1$，有 $n^2\le n^3$ 当 $n\ge 1$。
   - $n = \Omega(\log n)$：取 $c=1$、$n_0=2$，有 $\log n \le n$ 当 $n\ge 2$。

## 技巧与注意
- 寻找简单不等式（如 $\log n\le n$、$n\le n^2$）来构造常数因子。
- 若要证明“不属于 $\Omega(g)$”，通常展示 $f(n)$ 不能对所有大 $n$ 都主导 $g(n)$ 的常数倍。