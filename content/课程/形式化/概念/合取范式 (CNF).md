# CNF (Conjunctive Normal Form)

合取范式 (CNF) 是命题逻辑公式的一种标准形式。一个公式是 CNF，当且仅当它是子句的合取，而每个子句是文字的析取。

## 计算过程

将任意公式 $\Phi$ 转换为 CNF 的标准算法流程如下：

$$ \text{CNF}(\text{NNF}(\text{IMPL\_FREE}(\Phi))) $$

### 1. IMPL_FREE (消除蕴含)
消除公式中的蕴含 ($\rightarrow$) 和双蕴含 ($\leftrightarrow$)。
#### 保留所有$\neg$, 消去所有$\rightarrow$
*   $\Phi$ 是文字: return $\Phi$
*   **$\Phi$ 是 $\neg\Phi_1$: return $\neg\text{IMPL\_FREE}(\Phi_1)$**
*   $\Phi$ 是 $\Phi_1 \wedge \Phi_2$: return $\text{IMPL\_FREE}(\Phi_1) \wedge \text{IMPL\_FREE}(\Phi_2)$
*   $\Phi$ 是 $\Phi_1 \vee \Phi_2$: return $\text{IMPL\_FREE}(\Phi_1) \vee \text{IMPL\_FREE}(\Phi_2)$
*   **$\Phi$ 是 $\Phi_1 \rightarrow \Phi_2$: return $\neg\text{IMPL\_FREE}(\Phi_1) \vee \text{IMPL\_FREE}(\Phi_2)$**

### 2. NNF (否定范式)
将否定符号内推，直到它们仅出现在原子命题之前。
*   $\Phi$ 是文字: return $\Phi$
*   $\Phi$ 是 $\neg\neg\Phi_1$: return $\text{NNF}(\Phi_1)$
*   $\Phi$ 是 $\Phi_1 \wedge \Phi_2$: return $\text{NNF}(\Phi_1) \wedge \text{NNF}(\Phi_2)$
*   $\Phi$ 是 $\Phi_1 \vee \Phi_2$: return $\text{NNF}(\Phi_1) \vee \text{NNF}(\Phi_2)$
*   $\Phi$ 是 $\neg(\Phi_1 \wedge \Phi_2)$: return $\text{NNF}(\neg\Phi_1 \vee \neg\Phi_2)$
*   $\Phi$ 是 $\neg(\Phi_1 \vee \Phi_2)$: return $\text{NNF}(\neg\Phi_1 \wedge \neg\Phi_2)$

### 3. CNF & DISTR (转换为 CNF)
利用分配律处理析取项。
*   $\Phi$ 是文字: return $\Phi$
*   $\Phi$ 是 $\Phi_1 \wedge \Phi_2$: return $\text{CNF}(\Phi_1) \wedge \text{CNF}(\Phi_2)$
*   $\Phi$ 是 $\Phi_1 \vee \Phi_2$: return $\text{DISTR}(\text{CNF}(\Phi_1), \text{CNF}(\Phi_2))$

**DISTR 函数**:
*   $\Phi_1$ 是 $\Phi_{11} \wedge \Phi_{12}$: return $\text{DISTR}(\Phi_{11}, \Phi_2) \wedge \text{DISTR}(\Phi_{12}, \Phi_2)$
*   $\Phi_2$ 是 $\Phi_{21} \wedge \Phi_{22}$: return $\text{DISTR}(\Phi_1, \Phi_{21}) \wedge \text{DISTR}(\Phi_1, \Phi_{22})$
*   否则: return $\Phi_1 \vee \Phi_2$

---

## 示例

对公式 $\neg(r\wedge(\neg(q\rightarrow(\neg p\rightarrow(q\wedge r)))))$ 的转换过程：

### 1. IMPL_FREE
```text
IMPL_FREE(¬(r∧(¬(q→(¬p→(q∧r)))))) 
 = ¬(r∧(¬(¬q∨IMPL_FREE(¬p→(q∧r))))) 
 = ¬(r∧(¬(¬q∨(¬¬p∨(q∧r))))) 
```

### 2. NNF
```text
NNF(¬(r∧(¬(¬q∨(¬¬p∨(q∧r)))))) 
 = NNF(¬r)∨NNF(¬¬(¬q∨(¬¬p∨(q∧r)))) 
 = ¬r∨NNF(¬q∨(¬¬p∨(q∧r))) 
 = ¬r∨¬q∨NNF(¬¬p)∨(q∧r) 
 = ¬r∨¬q∨p∨(q∧r) 
```

### 3. CNF
```text
CNF(¬r∨¬q∨p∨(q∧r)) 
 = DISTR(CNF(¬r∨¬q∨p), CNF(q∧r)) 
 = DISTR(DISTR(CNF(¬r),CNF(¬q∨p)),CNF(q)∧CNF(r)) 
 = DISTR(¬r∨¬q∨p, q∧r) 
 = DISTR(¬r∨¬q∨p,q)∧DISTR(¬r∨¬q∨p,r) 
 = (¬r∨¬q∨p∨q)∧(¬r∨¬q∨p∨r)
```

