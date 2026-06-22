# PyTorch Softmax 激活函数详解

## 核心思想

Softmax 激活函数用于多分类问题，它将一个实数向量映射为一个概率分布。其数学表达式为：
$$ \text{Softmax}(x*i) = \frac{e^{x_i}}{\sum*{j} e^{x_j}} $$
它会将输入的每个分量转化为 $(0, 1)$ 之间，且所有分量之和恒为 1，非常适合作为分类模型的最后一层输出。

## PyTorch 使用

```python
import torch
import torch.nn as nn

# 1. 作为网络层使用 (指定维度 dim)
softmax_layer = nn.Softmax(dim=1)
x = torch.tensor([[1.0, 2.0, 3.0]]) # 模拟一个 batch 中包含 3 个类别的得分
output = softmax_layer(x)

# 2. 函数式 API 调用
import torch.nn.functional as F
output_f = F.softmax(x, dim=1)

print("输入:\n", x)
print("输出概率分布:\n", output)
print("概率和:\n", output.sum(dim=1)) # 恒为 1.0
```

## 内部实现（数值稳定性）

由于直接计算指数极易导致数值溢出（Overflow），实际框架在底层实现时会进行平移操作，即减去输入向量中的最大值 $M = \max(x)$，其数学等价变形为：
$$ \text{Softmax}(x*i) = \frac{e^{x_i - M}}{\sum*{j} e^{x_j - M}} $$

可以用纯 Python/PyTorch 张量操作模拟其内部实现：

```python
def custom_softmax(x, dim=-1):
    # 1. 找到指定维度上的最大值（保持维度以便广播）
    x_max = torch.max(x, dim=dim, keepdim=True)[0]

    # 2. 减去最大值并计算指数
    x_exp = torch.exp(x - x_max)

    # 3. 计算指数和（保持维度以便广播）
    x_sum = torch.sum(x_exp, dim=dim, keepdim=True)

    # 4. 归一化得到概率分布
    return x_exp / x_sum

# 测试与官方实现对比
x_test = torch.tensor([[1.0, 2.0, 300.0]]) # 模拟包含较大数值的输入，直接计算 e^300 会溢出
print("自定义实现:\n", custom_softmax(x_test, dim=1))
print("官方实现:\n", F.softmax(x_test, dim=1))
```

## 关键要点说明

- **数值稳定性**：指数计算极易导致数值溢出（OverFlow）。PyTorch 的实现通常会在分子分母同除以最大值的指数，以确保数值稳定。
- **与交叉熵损失结合**：在多分类任务中，**强烈建议直接使用 `nn.CrossEntropyLoss`**，因为它内部已经包含了 `nn.LogSoftmax` 和 `nn.NLLLoss`。如果在网络最后一层手动加上 `nn.Softmax` 再传给交叉熵损失，会导致重复计算且降低数值稳定性。
- **多维度计算（`dim` 参数）**：`dim` 的选择非常关键。如果是全连接层的输出 `[batch_size, num_classes]`，则应选择 `dim=1`；如果是图像分割任务的输出 `[batch_size, num_classes, H, W]`，通常在 `dim=1`（通道维度）上计算。
