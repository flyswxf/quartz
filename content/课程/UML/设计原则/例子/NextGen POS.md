在经典的 NextGen POS 示例中，计算 Sale 的 total 涉及三个类：

- **Sale**
- **SalesLineItem**
- **ProductSpecification**

它们之间的关系（你应该已经熟悉）：

```
Sale 1..*──contains──► SalesLineItem ──has──► ProductSpecification
```

计算 total 的核心流程：

### ▶ Step 1：Sale.total() 被调用

作为“整体”，Sale 是计算总额的入口点。

### ▶ Step 2：Sale 向每个 SalesLineItem 请求 subtotal

```java
public Money getTotal() {
    Money total = new Money(0);
    for (SalesLineItem sli : lineItems) {
        total = total.plus(sli.getSubtotal());
    }
    return total;
}
```

### ▶ Step 3：SalesLineItem 计算小计

SalesLineItem 知道：

- quantity（数量）
- productSpec（商品规格，包含单价）

因此它是 **部分信息专家**，负责计算小计：

```java
public Money getSubtotal() {
    return productSpec.getPrice().times(quantity);
}
```

### ▶ Step 4：ProductSpecification 提供 price

它是“价格信息”的 expert。

---

# 🎯 最终：三个类协作完成 total

- Sale：知道有哪些 line items → **负责遍历**
- SalesLineItem：知道 quantity 和 productSpec → **负责小计**
- ProductSpecification：知道 price → **提供单价**

每个类只做自己“知道的”，正是 Information Expert 的典范。

---

# 三个类协作是不是就会耦合？（书中解释）

书中说：

> _“the fulfillment of a responsibility often requires information that is spread across different classes. This implies that many partial information experts will collaborate.”_

翻译：  
一个职责的实现往往需要多个对象的信息，因此多个“部分专家”会协作。

**协作必然带来一些耦合，但不是坏事——是“必要耦合”。**

Larman 强调：
> Collaboration is normal and expected. What matters is to keep coupling low.

**关键点：合作 ≠ 高耦合。  
高耦合来自不必要的耦合。**

在 total 的案例里：

- Sale “本来就”与 SalesLineItem 有关联（contains）
- SalesLineItem “本来就”与 ProductSpecification 有关联（has）

**它们之间的引用关系本来就存在于领域模型中，因此协作不会增加额外耦合。**

Larman 在 Creator、Expert 都强调过：

> _Coupling is probably not increased because the classes were already visible to each other due to their associations in the domain model._

这句话完全适用于 total()。

---

# ❗结论：计算 total 不会导致高耦合，书中这么说

1. 协作是正常的
2. 这些类已经有关联关系 → 不是新增耦合
3. 责任分配遵循 Information Expert → 内聚高
4. 耦合保持在必要范围

因此，这是一个教科书式的 **低耦合、高内聚** 的设计。
