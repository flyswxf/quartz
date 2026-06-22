# Test Doubles (测试替身) 练习题

## 一、判断题 (True/False)

1.  **[判断]** "Test Double" (测试替身) 是一个通用术语，涵盖了 Dummy, Fake, Stub, Spy, 和 Mock 等多种用于替代真实依赖组件的技术。
2.  **[判断]** Mock 对象 (模拟对象) 主要用于 **状态验证** (State Verification)，即检查测试结束后系统的状态是否正确。
3.  **[判断]** Stub (桩对象) 通常包含预定义的响应 (Canned answers)，用于在测试期间对调用返回特定的数据。
4.  **[判断]** Fake 对象 (伪对象) 具有可工作的实现 (例如使用内存中的 Map 代替数据库)，但通常因性能或安全原因不适合用于生产环境。
5.  **[判断]** Dummy 对象 (哑对象) 通常被传递给被测系统，但从未被实际使用或调用，主要用于满足参数列表的编译要求。
6.  **[判断]** Spy (间谍对象) 既可以像 Stub 一样返回预定义值，又可以记录某些方法的调用信息 (如调用次数、参数) 以便事后验证。
7.  **[判断]** 在单元测试中过度使用 Mock 会导致测试代码与实现细节高度耦合，从而使重构变得困难 (Brittle tests)。
8.  **[判断]** 经典风格 (Classicist) 的测试者倾向于使用 Mock 来隔离所有依赖，而模拟风格 (Mockist) 的测试者倾向于只在必要时使用 Stub。
9.  **[判断]** 如果一个测试替身在接收到意外的方法调用时会抛出异常或导致测试失败，那么它更像是一个 Mock 而不是 Stub。
10. **[判断]** 使用测试替身的主要目的之一是隔离被测单元 (SUT)，使其不受外部依赖 (如网络、数据库、时间) 的不确定性或缓慢速度的影响。

---

## 二、选择题 (Multiple Choice)

1.  以下哪种测试替身拥有 **可工作的业务逻辑实现**，但采取了简化的方式 (例如，不持久化到磁盘)？
    A. Mock
    B. Stub
    C. Fake
    D. Dummy

2.  Mock 和 Stub 之间最本质的区别是什么？
    A. Mock 用于集成测试，Stub 用于单元测试。
    B. Mock 关注 **行为验证** (Behavior Verification)，Stub 关注 **状态验证** (State Verification)。
    C. Stub 可以抛出异常，而 Mock 不能。
    D. Mock 是手动创建的，Stub 是自动生成的。

3.  当你需要测试一个函数，该函数需要 5 个参数，但其中 3 个参数在当前测试场景中完全不会被用到，你应该使用哪种替身来填充这 3 个参数？
    A. Spy
    B. Mock
    C. Fake
    D. Dummy

4.  如果你想验证 "邮件发送服务是否在订单处理过程中被调用了一次，且收件人地址正确"，你应该使用哪种替身？
    A. Stub
    B. Fake
    C. Mock (或 Spy)
    D. Dummy

5.  以下关于 Stub 的描述，哪项是 **错误** 的？
    A. 它提供了对调用的预定义响应。
    B. 它通常用于模拟无法控制的外部依赖 (如获取当前时间)。
    C. 如果测试断言失败，通常是因为 Stub 返回了错误的数据。
    D. 它会在方法被调用时主动验证参数是否符合预期，如果不符直接报错。 (这是 Mock 的职责)

6.  Martin Fowler 在 "Mocks Aren't Stubs" 一文中区分了两种测试流派，分别是：
    A. Unit Testers vs Integration Testers
    B. Classicists (经典派/状态验证) vs Mockists (模拟派/行为验证)
    C. Black-box Testers vs White-box Testers
    D. Manual Testers vs Automated Testers

