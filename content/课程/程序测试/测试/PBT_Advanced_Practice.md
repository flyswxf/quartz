# Property-based Testing (PBT) 进阶练习与实战

## 第一部分：进阶客观题 (30题)

### 一、判断题 (True/False)

1.  **[判断]** 在 PBT 中，使用 `filter` (过滤) 来限制生成数据的范围通常比直接定义更精确的 `generator` (生成器) 效率更高。
2.  **[判断]** Stateful Testing (基于状态的测试) 是 PBT 的一种高级形式，用于验证对象或系统在经过一系列操作后的状态是否符合预期。
3.  **[判断]** "Shrinking" 过程不仅能简化输入数据，有时也能简化导致失败的操作序列 (在 Stateful Testing 中)。
4.  **[判断]** 所有的 PBT 框架都保证生成的随机数据在多次运行之间是完全确定性的 (Deterministic)，无需任何配置。
5.  **[判断]** 针对浮点数 (Floating-point) 的 PBT 往往比整数更复杂，因为需要处理 NaN, Infinity 以及精度误差带来的 `==` 判断失效问题。
6.  **[判断]** 如果一个属性仅仅是复制了被测函数的实现逻辑 (例如测试 `add(a, b)` 时写属性 `assert add(a, b) == a + b`)，这种测试通常价值较低，被称为 "Tautological Property" (重言式属性)。
7.  **[判断]** Model-based PBT (基于模型的 PBT) 通常需要维护一个简化的“模型” (Model)，该模型的行为应与被测系统 (SUT) 一致。
8.  **[判断]** 在 Hypothesis (Python) 库中，`@given` 装饰器用于指定测试函数的输入策略。
9.  **[判断]** PBT 无法用于测试非纯函数 (Impure functions) 或涉及 I/O 的代码。
10. **[判断]** 模糊测试 (Fuzzing) 与 Property-based Testing 有很多相似之处，现代 Fuzzing 工具也常用于发现安全漏洞。
11. **[判断]** 当测试一个压缩算法时，`len(compress(s)) < len(s)` 是一个总是成立的属性。
12. **[判断]** "Metamorphic Property" (蜕变属性) 不需要知道确切的预期输出，只需要知道输入变化时输出应如何变化。
13. **[判断]** 在 PBT 中，生成递归数据结构 (如树、图) 是不可能的。
14. **[判断]** 即使没有显式的断言，PBT 也能通过检测 "Crash" (崩溃) 或 "Exception" (未捕获异常) 来发现错误。
15. **[判断]** 只要定义了足够多的属性，就不再需要进行代码审查 (Code Review)。

### 二、选择题 (Multiple Choice)

16. 在 PBT 中，当你发现生成的测试数据大多被 `filter` (过滤器) 拒绝导致测试效率极低时，最好的做法是：
    A. 增加测试运行次数。
    B. 禁用 Shrinking。
    C. 重写 Generator，构造性地生成符合条件的数据 (Constructive generation)。
    D. 忽略这些警告。

17. 针对 "幂等性" (Idempotence) 属性，以下哪个表达式是正确的描述 (假设 `f` 是被测函数)？
    A. `f(a, b) == f(b, a)`
    B. `f(f(x)) == f(x)`
    C. `f(a) + f(b) == f(a + b)`
    D. `f(x) == x`

18. 在测试一个 "序列化/反序列化" 库时，最重要的属性通常是：
    A. 序列化后的字符串长度最短。
    B. 序列化过程不消耗内存。
    C. Round-trip (往返) 属性: `deserialize(serialize(x)) == x`。
    D. 序列化后的字符串只包含 ASCII 字符。

19. 假设你要测试一个寻找列表中最大值的函数 `my_max(list)`。以下哪个属性 **不是** 好的属性？
    A. 结果必须是列表中的一个元素: `my_max(l) in l`。
    B. 结果必须大于等于列表中的所有其他元素。
    C. `my_max(l)` 的结果等于 `sorted(l)[-1]` (与可信的参考实现对比)。
    D. `my_max(l)` 对于任何列表 `l` 都返回 42。

20. "Test Oracle" (测试预言) 在 PBT 中通常指：
    A. 一个能够预测未来的 AI 模型。
    B. 一个被认为是正确的参考实现 (Reference Implementation) 或验证机制。
    C. 数据库管理员。
    D. 测试生成的随机种子。

21. 在 Stateful PBT (基于状态的测试) 中，测试执行的过程通常被建模为：
    A. 单次函数调用。
    B. 两个并行线程。
    C. 一系列命令/动作 (Action) 的序列。
    D. 数据库事务。

