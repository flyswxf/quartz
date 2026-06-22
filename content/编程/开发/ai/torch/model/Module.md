# torch.nn.Module

`nn.Module` 是 PyTorch 中所有神经网络模块的基类。构建自定义的神经网络模型时，必须继承该类。

### 核心概念

自定义模型需要实现以下两个主要部分：
1. `__init__(self)`: 构造函数，用于定义网络的各个层（如线性层、卷积层等）和其他可学习的参数。**必须在最开始调用 `super().__init__()`**。
2. `forward(self, x)`: 前向传播函数，定义数据 `x` 在网络中的流向。PyTorch 会根据 `forward` 方法自动构建计算图，并借助 Autograd 引擎实现反向传播（自动求导）。

**注意**: 模块中可以嵌套包含其他 `nn.Module`（如 `nn.Linear`, `nn.Conv2d` 等），PyTorch 会自动追踪这些子模块的参数（Parameters）。

### 📝 举例说明

下面构建一个简单的多层感知机（MLP）用于分类任务。

```python
import torch
from torch import nn

class SimpleMLP(nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        super().__init__() # 必须调用父类的初始化方法
        
        # 定义网络层
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_size, num_classes)
        
    def forward(self, x):
        # 定义前向传播过程
        out = self.fc1(x)
        out = self.relu(out)
        out = self.fc2(out)
        return out

# 实例化模型
model = SimpleMLP(input_size=784, hidden_size=128, num_classes=10)

# 查看模型结构
print(model)

# 模拟输入一次前向传播
dummy_input = torch.randn(32, 784) # batch_size=32
output = model(dummy_input)
print("输出形状:", output.shape) # 预期: (32, 10)
```