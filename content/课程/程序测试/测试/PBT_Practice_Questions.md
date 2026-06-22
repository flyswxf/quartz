# Property-based Testing (PBT) 练习题

## 一、判断题 (True/False)

1.  **[判断]** Property-based Testing (基于属性的测试) 的主要思想是使用特定的、硬编码的输入输出来验证代码。
2.  **[判断]** Shrinking (约减/缩减) 是 PBT 工具的一个重要特性，它试图在测试失败时找到最小的失败用例。
3.  **[判断]** 只要基于属性的测试运行了足够多的次数 (例如 100 次) 且全部通过，我们就可以数学上证明代码是完全正确的。
4.  **[判断]** 在 PBT 中，"Generator" (生成器) 负责产生符合特定类型的随机数据。
5.  **[判断]** 只有纯函数式编程语言 (如 Haskell) 才能使用基于属性的测试。
6.  **[判断]** `decode(encode(x)) == x` 是一个典型的 "Round-trip" (往返) 属性。
7.  **[判断]** 基于属性的测试完全替代了传统的基于示例的单元测试 (Example-based Unit Testing)。
8.  **[判断]** 在 PBT 中，我们通常测试的是代码的不变量 (Invariants)，即在各种输入下都保持为真的条件。
9.  **[判断]** 如果一个属性测试失败，PBT 框架通常会保存该失败用例 (Replay/Reproduce)，以便在修复后进行回归测试。
10. **[判断]** Metamorphic Testing (蜕变测试) 的思想可以与 Property-based Testing 结合使用，利用蜕变关系作为属性。

---

## 二、选择题 (Multiple Choice)

1.  Property-based Testing 与传统的 Unit Testing (Example-based) 最大的区别在于：
    A. PBT 运行速度更快。
    B. PBT 自动生成输入数据，而传统测试使用手动定义的输入。
    C. PBT 不需要编写断言 (Assertions)。
    D. PBT 只能用于测试数学函数。

2.  以下哪个 Python 库是著名的 Property-based Testing 库？
    A. PyTest
    B. Unittest
    C. Hypothesis
    D. Mock

3.  当 PBT 发现一个失败的测试用例时，Shrinking 过程的作用是：
    A. 删除失败的测试代码。
    B. 忽略该错误并继续测试。
    C. 尝试简化输入数据，使其在保持失败的同时尽可能简单 (Human-readable)。
    D. 压缩日志文件以节省空间。

4.  属性 `sort(sort(list)) == sort(list)` 描述了排序函数的什么特性？
    A. Commutativity (交换律)
    B. Idempotence (幂等性)
    C. Associativity (结合律)
    D. Invertibility (可逆性)

5.  在 PBT 中，如果你想限制生成的数据范围 (例如，只生成正整数)，你应该使用：
    A. Assertions (断言)
    B. Filters / Preconditions (过滤器/前置条件)
    C. Mock objects (模拟对象)
    D. Global variables (全局变量)

6.  "Oracle Problem" (预言机问题) 指的是难以确定测试输出是否正确。PBT 如何缓解这个问题？
    A. 它自动修复代码。
    B. 它不需要检查输出。
    C. 它关注于检查通用的属性 (Properties) 而不是具体的输出值。
    D. 它总是使用参考实现 (Reference Implementation) 来对比。

7.  以下哪项 **不是** 常见的属性模式 (Property Pattern)？
    A. Round-trip (往返: 序列化 -> 反序列化 == 原始值)
    B. Invariant (不变量: 某些条件始终为真)
    C. Test Oracle (与可信的参考实现进行对比)
    D. Hardcoded Value (硬编码值: 输入 5 必须输出 10)

8.  如果你的 PBT 测试运行了 1000 次都通过了，这意味着：
    A. 代码没有 Bug。
    B. 代码在生成的这 1000 个输入上是正确的，增加了我们对代码的信心。
    C. 生成器可能坏了，没有生成有效数据。
    D. 你应该停止编写任何其他测试。

9.  QuickCheck 是 PBT 的鼻祖，它最初是为哪种语言开发的？
    A. Java
    B. Python
    C. Haskell
    D. C++

10. 假设你正在测试一个 `reverse` (反转列表) 函数，以下哪个属性是合理的？
    A. `reverse(list) == list` (仅对回文有效)
    B. `reverse(reverse(list)) == list` (两次反转回到原样)
    C. `len(reverse(list)) == 0`
    D. `reverse(list)[0] == list[0]`

---

## 答案与解析

### 一、判断题答案

1.  **False (错)** - 这是 Example-based testing 的描述。PBT 使用生成的数据和通用属性。
2.  **True (对)** - Shrinking 是 PBT 的核心优势之一，能帮助开发者快速定位问题根源。
3.  **False (错)** - 只能说明在这些特定样本上通过了，不能证明没有 Bug (除非输入空间极小被穷举)。
4.  **True (对)** - Generator 是 PBT 的基础组件。
5.  **False (错)** - 几乎所有主流语言 (Python, Java, C#, JS 等) 都有 PBT 库。
6.  **True (对)** - 编码/解码、序列化/反序列化是典型的 Round-trip 场景。
7.  **False (错)** - 两者互补。Example-based 适合文档化典型用法和边缘案例，PBT 适合探索性测试和覆盖更广的输入空间。
8.  **True (对)** - Invariants 是最常见的属性类型之一。
9.  **True (对)** - 许多框架支持 "Database" 或 "Example database" 来持久化失败案例。
10. **True (对)** - 蜕变关系 (如 `sin(x) = sin(pi - x)`) 就是一种很好的属性。

### 二、选择题答案

1.  **B** - 核心区别在于数据的来源 (自动生成 vs 手动指定)。
2.  **C** - Hypothesis 是 Python 中最流行的 PBT 库。
3.  **C** - Shrinking 的目的是去除无关噪声，找到触发 Bug 的最小输入。
4.  **B** - 幂等性：操作一次和操作多次结果相同。
5.  **B** - 使用 `filter`, `assume` 或在 generator 定义时施加约束。
6.  **C** - 通过检查属性 (如不变量、逆运算等)，我们不需要知道每个输入的具体确切输出，也能验证代码的正确性。
7.  **D** - 硬编码值是基于示例测试的做法，不是通用的属性模式。
8.  **B** - 测试只能证明 Bug 存在，不能证明 Bug 不存在。
9.  **C** - QuickCheck 由 Koen Claessen 和 John Hughes 在 Haskell 中发明。
10. **B** - 这是一个经典的 Involution (对合) 属性。
