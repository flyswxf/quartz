# Asymptotic Notations — 关系与定理总览

本笔记汇总 $O,\ \Omega,\ \Theta,\ o,\ \omega$ 五类渐近记号的核心关系、传递性/对称性等定理，配合你已建立的笔记：[[BigO-notation]], [[Omega-notation]], [[Theta-notation]], [[SmallO-notation]], [[little-omega-notation]].

---
## 基本定义（简述）
- $f=O(g)$：存在常数 $c>0, n_0$，当 $n\ge n_0$ 有 $0\le f(n)\le c\,g(n)$（上界）。
- $f=\Omega(g)$：存在常数 $c>0, n_0$，当 $n\ge n_0$ 有 $0\le c\,g(n)\le f(n)$（下界）。
- $f=\Theta(g)$：存在常数 $c_1,c_2>0, n_0$，当 $n\ge n_0$ 有 $c_1 g(n)\le f(n)\le c_2 g(n)$（同阶、紧确界）。
- $f=o(g)$：对任意常数 $c>0$，存在 $n_0$ 使 $f(n)\le c\,g(n)$；等价于比值极限 $f/g\to 0$（在渐近正前提下）。
- $f=\omega(g)$：对任意常数 $c>0$，存在 $n_0$ 使 $c\,g(n)\le f(n)$；等价于比值极限 $f/g\to \infty$（在渐近正前提下）。

> 直观类比：$O\sim\le$，$\Omega\sim\ge$，$\Theta\sim=$，$o\sim<$，$\omega\sim>$。但它们不是全序关系，许多函数彼此不可比较。

---
## 等价与包含关系
- $f=\Theta(g) \iff f=O(g)$ 且 $f=\Omega(g)$（两边界同时成立）。
- $o(g) \subsetneq O(g)$，$\omega(g) \subsetneq \Omega(g)$（严格包含）。
- $\Theta(g) = O(g) \cap \Omega(g)$（在“渐近正”前提下成立）。
- 转置对称：$f=O(g) \iff g=\Omega(f)$；$f=o(g) \iff g=\omega(f)$。

---
## 传递性（Transitivity）
- 若 $f=\Theta(g)$ 且 $g=\Theta(h)$，则 $f=\Theta(h)$。
- 若 $f=O(g)$ 且 $g=O(h)$，则 $f=O(h)$；同理对 $\Omega$。
- 若 $f=o(g)$ 且 $g=o(h)$，则 $f=o(h)$；同理对 $\omega$。

---
## 自反性（Reflexivity）与对称性（Symmetry）
- 自反性：对任意 $f$，都有 $f=\Theta(f)$、$f=O(f)$、$f=\Omega(f)$。
- 对称性：$\Theta$ 具对称性：$f=\Theta(g)$ 当且仅当 $g=\Theta(f)$。
- 注意：$O$、$\Omega$ 一般不具双向对称；其对偶体现在“转置对称”上。
- 严格记号：$f\ne o(f)$、$f\ne \omega(f)$（除非 $f$ 最终为 0）。

---
## 极限判别法（在 $f,g$ 渐近正时）
- 若 $\limsup\limits_{n\to\infty} f(n)/g(n) < \infty$，则 $f=O(g)$。
- 若 $\liminf\limits_{n\to\infty} f(n)/g(n) > 0$，则 $f=\Omega(g)$。
- 若极限 $\lim f/g = c\in(0,\infty)$，则 $f=\Theta(g)$。
- 若极限为 $0$，则 $f=o(g)$；若极限为 $\infty$，则 $f=\omega(g)$。

---
## 例子速查（与 PPT 一致的风格）
- $2n^2 = O(n^3)$；$n^2 = O(n^2)$；$n \ne O(n\log n)$?（错误示例）应为 $n = O(n\log n)$；而 $n^3 \ne O(n\log n)$。
- $n = \Omega(2n)$；$n^3 = \Omega(n^2)$；$n = \Omega(\log n)$。
- $n \ne \Theta(n^2)$；$6n^3 \ne \Theta(n^2)$；$(6n^3+1)\log n/(n+1)=\Theta(n^2\log n)$（主导项法）。
- $n = o(n\log n)$；$(\log n)^k = o(n^{\varepsilon})$；$2^n = \omega(n^k)$。

---
## 常见证明套路
- 主导项吸收：把低阶项并入高阶项（如 $n\le n^2$、$\log n\le n$）。
- 指数与多项式比较：$2^n$ 支配任何 $n^k$；多项式支配任意固定次数对数幂。
- 阶乘与指数：用斯特林公式 $n!\sim\sqrt{2\pi n}\,(n/e)^n$ 比较 $n!$ 与指数族。

> 小结：判断量级时，先确定族（常数/对数/多项式/指数/超指数），再用上述等价与传递性质快速定位。