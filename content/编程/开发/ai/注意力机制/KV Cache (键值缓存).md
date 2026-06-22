# KV Cache (键值缓存)

## 核心思想
KV Cache 是大语言模型（如 GPT 系列）在**推理（自回归生成）阶段**广泛使用的一种“以空间换时间”的加速技术。

它的核心目的是：在生成序列的下一个词时，**缓存历史 Token 对应的 Key ($K$) 和 Value ($V$) 向量**，从而避免对已经处理过的历史文本进行重复的矩阵乘法计算。

## 为什么需要 KV Cache？
在[[Transformer编码器与解码器#解码器 (Decoder)|解码器]]中，文本是**自回归**（Autoregressive）生成的，即“根据前 $t-1$ 个词，预测第 $t$ 个词”。

- **不使用 KV Cache（朴素计算）**：
  在生成第 $t$ 个词时，需要把前 $t-1$ 个词拼接起来重新输入模型，重新通过线性层计算所有 $t-1$ 个词的 $Q, K, V$。
  实际上，前 $t-2$ 个词的 $K$ 和 $V$ 在上一步就已经计算过了，这种每次都“推倒重来”的做法会导致大量的 $O(N^2)$ 冗余计算。

- **使用 KV Cache**：
  在生成第 $t$ 个词时，**只需要将上一步新生成的 1 个词输入模型**。
  利用这 1 个词映射出的 $Q$，去和缓存在显存中的历史 $K$ 矩阵进行[[注意力打分机制]]计算，并对历史 $V$ 矩阵加权求和。最后，将当前这个新词的 $K$ 和 $V$ 追加（Concat）到缓存中，供下一步使用。

## 伪代码演示
```python
import torch

def generate_with_kv_cache(model, input_ids, max_new_tokens=50):
    """
    带有 KV Cache 的自回归生成伪代码
    """
    # 初始状态：KV Cache 为空
    past_key_values = None  
    
    for _ in range(max_new_tokens):
        # 如果有缓存，说明是生成阶段，只需输入序列的最后一个 Token
        # 否则说明是第一次 Prefill (预填充) 阶段，需要输入完整 Prompt
        current_input = input_ids[:, -1:] if past_key_values is not None else input_ids
        
        # 模型前向传播，返回预测的 logits 和 更新后的 KV Cache
        logits, past_key_values = model(
            input_ids=current_input, 
            past_key_values=past_key_values # 传入历史缓存
        )
        
        # 预测下一个词
        next_token = torch.argmax(logits[:, -1, :], dim=-1, keepdim=True)
        
        # 将新词拼接到输入序列中，准备下一轮循环
        input_ids = torch.cat([input_ids, next_token], dim=-1)
        
    return input_ids
```

## 显存挑战与衍生优化技术
KV Cache 极大地降低了计算的时间复杂度，但代价是**极高的显存占用**。随着序列长度（Context Length）和批次大小（Batch Size）的增加，KV Cache 的体积会呈线性膨胀，成为制约大模型推理并发量的“显存墙”（Memory Wall）。

为了缓解 KV Cache 带来的显存压力，业界在[[多头注意力机制]]的基础上，演化出了以下几种关键架构与技术：

1. **MQA (Multi-Query Attention)**：
   让所有的注意力头（Heads）共享同一组 $K$ 和 $V$。这能将 KV Cache 的大小直接除以头数 $h$，极大节省显存，但可能会轻微牺牲模型的表达能力。
2. **GQA (Grouped-Query Attention)**：
   介于标准多头注意力和 MQA 之间的一种折中方案。将注意力头进行分组，**组内共享**一组 $K$ 和 $V$。在节省显存和保持模型性能之间取得了极佳的平衡（Llama 2、Llama 3 等主流开源模型均采用此架构）。
3. **PagedAttention**：
   受操作系统虚拟内存中“分页”机制的启发，将 KV Cache 划分为固定大小的块（Block），允许在物理显存中非连续存储。这彻底解决了长文本生成时 KV Cache 显存碎片化严重的问题，是推理框架 vLLM 能够实现极高吞吐量的核心技术。