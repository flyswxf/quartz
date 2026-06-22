# FSDP 与 ZeRO 显存优化

FSDP (Fully Sharded Data Parallel) 和 ZeRO (Zero Redundancy Optimizer) 是当前大语言模型（LLM）分布式训练中最主流的显存优化技术。它们**结合了数据并行的易用性和模型并行的显存效率**。

## 一、核心思想

在传统的分布式数据并行（DDP）中，每张 GPU 都必须保存一份**完整的**模型状态（包含模型参数、梯度、优化器状态）。对于百亿参数的大模型，单卡显存甚至无法装下这一份模型状态，更别提进行训练了。

ZeRO 提出了**状态分片（Sharding）**的概念，将完整的模型状态切分到各个 GPU 上，消除冗余：
- **ZeRO-1**：仅切分优化器状态（Optimizer States）。
- **ZeRO-2**：切分优化器状态 + 梯度（Gradients）。
- **ZeRO-3**：切分优化器状态 + 梯度 + 模型参数（Parameters）。在 ZeRO-3 下，单卡不再拥有完整模型，只在需要计算某一层时，通过网络通信临时拉取缺失的参数，计算完毕后立刻丢弃。

**FSDP** 是 PyTorch 官方受 ZeRO 启发推出的全切片数据并行方案，在功能上大致等效于 ZeRO-3，且深度集成在 PyTorch 生态中。

## 二、PyTorch 使用

使用 PyTorch 原生的 FSDP 非常简单，代码侵入性远小于传统的张量并行，只需用 `FSDP` 包装（Wrap）模型即可。

```python
import torch
import torch.nn as nn
import torch.optim as optim
import torch.distributed as dist
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP

# 1. 初始化分布式环境
dist.init_process_group(backend="nccl")
local_rank = dist.get_rank()
torch.cuda.set_device(local_rank)

# 2. 实例化模型并包装为 FSDP
model = nn.Sequential(
    nn.Linear(1024, 4096),
    nn.ReLU(),
    nn.Linear(4096, 1024)
).cuda(local_rank)

# FSDP 会自动将模型参数切分到当前进程组的所有 GPU 上
fsdp_model = FSDP(model)

# 3. 初始化优化器 (传入的是 FSDP 包装后的参数)
optimizer = optim.Adam(fsdp_model.parameters(), lr=1e-3)

# 4. 正常训练循环
for data, label in dataloader:
    data, label = data.cuda(local_rank), label.cuda(local_rank)
    
    optimizer.zero_grad()
    # 前向传播时，FSDP 会自动通过 All-Gather 收集当前所需的参数
    output = fsdp_model(data)
    loss = criterion(output, label)
    
    # 反向传播时，计算完梯度后会自动 Reduce-Scatter 释放参数并同步切片梯度
    loss.backward()
    optimizer.step()
```

## 三、关键要点说明

- **显存与通信的权衡**：ZeRO/FSDP 用大量的网络通信（All-Gather 和 Reduce-Scatter）换取了显存空间的释放。节点间的网络带宽（如 NVLink、InfiniBand）是决定训练速度的瓶颈。
- **Auto Wrapping**：对于深层复杂模型，通常不能只在外层包一个 `FSDP()`，而是需要配置 `auto_wrap_policy`（例如按 Transformer Block 进行包装），使得参数可以在每一层计算后及时释放。
- **与模型并行的对比**：张量并行（TP）需要重写模型的前向逻辑，极度依赖 NVLink，通常限于单机内；而 FSDP 可以在多机多卡间扩展，代码无需大幅修改，是训练百亿级大模型的首选基础策略。