22. 假设你在测试一个 Web API，以下哪个不是典型的 PBT 应用场景？
    A. 验证 API 对各种畸形 JSON 输入的鲁棒性 (不会 500 Crash)。
    B. 验证 API 的响应时间总是小于 10ms。
    C. 验证写入的数据能被正确读取 (Read-after-Write)。
    D. 验证分页接口返回的总条目数与数据库一致。

23. 关于 "Shrinking" (约减)，以下哪种说法是错误的？
    A. 它可以将一个包含 1000 个元素的失败列表约减为包含 1-2 个元素的列表。
    B. 它总是能找到全局最小的失败用例。
    C. 它依赖于数据类型的内部结构 (如整数趋向于 0，列表趋向于空)。
    D. 它是 PBT 区别于普通随机测试的关键特性。

24. 在 Hypothesis 库中，`strategies.integers(min_value=0, max_value=10)` 会生成什么？
    A. 0 到 10 之间的随机浮点数。
    B. 0 到 10 之间的随机整数 (包含 0 和 10)。
    C. 一个长度为 10 的整数列表。
    D. 总是返回 5。

25. 为什么说 `sort(reverse(list))` 和 `sort(list)` 的结果应该相同？这属于哪种属性？
    A. Round-trip
    B. Invariant
    C. Metamorphic property (蜕变属性) / Test Oracle via Model
    D. Performance property

26. 遇到 "Flaky Test" (不稳定的测试，有时过有时不过) 时，在 PBT 背景下最常见的原因是：
    A. 宇宙射线。
    B. 某个特定的边缘情况 (Edge case) 只有在极低概率下才会被生成器选中。
    C. 计算机过热。
    D. PBT 库本身的代码错误。

27. 如果你想测试两个函数 `f` 和 `g` 是互逆的 (Inverse)，你应该检查：
    A. `f(x) == g(x)`
    B. `f(g(x)) == x` AND `g(f(y)) == y`
    C. `f(x) + g(x) == 0`
    D. `f(x) * g(x) == 1`

28. 在测试并发系统 (Concurrency) 时，PBT 可以用来：
    A. 证明没有死锁。
    B. 生成随机的操作交错顺序 (Interleaving) 来寻找竞态条件 (Race Conditions)。
    C. 自动修复锁争用。
    D. 提高系统吞吐量。

29. "Validity Construction" (有效性构造) 在生成复杂数据 (如电子邮件地址、SQL 查询) 时意味着：
    A. 生成任意随机字符串，然后过滤掉无效的。
    B. 根据语法规则 (Grammar) 直接生成合法的字符串。
    C. 从互联网下载真实数据。
    D. 手动输入数据。

30. 以下哪个不是 PBT 的局限性？
    A. 可能难以定义通用的属性。
    B. 运行时间通常比执行单个单元测试长。
    C. 能够探索人类开发者未想到的边缘情况。 (这是优点，不是局限性)
    D. 随机性可能导致难以复现 (如果没有记录 Seed)。

---

## 第二部分：场景设计大题 (4题)

### 题目 1：字符串压缩算法 (RLE)
**规格描述 (Specification):**
你需要测试一个简单的 **游程编码 (Run-Length Encoding, RLE)** 模块。
该模块包含两个函数：
1.  `rle_encode(s: str) -> str`: 将字符串压缩。例如 "AAAABBBCCDAA" -> "4A3B2C1D2A"。
2.  `rle_decode(s: str) -> str`: 将压缩后的字符串解压回原始形式。

**要求:**
请设计至少 3 个关键的属性 (Properties) 来测试这个模块的正确性。

**参考答案与解析:**
> 1.  **Round-trip Property (往返属性)**:
>     *   **描述**: 对于任何字符串 `s`，先编码再解码，应该还原为原始字符串。
>     *   **代码/伪代码**: `rle_decode(rle_encode(s)) == s`
>     *   **解析**: 这是最基本也是最强的正确性保证。
>
> 2.  **Output Format Validity (输出格式有效性)**:
>     *   **描述**: 编码后的字符串应该只包含“数字”和“原始字符”。或者更具体地，它不应为空（除非输入为空）。
>     *   **解析**: 验证输出符合基本的格式约束。
>
> 3.  **Compression Effectiveness (非严格的压缩效果)**:
>     *   **描述**: 对于包含大量重复字符的输入（例如由 PBT 生成的长重复片段 `strategies.text().map(lambda c: c * 100)`），编码后的长度应小于原始长度。
>     *   **注意**: 不能对所有字符串断言 `len(encode(s)) < len(s)`，因为对于 "ABCDE" 这样的无重复串，RLE 可能会变长 ("1A1B1C1D1E")。
>
> 4.  **Compositionality / Concatenation (组合性)**:
>     *   **描述**: 虽然 RLE 不完全满足 `encode(s1 + s2) == encode(s1) + encode(s2)` (因为边界可能合并)，但我们可以测试特定情况。例如，如果 `s1` 的结尾字符与 `s2` 的开头字符不同，则该等式成立。

