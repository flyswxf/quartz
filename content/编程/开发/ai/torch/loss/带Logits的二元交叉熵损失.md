# 带 Logits 的二元交叉熵损失 (Binary Cross Entropy With Logits Loss)

`nn.BCEWithLogitsLoss()` 是 PyTorch 中做 **二分类** 和 **多标签分类** 时非常常用的损失函数。

它可以看作：
$$ \text{BCEWithLogitsLoss} = \text{Sigmoid} + \text{BCELoss} $$

也就是说，它会先把模型输出的原始分数 `logits` 通过 `Sigmoid` 变成概率，再计算二元交叉熵。

相比 `nn.BCELoss()`，它的最大优势是：**数值更稳定，训练时通常更推荐直接使用它。**

## 1. 为什么需要它？

### 1.1 什么是 logits？
`logits` 指模型最后一层输出的原始分数，它们还不是概率，可以是任意实数：
$$ z \in (-\infty, +\infty) $$

通过 `Sigmoid` 可以把它变成概率：
$$ p = \sigma(z) = \frac{1}{1 + e^{-z}} $$

然后就能把这个概率送入 BCE：
$$ \text{BCE}(y,p) = -\left[y \log p + (1-y)\log(1-p)\right] $$

### 1.2 为什么不手动 `Sigmoid` 再用 `BCELoss`？
数学上当然可以这么做，但工程上有个重要问题：**数值稳定性**。

当 logits 很大或很小时：
- `Sigmoid(z)` 会非常接近 0 或 1
- 再取 `\log` 时容易出现数值下溢或精度问题

`nn.BCEWithLogitsLoss()` 在底层把这些步骤合并后做了稳定化处理，因此更安全。

### 1.3 它的理论本质是什么？
它的理论本质和 BCE 一样，仍然来自 **伯努利分布的负对数似然**。

区别只是：
- `BCELoss` 的输入是“概率”
- `BCEWithLogitsLoss` 的输入是“未过 Sigmoid 的原始分数”

所以它不是一种全新的损失，而是 **BCE 的稳定实现版本**。

---

## 2. 为什么它有效？有什么特点？

### 2.1 二分类任务里的默认优选
如果模型最后输出一个 logit：
- logit 大于 0，表示更偏向正类
- logit 小于 0，表示更偏向负类
- logit 等于 0，对应概率 0.5

这种表示方式很自然，也方便神经网络直接学习。

### 2.2 数值稳定性更好
这是它最重要的工程优势。

例如：
- 若 $z = 20$，则 `Sigmoid(z)` 非常接近 1
- 若 $z = -20$，则 `Sigmoid(z)` 非常接近 0

如果先单独做 `Sigmoid`，再取对数，就容易有精度问题；而 `BCEWithLogitsLoss` 会在内部使用更稳定的公式来计算。

### 2.3 适合多标签分类
它不仅能做单输出二分类，还很适合 **多标签分类**。

例如一张图片可以同时有：
- 猫
- 狗
- 沙发

这时每个标签都独立做一个二分类，输出多个 logits，再使用 `BCEWithLogitsLoss()`。

### 2.4 支持类别不平衡加权
它支持 `pos_weight` 等参数来提高正类样本的损失权重，因此在正负样本极不平衡时非常实用。

---

## 3. PyTorch 中的 `nn.BCEWithLogitsLoss()`

### 3.1 使用方法

**接受参数：**
- `input`: 原始 logits，未经过 `Sigmoid`
- `target`: 真实标签，通常取值为 `0` 或 `1`，形状与 `input` 相同

```python
import torch
from torch import nn

# 原始输出分数（logits）
logits = torch.tensor([2.0, -1.0, 0.5])
targets = torch.tensor([1.0, 0.0, 1.0])

criterion = nn.BCEWithLogitsLoss()
loss = criterion(logits, targets)

print("BCEWithLogits 损失:", loss.item())
```

### 3.2 📐 数学理解

先经过 `Sigmoid`：
$$ p_i = \sigma(z_i) = \frac{1}{1 + e^{-z_i}} $$

再代入 BCE：
$$ \text{Loss} = \frac{1}{N} \sum_{i=1}^{N} -\left[ y_i \log p_i + (1-y_i)\log(1-p_i) \right] $$

这里只是把 $p_i$ 换成了由 logits 计算出的概率。

### 3.3 📝 举例说明

假设某个样本的 logit 为：
$$ z = 2.0 $$

则对应概率：
$$ p = \sigma(2.0) \approx 0.881 $$

如果真实标签是正类 $y=1$，则损失为：
$$ \text{loss} = -\log(0.881) \approx 0.127 $$

说明模型已经比较正确，所以损失较小。

如果另一个样本的 logit 为：
$$ z = -2.0 $$

则：
$$ p = \sigma(-2.0) \approx 0.119 $$

若真实标签仍然是正类 $y=1$，则：
$$ \text{loss} = -\log(0.119) \approx 2.127 $$

说明模型不仅预测错了，而且还比较自信，因此损失明显更大。

---

## 4. 和 `nn.BCELoss()` 的区别

- `nn.BCELoss()`：输入必须是概率，通常要手动先做 `Sigmoid`
- `nn.BCEWithLogitsLoss()`：输入直接是 logits，内部自动处理 `Sigmoid`
- 实际训练中：**通常优先使用 `nn.BCEWithLogitsLoss()`**

---

## 5. 一句话理解

`nn.BCEWithLogitsLoss()` 的本质是：

**用更稳定的方式，把原始输出分数先映射成概率，再计算二元交叉熵。它是二分类训练里的常用默认选择。**
