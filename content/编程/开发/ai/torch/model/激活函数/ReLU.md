# PyTorch ReLU 激活函数详解

## 核心思想
ReLU (Rectified Linear Unit) 是目前深度学习中最常用的激活函数。它的数学表达式非常简单：
$$ f(x) = \max(0, x) $$
即：当输入大于 0 时，直接输出该值；当输入小于或等于 0 时，输出 0。
它的主要作用是引入非线性，同时缓解梯度消失问题（在正半轴梯度恒为 1），并且计算速度非常快。

## PyTorch 使用

```python
import torch
import torch.nn as nn

# 1. 作为网络层使用 (推荐在 nn.Sequential 中使用)
relu_layer = nn.ReLU(inplace=False)
x = torch.randn(2, 3)
output = relu_layer(x)

# 2. 函数式 API 调用 (推荐在 forward 函数中直接使用)
import torch.nn.functional as F
output_f = F.relu(x, inplace=False)

print("输入:\n", x)
print("输出:\n", output)
```

## 关键要点说明
- **计算效率高**：仅涉及简单的阈值判断，没有复杂的指数运算。
- **缓解梯度消失**：正半轴导数恒为 1，使得深层网络的梯度能够有效反向传播。
- **Dead ReLU 问题（神经元死亡）**：当输入持续为负时，输出始终为 0，对应的权重梯度也为 0，导致该神经元无法再更新。如果学习率设置过大，可能会导致大量神经元“死亡”。
- **`inplace=True`**：在 PyTorch 中，设置 `inplace=True` 可以原地修改张量，节省内存，但可能会影响某些需要原始输入值计算梯度的反向传播操作（通常系统会自动报错，若无报错则可放心使用）。
