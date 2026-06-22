# Θ-notation（大 Θ 记号）

## 定义
$$ \Theta(g(n)) = \{\, f(n)\mid \exists\, c_1>0,\ c_2>0,\ n_0\ \text{s.t.}\ \forall\, n\ge n_0:\ 0\le c_1 g(n)\le f(n)\le c_2 g(n)\,\} $$
- 直观理解：$\Theta(g(n))$ 表示“与 $g(n)$ 同阶”的函数集合；亦即 $g(n)$ 是 $f(n)$ 的“紧确”渐近界（asymptotically tight bound）。
- 重要等价：$f(n)=\Theta(g(n)) \iff f(n)=O(g(n))$ 且 $f(n)=\Omega(g(n))$（上、下界同时成立）。

## 如何证明/否定一个 Θ 陈述
- 证明 $f(n)=\Theta(g(n))$：给出常数 $c_1,c_2>0$ 与阈值 $n_0$，使得对所有 $n\ge n_0$ 成立 $c_1 g(n)\le f(n)\le c_2 g(n)$。
- 否定：只需否定 $O$ 或 $\Omega$ 中任意一边即可（两者至少有一边不成立）。

## 例子
1) $\tfrac{1}{2}n^2 - \tfrac{1}{2}n = \Theta(n^2)$
   - 上界：对所有 $n\ge 0$，$$ \tfrac{1}{2}n^2 - \tfrac{1}{2}n \le \tfrac{1}{2}n^2, $$ 取 $c_2=\tfrac{1}{2}$。
   - 下界：当 $n\ge 2$，$$ \tfrac{1}{2}n^2 - \tfrac{1}{2}n \ge \tfrac{1}{2}n^2 - \tfrac{1}{4}n^2 = \tfrac{1}{4}n^2, $$ 取 $c_1=\tfrac{1}{4}$、$n_0=2$。

2) $n \ne \Theta(n^2)$
   - 尽管 $n=O(n^2)$，但 $n\ne \Omega(n^2)$（需要 $c n^2\le n$ 不可能），故非 $\Theta$。

3) $6n^3 \ne \Theta(n^2)$
   - $6n^3$ 不是 $O(n^2)$（增长阶更高），故非 $\Theta(n^2)$；虽然它是 $\Omega(n^2)$。

4) $n \ne \Theta(\log n)$
   - 需同时满足 $n=O(\log n)$ 与 $n=\Omega(\log n)$。后者成立，但前者不成立：若 $n\le c_2\log n$ 对所有大 $n$，则必须 $c_2\ge n/\log n$（趋于无穷），与“常数”矛盾。

## 技巧与注意
- 证明 $\Theta$ 时，通常先分别给出 $O$ 与 $\Omega$ 的常数与阈值，再取两者的较大阈值作为统一的 $n_0$。
- 若函数包含次要项，常用“吸收法”（如 $n\le n^2$、$\log n\le n$）来建立上下界常数。