---

### 题目 2：优先级队列 (Priority Queue)
**规格描述 (Specification):**
你需要测试一个 **Priority Queue (优先级队列)** 类。
它支持以下操作：
1.  `insert(item, priority)`: 插入一个元素及其优先级（整数，越小优先级越高）。
2.  `pop()`: 移除并返回优先级最高的元素。如果队列为空，抛出异常。
3.  `peek()`: 返回优先级最高的元素但不移除。
4.  `is_empty()`: 返回队列是否为空。

**要求:**
请设计测试用例或属性，特别是利用 **Model-based Testing (基于模型/状态的测试)** 的思路。

**参考答案与解析:**
> 1.  **Model-based Comparison (基于模型的对比)**:
>     *   **模型**: 使用一个简单的 `List` of `(priority, item)` tuples 作为模型。
>     *   **逻辑**:
>         *   当执行 `SUT.insert(i, p)` 时，`Model.append((p, i))` 并排序。
>         *   当执行 `SUT.pop()` 时，检查 `SUT` 是否为空。如果不为空，`Model` 也应不为空，且 `SUT.pop()` 的结果应等于 `Model.pop(0)` (假设模型已按优先级排序)。
>         *   **解析**: 用一个简单但低效的实现（列表+排序）来验证高效但复杂的实现（堆）。
>
> 2.  **Invariant: Output Ordering (输出有序性)**:
>     *   **描述**: 连续调用 `pop()` 直到队列为空，生成的元素序列应该是按优先级单调递增（或递减，取决于定义）的。
>     *   **伪代码**:
>         ```python
>         items = generate_random_items()
>         for i in items: pq.insert(i)
>         result = []
>         while not pq.is_empty(): result.append(pq.pop())
>         assert result == sorted(result, key=priority)
>         ```
>
> 3.  **Invariant: Size Consistency (大小一致性)**:
>     *   **描述**: 插入 N 个元素后，应该能恰好 Pop 出 N 个元素。`is_empty()` 在 Pop N 次后应为 True。
>
> 4.  **Idempotence of Peek (Peek 的幂等性)**:
>     *   **描述**: 连续多次调用 `peek()` 应返回相同结果，且不改变队列状态（即随后的 `pop()` 结果不变）。

---

### 题目 3：银行转账系统 (Bank Transfer)
**规格描述 (Specification):**
测试一个 `transfer(from_account, to_account, amount)` 函数。
*   每个账户有 `balance` (余额)。
*   转账要求 `from_account` 余额充足。
*   转账金额 `amount` 必须为正数。
*   如果成功，`from` 扣除 `amount`，`to` 增加 `amount`。

**要求:**
设计属性来验证资金安全和逻辑正确性。

**参考答案与解析:**
> 1.  **Invariant: Conservation of Money (资金守恒)**:
>     *   **描述**: 在转账前后，系统中所有账户的余额总和 (Total Balance) 必须保持不变。
>     *   **代码**: `sum(balances_before) == sum(balances_after)`
>     *   **解析**: 这是金融系统最重要的不变量，防止资金凭空产生或消失。
>
> 2.  **Post-condition: Balance Updates (余额更新正确性)**:
>     *   **描述**:
>         *   `from_account.balance_after == from_account.balance_before - amount`
>         *   `to_account.balance_after == to_account.balance_before + amount`
>         *   其他无关账户余额不变。
>
> 3.  **Validity Check: Non-negative Balance (非负余额约束)**:
>     *   **描述**: 任何操作结束后，任何账户的余额都不应为负数（假设不允许透支）。
>
> 4.  **Transaction Atomicity (事务原子性)**:
>     *   **描述**: 如果转账失败（例如余额不足），两个账户的余额都应保持原样。不能出现“扣了款但没到账”的情况。

---

### 题目 4：时间区间合并 (Time Interval Merge)
**规格描述 (Specification):**
函数 `merge_intervals(intervals: List[Tuple[int, int]]) -> List[Tuple[int, int]]`。
输入是一组时间区间 `(start, end)`，输出是合并重叠区间后的列表。
例如：`[(1, 4), (2, 5), (7, 9)]` -> `[(1, 5), (7, 9)]`。

