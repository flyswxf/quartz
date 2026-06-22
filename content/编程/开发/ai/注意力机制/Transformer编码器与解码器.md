
Transformer 架构由编码器（Encoder）和解码器（Decoder）两个核心组件构成。它们通过组合不同的注意力机制来完成特征提取与序列生成任务。

## 编码器 (Encoder)

编码器的主要职责是“理解”输入序列，将其映射为蕴含丰富全局上下文逻辑的连续表示（隐状态向量）。

### 核心技术
- **双向自注意力**：编码器通过[[自注意力机制 (Self-Attention)]]来捕捉序列内部的依赖关系。计算时没有任何掩码限制，序列中的每个位置都可以与所有其他位置进行交互。
- **多头并行**：采用[[多头注意力机制]]，将输入映射到多个子空间中并行计算，以捕捉不同的语义特征。
- **前馈神经网络 (FFN)**：对注意力层的输出进行非线性变换。
- **残差连接与层归一化 (Add & Norm)**：解决深层网络中的梯度消失问题，加速收敛。

### 代码实现结构
在 PyTorch 中，一个典型的编码器层（Encoder Layer）包含自注意力模块和前馈神经网络：

```python
import torch.nn as nn

class EncoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        # 核心组件：多头自注意力
        self.self_attn = nn.MultiheadAttention(d_model, num_heads, batch_first=True)
        self.feed_forward = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Linear(d_ff, d_model)
        )
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, src_mask=None):
        # 1. 多头自注意力 + 残差连接 + 层归一化
        attn_output, _ = self.self_attn(x, x, x, attn_mask=src_mask)
        x = self.norm1(x + self.dropout(attn_output))
        
        # 2. 前馈神经网络 + 残差连接 + 层归一化
        ff_output = self.feed_forward(x)
        x = self.norm2(x + self.dropout(ff_output))
        return x
```

## 解码器 (Decoder)

解码器的主要职责是“生成”输出序列。它通常以自回归的方式运行，即根据已生成的词预测下一个词。

### 核心技术
- **掩码自注意力**：解码器的第一层依然是自注意力层，但为了防止在预测第 $t$ 步时“偷看”到 $t+1$ 步及之后的信息，必须引入[[因果掩码]]（Causal Mask）。
- **交叉注意力**：解码器的第二层是[[交叉注意力机制 (Cross-Attention)]]。此时，Query 来自解码器自身的上一层输出，而 Key 和 Value 则来自**编码器的最终输出**。这是编码器与解码器产生信息交互的桥梁。

### 代码实现结构
解码器层（Decoder Layer）比编码器层多了一个交叉注意力模块：

```python
class DecoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        # 组件1：掩码多头自注意力
        self.self_attn = nn.MultiheadAttention(d_model, num_heads, batch_first=True)
        # 组件2：交叉注意力
        self.cross_attn = nn.MultiheadAttention(d_model, num_heads, batch_first=True)
        
        self.feed_forward = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Linear(d_ff, d_model)
        )
        
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.norm3 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, memory, tgt_mask=None, memory_mask=None):
        # memory 为编码器的最终输出
        
        # 1. 掩码自注意力 (防止看到未来信息)
        attn_output, _ = self.self_attn(x, x, x, attn_mask=tgt_mask)
        x = self.norm1(x + self.dropout(attn_output))
        
        # 2. 交叉注意力 (Q来自解码器x，K和V来自编码器memory)
        cross_output, _ = self.cross_attn(query=x, key=memory, value=memory, attn_mask=memory_mask)
        x = self.norm2(x + self.dropout(cross_output))
        
        # 3. 前馈神经网络
        ff_output = self.feed_forward(x)
        x = self.norm3(x + self.dropout(ff_output))
        return x
```

## 二者的组合与关系
1. **输入预处理**：由于注意力机制本身没有顺序概念，在词嵌入（Token Embedding）输入编码器和解码器之前，都必须先加上[[位置编码 (Positional Encoding)]]，为序列注入顺序信息。
2. **串联关系**：完整的 Transformer 模型先将输入传递给由 $N$ 个 EncoderLayer 组成的编码器，提取出全局特征（通常称为 `memory`）。
3. **信息注入**：随后，由 $N$ 个 DecoderLayer 组成的解码器开始工作，在每一层的交叉注意力计算中，都会将编码器的输出 `memory` 作为 Key 和 Value 注入，指导解码器生成正确的输出序列。