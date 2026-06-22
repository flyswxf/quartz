## 核心思想
交叉注意力（Cross-Attention）是一种将两个不同序列（模态或来源）的信息进行融合的机制。它与自注意力机制的公式完全相同，区别仅在于 Query、Key、Value 的**来源不同**。

在典型的 Encoder-Decoder 架构（如 Transformer 机器翻译模型、Stable Diffusion 的文本到图像生成）中：
- **Query (Q)**：通常来自于当前正在处理的主序列（例如：Decoder 中的目标语言翻译进度，或图像生成的特征图）。
- **Key (K) 和 Value (V)**：通常来自于另一个提供上下文条件的辅助序列（例如：Encoder 输出的源语言特征，或文本提示词的 Embedding）。

核心作用是：**让主序列（Q）主动去另一序列（K, V）中"寻找"对自己有用的信息并提取过来。**

## PyTorch 使用

在 PyTorch 中，交叉注意力的实现依然是使用 `nn.MultiheadAttention`，仅仅是在前向传播时，传入的 `query` 与 `key/value` 张量不同。

```python
import torch
import torch.nn as nn

d_model = 512
num_heads = 8
mha = nn.MultiheadAttention(embed_dim=d_model, num_heads=num_heads, batch_first=True)

batch_size = 2
# 假设这是 Decoder 端生成的当前状态 (例如长度为 5 的目标序列)
target_seq_len = 5
query = torch.randn(batch_size, target_seq_len, d_model) 

# 假设这是 Encoder 端输出的上下文信息 (例如长度为 10 的源序列)
source_seq_len = 10
key = torch.randn(batch_size, source_seq_len, d_model)
value = torch.randn(batch_size, source_seq_len, d_model) # 通常 Key 和 Value 是同一个张量

# 执行交叉注意力计算
# 注意力去寻找：对于目标序列的每个元素，源序列中哪些元素最重要？
attn_output, attn_weights = mha(query, key, value)

print("输出形状:", attn_output.shape)    # [2, 5, 512] (与 query 的长度保持一致)
print("权重形状:", attn_weights.shape)   # [2, 5, 10]  (目标序列中的每个词，对源序列 10 个词的注意力分布)
```

## 关键要点说明
- **输出形状取决于 Query**：在交叉注意力中，输出张量的序列长度和特征维度都与传入的 `query` 保持一致。`key` 和 `value` 的序列长度可以不同，但必须一致。
- **多模态融合的桥梁**：在多模态大模型（如图文大模型、Stable Diffusion）中，文本提示词通常作为 K 和 V，图像特征作为 Q，通过交叉注意力实现文本对图像生成的精准引导。
- **不对称性**：交叉注意力是有方向性的，$Attention(X, Y, Y)$ 和 $Attention(Y, X, X)$ 代表着完全不同的含义（谁去关注谁）。