**要求:**
设计属性来测试该算法。

**参考答案与解析:**
> 1.  **Property: Coverage Consistency (覆盖范围一致性)**:
>     *   **描述**: 合并前后，任何一个时间点 `t`，如果它在输入的某个区间内，它也必须在输出的某个区间内；反之亦然。
>     *   **伪代码**: 对于任意整数 `t`，`any(s <= t < e for s, e in input) == any(s <= t < e for s, e in output)`。
>
> 2.  **Property: Disjoint Output (输出不重叠)**:
>     *   **描述**: 输出的区间列表中，任意两个区间不应重叠。
>     *   **解析**: 这是 `merge` 操作的基本定义。
>
> 3.  **Property: Ordering (有序性)**:
>     *   **描述**: 输出的区间通常应按 `start` 时间排序（取决于具体需求，但通常如此）。
>
> 4.  **Metamorphic: Permutation Invariance (排列不变性)**:
>     *   **描述**: 打乱输入列表的顺序 (`shuffle(input)`), `merge_intervals` 的结果应该是一样的（假设输出是规范化排序的）。
>     *   **代码**: `merge_intervals(shuffle(intervals)) == merge_intervals(intervals)`

---

## 附：第一部分客观题答案与解析

### 一、判断题答案

1.  **False (错)** - `filter` 会拒绝大量不符合条件的数据，导致生成器不断重试，效率极低。应尽量使用 `map` 或更精确的构造方法。
2.  **True (对)** - Stateful testing 用于验证状态机行为。
3.  **True (对)** - Shrinking 也会尝试缩短操作序列 (Sequence of actions) 以找到最小复现步骤。
4.  **False (错)** - 通常需要固定 Seed (种子) 才能保证 Deterministic。
5.  **True (对)** - 浮点数比较需要 epsilon，且 NaN/Inf 会破坏很多假设。
6.  **True (对)** - 这种属性没有增加额外价值，被称为 Oracle property 但如果实现一样就是 Tautology。
7.  **True (对)** - Model 通常是简单的、低效的参考实现。
8.  **True (对)** - Hypothesis 的核心装饰器。
9.  **False (错)** - 可以测试，但需要 Setup/Teardown 或 Mock，或者在 Stateful Testing 中建模副作用。
10. **True (对)** - 两者边界日益模糊，都涉及随机数据生成。
11. **False (错)** - 鸽巢原理 (Pigeonhole Principle)，不可能所有字符串都被压缩变短，否则可以无限压缩至 0。
12. **True (对)** - 蜕变测试的核心优势。
13. **False (错)** - Hypothesis 等库支持 `recursive()` 策略生成树或图。
14. **True (对)** - 这种称为 "Implicit Oracle" (隐式预言)，如不崩溃、不超时。
15. **False (错)** - 测试不能替代审查，测试只能证明 Bug 存在。

### 二、选择题答案

16. **C** - 构造性生成 (Constructive generation) 是解决 Filter 效率低下的标准方法。
17. **B** - 幂等性定义：`f(f(x)) = f(x)`。A 是交换律。
18. **C** - Round-trip 是序列化最核心的属性。
19. **D** - 这是一个无意义的属性（除非题目是测试返回 42 的函数）。
20. **B** - Oracle 是判定结果是否正确的机制，通常指参考实现。
21. **C** - 状态测试通常生成一系列的操作 (Actions) 来改变状态。
22. **B** - 性能测试 (Performance Testing) 通常是独立的，虽然 PBT 可以做，但鲁棒性(A)、功能一致性(C, D) 是更典型的 PBT 目标。
23. **B** - Shrinking 只能找到 **局部** 最小值 (Local minimum)，不能保证全局最小，且依赖于收敛算法。
24. **B** - `integers` 策略通常包含端点。
25. **C** - Metamorphic Property (或者 Model Oracle)。利用了“排序后的结果应与输入顺序无关”这一蜕变关系。
26. **B** - 随机测试生成的边缘数据 (如 0, 空列表, 极大数) 触发了隐藏 Bug，而下次没生成就过了。
27. **B** - 互逆函数的定义。
28. **B** - 随机调度/交错 (Interleaving) 是发现并发 Bug 的利器。
29. **B** - 基于文法 (Grammar-based) 的生成。
30. **C** - 能够探索边缘情况是 PBT 的 **优势 (Strength)**，不是局限性。
