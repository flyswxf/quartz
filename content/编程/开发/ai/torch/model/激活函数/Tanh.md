# PyTorch Tanh 激活函数详解

## 核心思想
Tanh (双曲正切, Hyperbolic Tangent) 激活函数是 Sigmoid 函数的变体，它将任意实数映射到 $(-1, 1)$ 区间。其数学表达式为：
$$ f(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}} $$
Tanh 在形状上与 Sigmoid 类似，但它是零中心的（Zero-centered），这使得它在隐藏层中的表现通常优于 Sigmoid。

## PyTorch 使用

```python
import torch
import torch.nn as nn

# 1. 作为网络层使用
tanh_layer = nn.Tanh()
x = torch.tensor([-2.0, 0.0, 2.0])
output = tanh_layer(x)

# 2. 函数式 API 调用
import torch.nn.functional as F
output_f = F.tanh(x) # 也可以直接使用 torch.tanh(x)

print("输入:\n", x)
print("输出:\n", output)
```

## 关键要点说明
- **零中心化**：输出范围是 $(-1, 1)$，均值接近 0。这解决了 Sigmoid 非零中心化的问题，使得后续网络层的权重更新更加稳定，收敛速度通常比 Sigmoid 快。
- **梯度消失问题**：与 Sigmoid 相同，当输入值很大或很小时，梯度会趋近于 0，依然存在梯度消失的问题。
- **应用场景**：在循环神经网络（RNN、LSTM）的内部状态计算中非常常见；如果需要在二分类或概率外使用饱和型激活函数，通常首选 Tanh 而非 Sigmoid。
