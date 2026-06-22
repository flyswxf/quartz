# PyTorch GELU 激活函数详解

## 核心思想
GELU (Gaussian Error Linear Unit) 是一种结合了 Dropout 和 ReLU 思想的高级激活函数，广泛应用于基于 Transformer 的模型（如 BERT、GPT、ViT）。其数学表达式基于标准正态分布的累积分布函数（CDF）：
$$ f(x) = x \Phi(x) $$
其中 $\Phi(x) = P(X \le x), X \sim \mathcal{N}(0, 1)$。由于误差函数计算复杂，常采用近似形式：
$$ f(x) \approx 0.5 x \left(1 + \tanh\left(\sqrt{2/\pi}(x + 0.044715 x^3)\right)\right) $$

## PyTorch 使用

```python
import torch
import torch.nn as nn

# 1. 作为网络层使用 (默认无近似)
gelu_layer = nn.GELU(approximate='none')
# 可以使用近似计算加速: nn.GELU(approximate='tanh')
x = torch.tensor([-2.0, 0.0, 2.0])
output = gelu_layer(x)

# 2. 函数式 API 调用
import torch.nn.functional as F
output_f = F.gelu(x, approximate='none')

print("输入:\n", x)
print("输出:\n", output)
```

## 关键要点说明
- **平滑且非单调**：GELU 是非单调的，并且在 $x=0$ 附近是平滑的（连续可导）。相比 ReLU，这为模型提供了更复杂的非线性表达能力。
- **结合了随机正则化**：可以将 GELU 视为对输入进行随机 Dropout（根据输入自身的大小决定保留概率），这使得它在自注意力机制等复杂结构中表现极佳。
- **计算代价**：精确计算 GELU 需要计算误差函数（erf），这比 ReLU 和 LeakyReLU 都要耗时。因此在许多库（包括早期的 PyTorch）和硬件中，会使用 `tanh` 近似形式来加速计算。
- **Transformer 的标配**：几乎所有主流的 NLP（GPT系列, BERT, RoBERTa）和 CV（Vision Transformer）前沿模型都默认使用 GELU 作为前馈神经网络（FFN/MLP）的激活函数。
