# PyTorch Sigmoid 激活函数详解

## 核心思想
Sigmoid 函数是一种常用的非线性激活函数，它将任意实数映射到 $(0, 1)$ 区间。其数学表达式为：
$$ f(x) = \frac{1}{1 + e^{-x}} $$
常用于二分类模型的输出层（表示概率），或在门控机制（如 LSTM、GRU 中的各种门）中控制信息的流通率。

## PyTorch 使用

```python
import torch
import torch.nn as nn

# 1. 作为网络层使用
sigmoid_layer = nn.Sigmoid()
x = torch.tensor([-2.0, 0.0, 2.0])
output = sigmoid_layer(x)

# 2. 函数式 API 调用
import torch.nn.functional as F
output_f = F.sigmoid(x)  # 注意：更推荐直接使用 torch.sigmoid(x)

print("输入:\n", x)
print("输出:\n", output)
```

## 关键要点说明
- **输出范围**：$(0, 1)$，天然适合表示概率或比例（如门控值）。
- **梯度消失问题**：当输入值很大或很小时，Sigmoid 曲线变得非常平缓，导数趋近于 0。这会导致在深层网络反向传播时梯度消失，因此在较深的隐藏层中很少使用。
- **非零中心化**：Sigmoid 的输出恒为正（均值大于 0），这会导致下一层神经元接收到的输入总是非零中心的，影响梯度的更新方向，降低收敛速度。
- **计算成本较高**：包含指数运算 $e^{-x}$，相比 ReLU 计算量更大。
