# 自然演绎各证明规则的健全性（Soundness）补全

目标：对任意证明长度 $k$，若某赋值 $v$ 使前提集合 $\Gamma$ 以及第 $k$ 行的所有开放假设（记为 $\mathrm{Assum}(d_k)$）均为真，则第 $k$ 行公式 $\chi_k$ 也为真。

我们对证明长度进行归纳，并对第 $k$ 行使用的“证明规则”做分案。

- 基例 $k=1$：唯有“premise（前提）”，显然成立。
- 归纳步 $k>1$：假设对所有长度小于 $k$ 的证明该命题成立。设第 $k$ 行由规则 justification-$k$ 得到，作规则分案如下。

下文统一约定：若某行编号 $i<k$ 出现公式 $\varphi$ 或 $\psi$，则依归纳假设，在任意使 $\Gamma$ 与 $\mathrm{Assum}(d_k)$ 为真的赋值 $v$ 下，该行结论均取真值 $T$。并使用命题联结词的真值表语义：

- $\varphi \land \psi$ 为真当且仅当 $\varphi$ 与 $\psi$ 均真。
- $\varphi \lor \psi$ 为假当且仅当 $\varphi$ 与 $\psi$ 均假。
- $\varphi \to \psi$ 为假当且仅当 $\varphi$ 真而 $\psi$ 假，否则为真。
- $\lnot\varphi$ 为真当且仅当 $\varphi$ 为假。
- $\bot$ 恒为假。

---

## 规则分案证明（补全）

1) $\land$e1（合取消去-左）：
- 第 $k$ 行结论为 $\varphi$，且某行 $i<k$ 给出 $\varphi\land\psi$。
- 由归纳假设，$\varphi\land\psi$ 为 $T$。由 $\land$ 的真值表，$\varphi$ 为 $T$。故第 $k$ 行也为真。

1) $\land$e2（合取消去-右）：
- 同上，某行 $i<k$ 给出 $\varphi\land\psi$，第 $k$ 行结论为 $\psi$。
- 由归纳假设与真值表，$\psi$ 为 $T$。故第 $k$ 行为真。

1) $\bot$e（爆炸律/矛盾消去）：
- 第 $k$ 行从某行 $i<k$ 的 $\bot$ 推出任意公式 $\chi_k$。
- 若依归纳假设该行 $\bot$ 需在 $v$ 下取 $T$，但 $\bot$ 在任何赋值下恒为假，因此不存在满足“前提与开放假设都真”的赋值使该前提成立。
- 因此命题“对一切使前提与开放假设为真的赋值 $v$，结论 $\chi_k$ 为真”在语义上真（空真/vacuous）。故规则健全。

1) $\lnot$i（否定引入）：
- 有一子证明盒以假设 $\varphi$ 起始并导出 $\bot$，第 $k$ 行结论为 $\lnot\varphi$。
- 取任意使 $\Gamma$ 与 $\mathrm{Assum}(d_k)$ 为真的赋值 $v$。若再令 $v(\varphi)=T$，则该盒的开放假设得到满足，依归纳假设其末行 $\bot$ 必为真，与 $\bot$ 恒假矛盾。
- 故在满足 $\Gamma$ 与 $\mathrm{Assum}(d_k)$ 的任意 $v$ 下必有 $v(\varphi)=F$，于是由 $\lnot$ 的真值表，$v(\lnot\varphi)=T$。

1) $\lnot\lnot$e（双重否定消去，经典）：
- 某行 $i<k$ 给出 $\lnot\lnot\varphi$，结论为 $\varphi$。
- 由归纳假设，$\lnot\lnot\varphi$ 在 $v$ 下为真，因 $\lnot\varphi$ 为假，故 $\varphi$ 为真。于是第 $k$ 行为真。

1) $\lor$i1（析取引入-左）：
- 某行 $i<k$ 给出 $\varphi$，第 $k$ 行结论为 $\varphi\lor\psi$。
- 依归纳假设 $\varphi$ 为真，故由 $\lor$ 的真值表，$\varphi\lor\psi$ 为真。

1) $\lor$i2（析取引入-右）：
- 类似地，若 $\psi$ 为真，则 $\varphi\lor\psi$ 为真。故成立。

1) $\to$i（蕴含引入）：
- 有一子证明盒以假设 $\varphi$ 起始并导出 $\psi$，第 $k$ 行结论为 $\varphi\to\psi$。
- 取任意满足 $\Gamma$ 与 $\mathrm{Assum}(d_k)$ 的赋值 $v$。
  - 若 $v(\varphi)=T$，则该盒开放假设满足，依归纳假设末行 $\psi$ 为真，于是 $\varphi\to\psi$ 真。
  - 若 $v(\varphi)=F$，则 $\varphi\to\psi$ 按真值表亦真。
- 因而 $\varphi\to\psi$ 在所有此类 $v$ 下为真。

1) $\to$e（蕴含消去/MP）：
- 某行 $i<k$ 给出 $\varphi\to\psi$，另一行 $j<k$ 给出 $\varphi$，第 $k$ 行结论为 $\psi$。
- 由归纳假设，$\varphi\to\psi$ 与 $\varphi$ 在 $v$ 下都为真。若 $\varphi$ 真而 $\varphi\to\psi$ 真，则必有 $\psi$ 真（否则蕴含为假）。故第 $k$ 行为真。

1)  $\lnot$e（否定消去/矛盾导出）：
- 有两行分别给出 $\lnot\varphi$ 与 $\varphi$，第 $k$ 行结论为 $\bot$。
- 若两前提在同一赋值 $v$ 下同为真，则由 $\lnot$ 的真值表得 $\varphi$ 必假，与 $\varphi$ 真矛盾。故不存在使两前提与开放假设同时为真的赋值。
- 因而命题“对一切满足条件的赋值 $v$，第 $k$ 行（$\bot$）为真”空真成立；该规则健全。

---

由以上分案可知，所有列出的规则在给定的语义下均满足健全性：凡从 $\Gamma$ 以自然演绎可证的结论，都被所有使 $\Gamma$ 为真的赋值满足。至此补全完成。