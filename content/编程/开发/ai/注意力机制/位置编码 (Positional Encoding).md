# 位置编码 (Positional Encoding)

## 核心思想
Transformer 完全基于注意力机制，没有像 RNN 那样的循环结构，也没有 CNN 那样的局部感受野。因此，原生的 Transformer 无法感知输入序列中词语的**顺序**或**位置信息**（即它具有置换不变性）。
为了让模型理解 "A 欠 B 钱" 和 "B 欠 A 钱" 的区别，必须显式地将位置信息注入到输入中，这就是**位置编码**的作用。

通常，位置编码向量会与词嵌入（Token Embedding）直接**相加**，然后再输入到模型中：
$$ \text{Input} = \text{TokenEmbedding} + \text{PositionalEncoding} $$

## 两大阵营：绝对位置编码 vs 相对位置编码

在技术演进中，位置编码主要分为两大流派：绝对位置编码和相对位置编码。

### 绝对位置编码 (Absolute Positional Encoding)
**核心思想**：为序列中的每个绝对物理位置（第 1 个词、第 2 个词...）分配一个唯一的向量。
- **操作方式**：通常在输入层，直接将位置向量与词嵌入向量**相加**。
- **代表方案**：
  - [[位置编码 (Positional Encoding)#正弦波绝对位置编码 (Sinusoidal Positional Encoding)|正弦波位置编码]]（Transformer 原论文采用，无需训练）。
  - **可学习绝对位置编码 (Learned Positional Embedding)**：初始化一个形状为 `(max_len, d_model)` 的参数矩阵，让模型在训练中自己学习每个位置的表示。BERT 和早期的 GPT 系列广泛采用。
- **优缺点**：实现简单，但泛化到超过训练长度的序列（外推性）表现较差，且未显式建模词与词之间的相对距离。

### 相对位置编码 (Relative Positional Encoding)
**核心思想**：语言理解中，词与词之间的**相对距离**（如“词 A 在词 B 前面 3 个位置”）往往比绝对位置（如“词 A 是句子中的第 10 个词”）更重要。相对位置编码致力于在注意力计算时直接注入这种相对距离信息。
- **操作方式**：不在输入层相加，而是在[[自注意力机制 (Self-Attention)]]的**打分（Dot-Product）环节**或**权重环节**做手脚。
- **代表方案**：
  - **Transformer-XL / T5**：在计算 $QK^T$ 的公式中，将绝对位置展开，替换为基于相对距离的可学习偏置项。
  - **ALiBi (Attention with Linear Biases)**：非常简单粗暴，直接在计算出的注意力打分矩阵上，减去一个与距离成正比的惩罚项（距离越远，惩罚越大）。
  - **RoPE (旋转位置编码, Rotary Position Embedding)**：目前大模型（如 Llama, ChatGLM）的绝对主流。它通过旋转 Query 和 Key 的向量来注入**绝对位置**，但由于旋转矩阵的数学性质，两个向量点积后的结果天然只依赖于它们的**相对距离**，完美融合了绝对和相对的优势。

---

## 正弦波绝对位置编码 (Sinusoidal Positional Encoding)
在原论文《Attention Is All You Need》中，使用不同频率的正弦和余弦函数来计算绝对位置编码。

公式如下：
$$ PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right) $$
$$ PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right) $$
- $pos$：当前词在序列中的位置（如 0, 1, 2...）
- $i$：维度的索引（如 $0 \le i < d_{\text{model}}/2$）
- $d_{\text{model}}$：模型的隐藏层维度大小

### 为什么使用正弦和余弦？
1. **相对位置表达**：对于任意固定的偏移量 $k$，$PE_{pos+k}$ 可以表示为 $PE_{pos}$ 的线性函数。这使得模型很容易学习到相对位置关系。
2. **泛化能力强**：即使推理时的序列长度超过了训练时的最大长度，正弦函数也能通过周期性计算出对应的编码，具有外推性。

## PyTorch 实现
```python
import torch
import torch.nn as nn
import math

class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000):
        super().__init__()
        
        # 创建一个足够长的位置编码矩阵
        pe = torch.zeros(max_len, d_model)
        # pos: (max_len, 1)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        
        # 计算 10000^(2i/d_model)
        # div_term: (d_model/2,)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        
        # 偶数维度应用 sin，奇数维度应用 cos
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        
        # 扩展 batch 维度: (1, max_len, d_model)
        pe = pe.unsqueeze(0)
        
        # 注册为 buffer，这样它不会被视为模型参数进行更新，但会随模型保存
        self.register_buffer('pe', pe)

    def forward(self, x):
        # x 形状: (batch_size, seq_len, d_model)
        seq_len = x.size(1)
        # 截取对应长度的位置编码，并加到词嵌入上
        x = x + self.pe[:, :seq_len, :]
        return x
```

