## 核心思想
缩放点积注意力是 Transformer 架构中最核心的基础计算单元。它通过计算 Query (查询) 和 Key (键) 之间的点积来衡量相关性，然后利用这些相关性对 Value (值) 进行加权求和。

其数学公式为：
$$ \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V $$
其中，$Q, K, V$ 分别代表 Query, Key, Value 矩阵；$d_k$ 是 Key 的维度大小。

## PyTorch 实现 (从头编写)

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

def scaled_dot_product_attention(query, key, value, mask=None):
    """
    计算缩放点积注意力
    query, key, value 形状: (batch_size, num_heads, seq_len, d_k)
    """
    d_k = query.size(-1)
    
    # 1. 计算点积并缩放
    # key.transpose(-2, -1) 将最后两个维度转置用于矩阵乘法
    scores = torch.matmul(query, key.transpose(-2, -1)) / math.sqrt(d_k)
    
    # 2. (可选) 掩码操作：在 softmax 前将不需要注意的位置设为极小值
    if mask is not None:
        scores = scores.masked_fill(mask == 0, -1e9)
        
    # 3. 计算注意力权重
    p_attn = F.softmax(scores, dim=-1)
    
    # 4. 乘以 Value
    return torch.matmul(p_attn, value), p_attn

# 示例使用
batch_size, num_heads, seq_len, d_k = 2, 8, 10, 64
q = torch.randn(batch_size, num_heads, seq_len, d_k)
k = torch.randn(batch_size, num_heads, seq_len, d_k)
v = torch.randn(batch_size, num_heads, seq_len, d_k)

output, attn_weights = scaled_dot_product_attention(q, k, v)
print("输出形状:", output.shape) # [2, 8, 10, 64]
```

## 关键要点说明
- **为什么需要缩放 ($\sqrt{d_k}$)？**：当维度 $d_k$ 很大时，点积结果的值域会非常大，导致输入到 Softmax 函数后落入梯度极小的饱和区（梯度消失）。除以 $\sqrt{d_k}$ 可以将方差重新拉回 1，使得梯度更加平稳。
- **Mask (掩码) 机制**：
  - **Padding Mask**：用于忽略序列中为了对齐而填充（Padding）的无效部分（如 `<PAD>` token）。
  - [[因果掩码|Causal/Subsequent Mask (因果掩码)]]
- **PyTorch 内置优化**：在 PyTorch 2.0+ 中，可以使用 `torch.nn.functional.scaled_dot_product_attention`，它底层集成了 FlashAttention 等优化，计算速度极快且节省显存。当参数设为 `is_causal=True` 时，底层会自动应用因果掩码。
