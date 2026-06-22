在 Property-based Testing (PBT) 中，构思属性（Properties）往往比写代码更难。这里用通俗易懂的语言介绍四种核心方法：

## 1. Validity Testing (有效性测试)
**通俗理解**：“底线检查”或“冒烟测试”。
不管输入是什么，程序都**绝对不能**做的事情是什么？结果必须满足哪些**最基本的格式要求**？

*   **核心思想**：我们不检查结果的具体数值是否完全正确，而是检查结果是否“合法”。
*   **适用场景**：任何代码。这是最容易写的属性。
*   **检查点**：
    1.  **不崩溃**：程序不应抛出 500 错误或未捕获的异常。
    2.  **基本约束**：例如，生成的“价格”不能是负数；生成的“用户对象”必须有 ID。
    3.  **Schema 校验**：返回的 JSON 结构必须符合定义。

### 示例
假设有一个 `User` 类，不管怎么操作，用户名不能为空，邮箱必须合法。

```python
# Invariant check function (定义“什么是合法的”)
def is_valid_user(user):
    return len(user.username) > 0 and "@" in user.email

# Property
@given(user_strategy())
def test_user_validity(user):
    # 只要是生成的 User 对象，就必须合法
    assert is_valid_user(user)

@given(user_strategy(), new_email_strategy())
def test_update_validity(user, new_email):
    # 即使修改了邮箱，结果对象依然必须合法
    updated_user = user.update_email(new_email)
    assert is_valid_user(updated_user)
```

## 2. Postconditions (后置条件)
**通俗理解**：“输入与输出的契约”。
既然我给了你这个输入，那么你的输出必须满足什么条件？

*   **核心思想**：直接根据输入参数，验证输出结果的特征。注意，我们不需要在测试代码里重新实现一遍算法，而是检查结果的**性质**。
*   **适用场景**：简单的计算函数、转换函数。
*   **思考方式**：“调用函数后，什么事情**一定**会发生？”

### 示例
测试一个 `filter(predicate, list)` 函数（过滤列表）。

```python
@given(st.lists(st.integers()), st.functions(ret=st.booleans()))
def test_filter_postconditions(lst, predicate):
    result = filter(predicate, lst)
    
    # 1. 范围检查：过滤后的长度绝不会比原列表长
    assert len(result) <= len(lst)
    
    # 2. 效果检查：结果里的每个元素，都必须通过过滤条件
    for x in result:
        assert predicate(x)
        
    # 3. 关系检查：结果必须来源于原列表（不能凭空造数据），且保持原顺序
    assert is_subsequence(result, lst)
```

## 3. Metamorphic Properties (蜕变属性)
**通俗理解**：“相对正确性”或“参照系测试”。
我不知道 `f(x)` 的正确答案是多少（太难算了），但我知道 `f(x)` 和 `f(y)` 之间应该有什么关系。

*   **核心思想**：通过**两次不同的调用**，比较它们结果之间的差异。
*   **适用场景**：
    *   **Oracle Problem**：难以知道预期结果（比如搜索引擎、AI 模型、复杂模拟）。
    *   **部分功能验证**：优化前后的对比。
*   **常见模式**：
    *   **逆运算**：加密再解密等于原值 (`decode(encode(x)) == x`)。
    *   **幂等性**：去重两次等于去重一次 (`unique(unique(x)) == unique(x)`)。
    *   **不变性**：打乱顺序不影响排序结果 (`sort(shuffle(x)) == sort(x)`)。

### 示例
测试一个搜索引擎。很难知道 `search("apple")` 到底该返回哪些文档，但我们可以利用蜕变关系。

```python
@given(term=st.text(), extra=st.text())
def test_search_refinement(term, extra):
    # 构造两个相关的输入：宽泛的 vs 具体的
    term_general = term
    term_specific = term + " " + extra
    
    # 获取两个结果
    results_general = search(term_general)
    results_specific = search(term_specific)
    
    # 验证关系：搜得越细，结果应该越少（或者是原结果的子集）
    # 这就是“蜕变关系”：输入变长了 -> 输出变少了
    assert set(results_specific).issubset(set(results_general))
```

## 4. Inductive Testing (归纳测试)
**通俗理解**：“搭积木”或“数学归纳法”。
如果小规模的问题是对的，并且我们知道如何把小问题变成大问题，那么大问题也是对的。

*   **核心思想**：验证 **复杂结构** 的属性时，将其拆解为 **简单结构** 的属性。
*   **适用场景**：递归数据结构（树、链表）、递归算法。
*   **逻辑**：
    1.  **Base Case**：最简单的情况是对的（如空列表）。
    2.  **Step**：如果 `size(list)` 是对的，那么 `size(list + [new_item])` 也是对的。

### 示例
测试计算列表长度的函数 `size()`。

```python
# Base Case: 哪怕是空列表，也要能算出 0
def test_size_base_case():
    assert size([]) == 0

# Induction Step: 只要知道“小列表”的大小，就能算出“大列表”的大小
@given(x=st.integers(), xs=st.lists(st.integers()))
def test_size_induction_step(x, xs):
    # xs 是 "小列表"
    # full_list 是 "大列表" (多了一个元素)
    full_list = [x] + xs
    
    # 属性：大列表的大小 == 1 + 小列表的大小
    # 我们不需要知道 size(xs) 具体是几，只要这个关系成立就行
    assert size(full_list) == 1 + size(xs)
```
