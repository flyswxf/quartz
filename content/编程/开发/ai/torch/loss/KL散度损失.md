# KL 散度损失 (Kullback-Leibler Divergence Loss)

`nn.KLDivLoss()` 用来衡量两个概率分布之间的差异。

如果说交叉熵是在问：
“模型有没有把高概率给到真实标签？”

那么 KL 散度更像是在问：
“模型预测出的整个分布，和目标分布到底差了多少？”

它常用于：
- 知识蒸馏
- 概率分布匹配
- 变分推断
- 与 soft label 相关的训练任务

## 1. 理论基础：KL 散度是什么？

### 1.1 两个分布的差异
设：
- $P$ 是真实分布
- $Q$ 是模型预测分布

KL 散度定义为：
$$ D_{KL}(P \parallel Q) = \sum_x P(x)\log\frac{P(x)}{Q(x)} $$

它衡量的是：

**如果真实世界遵循 $P$，但我们用 $Q$ 来近似描述它，会损失多少信息。**

### 1.2 和交叉熵的关系
把公式展开：
$$ D_{KL}(P \parallel Q) = \sum_x P(x)\log P(x) - \sum_x P(x)\log Q(x) $$

可写成：
$$ D_{KL}(P \parallel Q) = -H(P) + H(P, Q) $$

其中：
- $H(P)$ 是真实分布自身的熵
- $H(P,Q)$ 是交叉熵

所以：

**KL 散度 = 交叉熵 - 真实分布的熵**

当真实分布固定时，最小化 KL 散度与最小化交叉熵是等价的。

### 1.3 它不是普通“距离”
KL 散度虽然常被叫做“距离”，但严格来说它不是数学意义上的距离：
- 不对称：$D_{KL}(P\parallel Q)\neq D_{KL}(Q\parallel P)$
- 不满足三角不等式

因此它更像是“信息损失度量”。

---

## 2. 为什么它有效？有什么特点？

### 2.1 它适合软标签
交叉熵常用于 one-hot 标签，而 KL 散度非常适合目标本身就是一个分布的情况。

例如知识蒸馏中，教师模型给出的标签可能不是：
```text
[0, 1, 0]
```

而是：
```text
[0.1, 0.7, 0.2]
```

这时 KL 散度比普通交叉熵更自然，因为它是在直接匹配两个分布。

### 2.2 它衡量“整体分布”的差异
NLLLoss 或 CrossEntropyLoss 更强调真实标签那一项；而 KL 散度会考虑整个分布上每个位置的差异。

这在分布学习任务中非常重要。

### 2.3 它对方向敏感
由于 KL 散度不对称，使用时必须注意：
- $D_{KL}(P\parallel Q)$
- $D_{KL}(Q\parallel P)$

这两个量通常不同，训练行为也会不同。

### 2.4 在 PyTorch 中输入形式容易混淆
这是 `nn.KLDivLoss()` 最常见的坑。

PyTorch 里通常要求：
- `input`: 是模型输出的 **log-probabilities**
- `target`: 是目标概率分布

也就是经常要手动写：
```python
log_probs = torch.log_softmax(logits, dim=1)
```

---

## 3. PyTorch 中的 `nn.KLDivLoss()`

### 3.1 使用方法

**接受参数：**
- `input`: 对数概率 `log_probs`
- `target`: 目标概率分布

```python
import torch
from torch import nn

logits = torch.tensor([[2.0, 0.5, 0.1]])
log_probs = torch.log_softmax(logits, dim=1)

target = torch.tensor([[0.1, 0.7, 0.2]])

criterion = nn.KLDivLoss(reduction="batchmean")
loss = criterion(log_probs, target)

print("KL 散度损失:", loss.item())
```

### 3.2 📐 数学公式

对于离散分布：
$$ D_{KL}(P \parallel Q) = \sum_{j=1}^{C} P_j \log\frac{P_j}{Q_j} $$

如果在 PyTorch 中 `input` 传入的是 $\log Q_j$，则计算时本质上是在做：
$$ \sum_{j=1}^{C} P_j(\log P_j - \log Q_j) $$

### 3.3 📝 举例说明

假设目标分布为：
$$ P = [0.1, 0.7, 0.2] $$

模型预测分布为：
$$ Q = [0.2, 0.6, 0.2] $$

则：
$$
D_{KL}(P\parallel Q)
= 0.1\log\frac{0.1}{0.2}
+ 0.7\log\frac{0.7}{0.6}
+ 0.2\log\frac{0.2}{0.2}
$$

最后一项为 0，因为：
$$ \log \frac{0.2}{0.2} = \log 1 = 0 $$

这个值越小，说明模型分布越接近目标分布。

---

## 4. 典型应用

- **知识蒸馏**：学生模型拟合教师模型输出的 soft label
- **变分自编码器（VAE）**：约束潜变量分布接近先验分布
- **语言模型分布匹配**：比较两个概率分布的差异

---

## 5. 一句话理解

`nn.KLDivLoss()` 的本质是：

**不只看“真实类对不对”，而是看整个预测分布和目标分布之间到底差了多少信息。**