7.  在 Python 的 `unittest.mock` 库或 Java 的 `Mockito` 中，默认创建的对象通常具备哪种替身的功能？
    A. 仅 Dummy
    B. 仅 Fake
    C. 混合了 Stub (可配置返回值) 和 Spy/Mock (可验证调用) 的功能
    D. 仅 Stub

8.  "Seam" (接缝) 这个概念在测试中的作用是：
    A. 指代码中的缺陷。
    B. 指可以插入测试替身而不修改源代码的地方 (例如通过依赖注入)。
    C. 指测试用例之间的依赖关系。
    D. 指数据库连接池。

9.  为什么说 Fake (伪对象) 比 Mock 更接近真实实现？
    A. 因为 Fake 实际上就是生产代码。
    B. 因为 Fake 有状态和行为逻辑 (如在内存中维护列表)，而 Mock 通常只是根据预期设定行为。
    C. 因为 Fake 运行速度更慢。
    D. 因为 Fake 只能由开发人员编写。

10. 当我们说 "Only Mock Types You Own" (只模拟你拥有的类型) 时，主要建议是：
    A. 不要模拟第三方库的 API，而应该为第三方库编写一层包装 (Wrapper/Adapter)，然后模拟这个包装。
    B. 只能模拟自己定义的类，不能模拟接口。
    C. 只能模拟私有方法。
    D. 必须购买 Mock 库的许可证。

---

## 答案与解析

### 一、判断题答案

1.  **True (对)** - Gerard Meszaros 在 xUnit Test Patterns 中定义的标准分类。
2.  **False (错)** - Mock 主要用于 **行为验证** (验证方法是否被调用、调用顺序、参数等)。状态验证通常不需要 Mock，只需检查 SUT 的返回值或状态变化。
3.  **True (对)** - Stub 的核心定义：提供 "Canned answers"。
4.  **True (对)** - Fake 是轻量级的实现，如 In-Memory Database。
5.  **True (对)** - Dummy 只是为了填坑，让代码能编译或运行，不产生实际交互。
6.  **True (对)** - Spy 就像是潜伏的间谍，默默记录发生的一切。
7.  **True (对)** - 这种现象被称为 "Overspecified Software" 或 "Coupling to Implementation"。如果重构改变了内部调用细节但没改变外部行为，Mock 测试可能会挂，这是不好的。
8.  **False (错)** - 说反了。Mockist (模拟派) 倾向于大量使用 Mock 进行行为验证；Classicist (经典派) 倾向于使用真实对象或 Stub 进行状态验证。
9.  **True (对)** - Mock 通常带有“期望”(Expectations)，如果不满足期望(如未调用或调用参数不对)就会失败。
10. **True (对)** - 提高测试速度和确定性是核心动力。

### 二、选择题答案

1.  **C** - Fake (伪对象) 是唯一有“逻辑”和“状态”的替身，只是实现方式简单。
2.  **B** - 这是最经典的区分。Stub 帮助测试运行(提供数据)，Mock 观察测试运行(验证行为)。
3.  **D** - Dummy (哑对象)。因为它们不被使用，仅仅是占位符。
4.  **C** - 需要验证“副作用” (Side Effect) —— 即邮件发送行为发生了。Mock 或 Spy 最适合。
5.  **D** - Stub 通常不负责验证调用参数 (那是 Mock 的工作)，它只负责无脑返回预设值。
6.  **B** - 经典派 (State Verification) vs 模拟派 (Behavior Verification)。
7.  **C** - 现代 Mock 框架生成的对象通常既可以 `when(...).thenReturn(...)` (Stub功能)，也可以 `verify(...)` (Mock/Spy功能)。
8.  **B** - 接缝 (Seam) 是解耦和注入替身的关键点。
9.  **B** - Fake 模拟了行为逻辑 (contract)，而 Mock 模拟了特定的交互场景。
10. **A** - 这是一个重要的设计原则。模拟第三方库会导致测试与第三方库的具体 API 耦合，且第三方库行为变更时测试可能无法感知。应该定义自己的 Adapter 接口并模拟它。
