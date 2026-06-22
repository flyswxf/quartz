# PyTorch LeakyReLU 激活函数详解

## 核心思想
LeakyReLU 是 ReLU 的一个变体，旨在解决 ReLU 的“神经元死亡”问题。它在输入为负数时，不再直接输出 0，而是赋予一个非常小的非零斜率（如 0.01）。其数学表达式为：
$$ f(x) = \begin{cases} x & \text{if } x > 0 \\ \text{negative\_slope} \times x & \text{if } x \le 0 \end{cases} $$

## PyTorch 使用

```python
import torch
import torch.nn as nn

# 1. 作为网络层使用 (默认 negative_slope=0.01)
leaky_relu_layer = nn.LeakyReLU(negative_slope=0.01, inplace=False)
x = torch.tensor([-2.0, 0.0, 2.0])
output = leaky_relu_layer(x)

# 2. 函数式 API 调用
import torch.nn.functional as F
output_f = F.leaky_relu(x, negative_slope=0.01)

print("输入:\n", x)
print("输出:\n", output)
# 输出: tensor([-0.0200,  0.0000,  2.0000])
```

## 关键要点说明
- **解决 Dead ReLU 问题**：由于负半轴存在微小的梯度（`negative_slope`），即使神经元处于负激活状态，梯度也能反向传播，从而使权重有机会被更新，神经元可以“起死回生”。
- **性能表现**：在许多任务中，LeakyReLU 的表现与 ReLU 相当甚至更好，但由于其负半轴引入了额外的乘法计算，计算量略微增加。
- **参数选择**：`negative_slope` 通常设置为一个小常数（如 0.01）。如果是 PReLU（Parametric ReLU），这个斜率会作为可学习的参数在训练过程中自动调整。
