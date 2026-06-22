## 1. 模型检测 (Model Checking) 概览

### 核心定义
模型检测是一种自动化验证技术，用于检查系统是否满足特定属性。其核心数学表达为：
$$ M \models \psi $$

*   **$M$ (Model)**: 系统模型。通常使用 **Kripke Structure** (Kripke 结构) 来描述系统的状态和转换。在 Maude 中，这对应于 `system module` 定义的重写理论。
*   **$\psi$ (Property)**: 属性规范。使用 **LTL (线性时序逻辑)** 公式来描述系统应该满足的行为性质。
*   **$\models$ (Satisfies)**: 满足关系。即检查模型 $M$ 的所有执行路径是否都符合规范 $\psi$。

---

## 2. 数学基础: Kripke Structure

为了进行模型检测，系统必须被形式化为一个 **Kripke 结构** $\mathcal{K}$：
$$ \mathcal{K} = \langle S, \rightarrow_{\mathcal{K}}, L \rangle $$

1.  **$S$ (Set of States)**: 所有可能状态的集合。
2.  **$\rightarrow_{\mathcal{K}}$ (Transition Relation)**: 状态转换关系 ($S \times S$)。
    *   **关键特性 - Totality (完全性)**: 关系必须是完全的。即对于每一个状态 $s \in S$，都必须存在一个后继状态 $s'$ 使得 $s \rightarrow_{\mathcal{K}} s'$。
    *   *直观理解*: 系统不能有“死胡同”（死锁）。如果系统设计中有终止状态，通常通过添加指向自身的自环 (Self-loop) 来满足完全性。
3.  **$L$ (Labeling Function)**: 标记函数 $L: S \rightarrow 2^{AP}$。
    *   它将每个状态映射到一组在该状态下为 **True** 的**原子命题 (Atomic Propositions, AP)**。
    *   例如: $L(s_{on}) = \{on\}$ 表示在 $s_{on}$ 状态下，命题 "on" 是真的。

---

## 3. LTL 线性时序逻辑 (Syntax & Semantics)

LTL 用于描述沿着时间轴（无限执行路径）的属性。

### 3.1 核心时序算子 (Temporal Operators)

| 符号 (Math)            | Maude 表示  | 名称                  | 含义 (Intuition)                                                                     |
| :------------------- | :-------- | :------------------ | :--------------------------------------------------------------------------------- |
| $\bigcirc P$         | `O P`     | **Next** (下一步)      | 在**紧接的下一个**状态，P 必须成立。                                                              |
| $P_1 \mathbf{U} P_2$ | `P1 U P2` | **Until** (直到)      | $P_1$ 必须一直保持成立，**直到** $P_2$ 成立的那一刻。注意：**$P_2$ 必须在未来某一刻真的发生**，不能无限推迟。               |
| $\square P$          | `[] P`    | **Always** (总是)     | 在路径的**每一个**状态（从现在到无穷远），P 都必须成立。<br>*(推导: $\square P \equiv \neg \Diamond \neg P$)* |
| $\Diamond P$         | `<> P`    | **Eventually** (最终) | 在未来的**某个**时刻，P 终将成立。<br>*(推导: $\Diamond P \equiv \text{True } \mathbf{U} P$)*      |

### 3.2 逻辑连接词 (Boolean Connectives)
*   $\neg P$ (`~ P`): 非
*   $P_1 \land P_2$ (`P1 /\ P2`): 与
*   $P_1 \lor P_2$ (`P1 \/ P2`): 或
*   $P_1 \rightarrow P_2$ (`P1 -> P2`): 蕴含

---

## 4. 属性分类: Safety vs. Liveness

| 类型                | 描述                                                  | 典型公式模式                    | 例子                                                                |
| :---------------- | :-------------------------------------------------- | :------------------------ | :---------------------------------------------------------------- |
| **Safety** (安全性)  | **"Nothing bad ever happens"**<br>(坏事永不发生)          | $\square \neg \text{bad}$ | **互斥**: 两个进程永远不能同时在临界区。<br>$\square \neg (crit(A) \land crit(B))$ |
| **Liveness** (活性) | **"Something good eventually happens"**<br>(好事终将发生) | $\Diamond \text{good}$    | **响应**: 请求最终会被处理。<br>$\square (req \rightarrow \Diamond ack)$     |

---

## 5. 实战演练 (LTL Practice Exercises)

以下例子来自课程 PPT，展示了如何将自然语言需求转化为严格的 LTL 公式。

### Exercise 1: Safe temperature (Safety)
> **Requirement**: From now on, the temperature is always within a safe range.
> (从现在起，温度永远保持在安全范围内。)

*   **Atomic Proposition**: `tempOK`
*   **LTL Formula**:
    $$ \square \text{tempOK} $$

### Exercise 2: Alarm must be reset (Response)
> **Requirement**: Whenever the alarm is active, it must eventually be reset.
> (每当报警器激活，它必须最终被重置。)

*   **Atomic Propositions**: `alarm`, `reset`
*   **LTL Formula**:
    $$ \square (\text{alarm} \rightarrow \Diamond \text{reset}) $$

### Exercise 3: At least one reboot (Liveness)
> **Requirement**: The controller will reboot at least once in the future.
> (控制器在未来至少会重启一次。)

*   **Atomic Proposition**: `reboot`
*   **LTL Formula**:
    $$ \Diamond \text{reboot} $$

### Exercise 4: Initialization (Complex Until)
> **Requirement**: The system stays in initialization mode until it becomes configured, and while it is in initialization mode it must not be running.
> (系统保持初始化模式直到配置完成；且在初始化期间，它绝不能处于运行状态。)

*   **Atomic Propositions**: `init`, `configured`, `running`
*   **LTL Formula**:
    $$ (\text{init } \mathbf{U} \text{ configured}) \land \square (\text{init} \rightarrow \neg \text{running}) $$
*   **解析**:
    1.  前半部分 `init U configured` 描述了状态流转：一直是 `init` 直到 `configured`。
    2.  后半部分 `[] (init -> ~ running)` 是一个附加的不变式约束：只要是在 `init` 状态，`running` 就必须是假。

### Exercise 5: Maintenance (Next & Until)
> **Requirement**: Whenever a maintenance request occurs, then at the **next step** the system must enter maintenance mode and remain in maintenance mode **until** maintenance is done.
> (每当收到维护请求，下一步系统必须进入维护模式，并保持在该模式直到维护完成。)

*   **Atomic Propositions**: `maintenanceRequest`, `maintenance`, `done`
*   **LTL Formula**:
    $$ \square (\text{maintenanceRequest} \rightarrow \bigcirc (\text{maintenance} \land (\text{maintenance } \mathbf{U} \text{ done}))) $$
*   **解析**:
    *   $\square (\text{req} \rightarrow ...)$: 全局监视请求。
    *   $\bigcirc (...)$: 约束**下一时刻**。
    *   $\text{maint} \land (\text{maint } \mathbf{U} \text{ done})$: 
        *   进入维护模式 (`maintenance` is true)。
        *   并且保持维护模式直到完成 (`maintenance U done`)。
