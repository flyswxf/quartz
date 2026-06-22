# torch.utils.data.DataLoader

`DataLoader` 是 PyTorch 中用于加载数据的迭代器。它将 `Dataset` 包装起来，提供批处理（Batching）、数据打乱（Shuffling）以及多进程并行加载等功能。

### 核心功能

- **批处理 (Batching)**: 将数据集分成多个小批次（Mini-batches），以便进行批量梯度下降。
- **打乱数据 (Shuffling)**: 在每个 Epoch 开始时打乱数据顺序，有助于提高模型的泛化能力。
- **并行加载 (Multiprocessing)**: 使用多个 worker 进程异步加载数据，避免数据加载成为训练的性能瓶颈。

### 常用参数

- `dataset`: 要加载的数据集实例（继承自 `Dataset`）。
- `batch_size`: 每个批次包含的样本数。
- `shuffle`: 是否在每个 Epoch 开始前打乱数据（训练集通常设为 `True`，测试集设为 `False`）。
- `num_workers`: 用于数据加载的子进程数量（默认为 0，表示在主进程中加载）。

### 📝 举例说明

结合自定义或内置的 `Dataset`，展示如何使用 `DataLoader` 进行批量读取。

```python
import torch
from torch.utils.data import DataLoader, TensorDataset

# 创建一个简单的张量数据集
features = torch.randn(100, 10)  # 100个样本，每个样本10个特征
labels = torch.randint(0, 2, (100,)) # 100个二分类标签
dataset = TensorDataset(features, labels)

# 实例化 DataLoader
dataloader = DataLoader(
    dataset=dataset,
    batch_size=32,     # 每个批次 32 个样本
    shuffle=True,      # 打乱数据
    num_workers=0      # 使用主进程加载
)

# 迭代读取数据
for batch_idx, (batch_features, batch_labels) in enumerate(dataloader):
    print(f"Batch {batch_idx}:")
    print(f"  特征形状: {batch_features.shape}")
    print(f"  标签形状: {batch_labels.shape}")
    
    # 只打印前两个 batch 作为演示
    if batch_idx == 1:
        break
```