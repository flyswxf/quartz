# torch.optim.Optimizer

`Optimizer`（优化器）负责在反向传播计算出梯度后，根据特定的优化算法更新神经网络的参数，以最小化损失函数。

### 核心方法

优化器的典型工作流依赖于以下两个核心方法：
1. `zero_grad()`: 清空模型所有参数的梯度。由于 PyTorch 默认会累加梯度，因此在每次反向传播（`loss.backward()`）之前，必须手动将梯度清零。
2. `step()`: 根据计算得到的梯度和优化算法（如 SGD, Adam），更新模型的参数。

### 常用优化器

- **SGD (Stochastic Gradient Descent)**: 随机梯度下降，支持动量（Momentum）和权重衰减（Weight Decay）。
- **Adam (Adaptive Moment Estimation)**: 自适应学习率优化算法，收敛速度快，是目前深度学习中最常用的默认优化器之一。

### 📝 举例说明

展示一个完整的优化步骤，包含前向传播、计算损失、反向传播和参数更新。

```python
import torch
from torch import nn
from torch import optim

# 1. 准备模型、数据和损失函数
model = nn.Linear(10, 2)
inputs = torch.randn(5, 10)
targets = torch.randn(5, 2)
criterion = nn.MSELoss()

# 2. 实例化优化器，传入需要更新的模型参数和学习率 (lr)
optimizer = optim.Adam(model.parameters(), lr=0.01)

# --- 训练循环的单次迭代 ---

# 步骤 A: 梯度清零
optimizer.zero_grad()

# 步骤 B: 前向传播与损失计算
outputs = model(inputs)
loss = criterion(outputs, targets)

# 步骤 C: 反向传播，计算每个参数的梯度
loss.backward()

# 步骤 D: 更新参数
optimizer.step()

print("单次迭代完成，参数已更新。损失值:", loss.item())
```