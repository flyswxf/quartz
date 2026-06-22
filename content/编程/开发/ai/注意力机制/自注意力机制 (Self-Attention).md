## 核心思想
自注意力机制（Self-Attention）的核心在于 **"Self"（自身）**，即 Query、Key、Value 都来自同一个输入序列（例如同一句话的词向量）。
它的目的是发现序列内部各个元素之间的依赖关系，不论这两个元素在序列中相隔多远。相比于传统的 RNN 或 CNN，自注意力能够实现全局视野。

数学上，给定输入矩阵 $X$（形状为 `[seq_len, d_model]`），模型通过三个可学习的线性变换矩阵 $W^Q, W^K, W^V$ 将其映射：
$$ Q = X W^Q, \quad K = X W^K, \quad V = X W^V $$
随后将其代入缩放点积注意力公式中进行计算。

## PyTorch 实现

```python
import torch
import torch.nn as nn
import math

class SelfAttention(nn.Module):
    def __init__(self, d_model):
        super(SelfAttention, self).__init__()
        self.d_model = d_model
        
        # 定义用于生成 Q, K, V 的线性层
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        
    def forward(self, x, mask=None):
        # x 形状: (batch_size, seq_len, d_model)
        
        # 1. 线性映射得到 Q, K, V
        Q = self.W_q(x)
        K = self.W_k(x)
        V = self.W_v(x)
        
        # 2. 计算缩放点积注意力
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_model)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
            
        attn_weights = torch.softmax(scores, dim=-1)
        output = torch.matmul(attn_weights, V)
        
        return output, attn_weights

# 示例使用
batch_size, seq_len, d_model = 2, 10, 512
x = torch.randn(batch_size, seq_len, d_model)
self_attn = SelfAttention(d_model)
output, weights = self_attn(x)
print("输出形状:", output.shape) # [2, 10, 512]
```

## 关键要点说明
- **全局感受野**：在一步计算内，序列中的任何两个词都可以直接发生交互，解决长距离依赖问题。
- **排列不变性 (Permutation Invariance)**：自注意力计算本身是一个集合操作，它**不感知序列的顺序**。如果打乱输入词的顺序，输出结果的对应位置值不会改变。因此，必须引入[[位置编码 (Positional Encoding)]]来提供顺序信息。
- **计算复杂度**：时间复杂度和空间复杂度均为 $O(N^2)$（$N$ 为序列长度）。这使得处理极长序列（如上万 token 的文本或高分辨率图像）时会面临内存爆炸，这也是近年来各类线性注意力 (Linear Attention) 或稀疏注意力 (Sparse Attention) 优化的重点。

## 扩展：自注意力与全连接层 (FC) 的区别
虽然自注意力和全连接层（Linear/Dense Layer）看起来都是“让输出的每一个元素与输入的每一个元素发生关联”，但它们在本质上有以下几个核心区别：

1. **权重的生成方式：动态 (Dynamic) vs 静态 (Static)**
   - **全连接层**：权重矩阵 $W$ 是在训练阶段学习到的，训练完成后就**固定不变**了。不管输入什么样的数据，连接权重都是死的。
   - **自注意力**：注意力权重（打分）是**动态计算**的。对于输入序列中的任意两个词 $i$ 和 $j$，它们之间的关联权重是由它们自身的表示（Query 和 Key）通过点积实时计算出来的。换句话说，自注意力是**“数据依赖的” (Data-dependent)**。

2. **作用的维度：序列交互 vs 特征映射**
   - **全连接层**（在 NLP 中通常是 Token-wise 的）：一般只作用于**特征维度**（$d_{model}$）。即对于序列中的每一个词，独立地对其特征进行线性映射，**词与词之间没有发生信息交互**。
   - **自注意力**：主要作用于**序列维度**（Sequence Dimension）。它的核心使命就是让当前词去“看”序列中的其他词，实现上下文信息的融合。

3. **对序列长度的适应性：可变 vs 固定**
   - **全连接层**：如果想要用全连接层来实现词与词的交互（比如把整个序列展平 Flatten 后输入），那么输入序列的长度必须是**固定**的，因为权重矩阵的大小是固定的。
   - **自注意力**：权重是两两点积算出来的，与序列长度无关。因此自注意力天然支持**任意长度**的输入序列。

4. **位置敏感性：位置无关 vs 位置强绑定**
   - **全连接层**：如果是展平后做 FC，第 1 个位置和第 5 个位置的权重是独立的，模型天然知道绝对位置（位置强绑定）。
   - **自注意力**：自注意力计算本身是一个集合操作，具有**排列不变性**。打乱输入词的顺序，输出结果只是对应位置被打乱，值不会变。这就是为什么 Transformer 必须额外引入[[位置编码 (Positional Encoding)]]。

## 扩展 2：为什么说自注意力是一个“集合操作”？
在数学和计算机科学中，“集合 (Set)”的核心特征是**无序性**（$\{A, B\} = \{B, A\}$），而“序列 (Sequence)”具有**有序性**（$[A, B] \neq [B, A]$）。

我们说自注意力是一个集合操作，是因为**它在计算时，完全忽略了输入元素的物理位置顺序**。具体原因如下：

### 1. 公式的本质是“无序的加权求和”
对于输入序列中的某个词 $i$，自注意力输出 $y_i$ 的计算过程是：
$$ y_i = \sum_{j=1}^{N} \text{softmax}(q_i \cdot k_j) v_j $$
在这个求和公式中，加法是满足交换律的（$a+b = b+a$）。因此，不管输入词 $j$ 是在词 $i$ 的前面还是后面，相隔 1 个位置还是 100 个位置，**只要词 $j$ 的内容（即它的 $k_j$ 和 $v_j$）不变，它对 $y_i$ 的贡献就完全一样**。这就像把所有的词丢进一个没有顺序的“袋子（集合）”里，然后从中按需提取。

### 2. 矩阵视角的“排列等变性” (Permutation Equivariance)
如果我们将输入序列的顺序打乱（例如，将输入的第 1 个词和第 3 个词互换），在自注意力的计算中，输出结果也会发生完全相同的打乱（第 1 个输出和第 3 个输出互换），但是**每一个词计算出的特征向量的值本身没有任何改变**。
用数学语言描述：如果 $P$ 是一个打乱顺序的置换矩阵，那么 $Attention(PX) = P \cdot Attention(X)$。

### 3. 与 RNN 和 CNN 的鲜明对比
- **RNN** 是一个真正的“序列”操作：计算第 $t$ 步的输出必须依赖第 $t-1$ 步的隐藏状态。顺序一旦打乱，整个结果完全面目全非。
- **CNN** 是一个“局部”操作：卷积核只对相邻物理位置的元素（如 $x_{t-1}, x_t, x_{t+1}$）进行计算，它天然感知元素的相对位置。
- **自注意力**：纯粹看“谁和我的点积得分高”，完全不看“谁在我的旁边”。这种打破物理位置限制的特性，就是典型的“集合操作”体现。

### 致命的缺陷与补救 (位置编码)
正因为自注意力把输入当成一个无序的集合，它**本身无法区分“狗咬人”和“人咬狗”**（因为输入给自注意力的词向量集合都是 $\{\text{狗}, \text{咬}, \text{人}\}$）。
为了弥补作为“集合操作”丧失了序列顺序这一致命缺陷，Transformer 必须在输入数据进入自注意力层之前，给每个词向量强行加上[[位置编码 (Positional Encoding)]]，这就相当于给集合里的每个元素贴上了一个写着“我是第几个词”的序号标签，让模型重新感知到顺序。
