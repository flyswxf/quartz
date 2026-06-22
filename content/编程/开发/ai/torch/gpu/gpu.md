# PyTorch中的GPU与数据操作

在PyTorch中，管理数据（张量）和模型在CPU与GPU之间的移动是深度学习编程的基础。以下是常见的设备与数据操作。

## 1. 检查 GPU 状态

在进行任何GPU操作前，通常需要检查当前环境是否支持CUDA，并获取GPU信息：

```python
import torch

# 检查 CUDA 是否可用
is_gpu_available = torch.cuda.is_available()
print(f"GPU 可用: {is_gpu_available}")

if is_gpu_available:
    # 获取 GPU 数量
    print(f"GPU 数量: {torch.cuda.device_count()}")
    
    # 获取当前 GPU 索引 (默认是 0)
    print(f"当前 GPU 索引: {torch.cuda.current_device()}")
    
    # 获取 GPU 名称
    print(f"GPU 名称: {torch.cuda.get_device_name(0)}")
```

## 2. 定义设备 (Device)

最佳实践是动态定义设备对象，这样代码既可以在有GPU的机器上运行，也能在只有CPU的机器上运行：

```python
# 动态分配设备
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 如果有多块GPU，也可以指定具体的GPU编号
device_1 = torch.device("cuda:1") # 第二块GPU
```

## 3. 张量 (Tensor) 的 GPU 操作

### 3.1 将张量移动到 GPU/CPU
使用 `.to(device)` 方法是推荐的做法：

```python
x = torch.tensor([1.0, 2.0]) # 默认在 CPU 上

# 移动到指定设备 (推荐)
x = x.to(device)

# 显式移动 (不推荐用于通用代码，因为如果没GPU会报错)
x_gpu = x.cuda()
x_cpu = x_gpu.cpu()
```

### 3.2 直接在 GPU 上创建张量
为了节省从CPU拷贝到GPU的时间，可以直接在目标设备上初始化张量：

```python
# 推荐：直接在指定设备上创建
y = torch.ones(2, 3, device=device)
z = torch.randn((2, 2), device="cuda:0")
```

## 4. 模型 (Module) 的 GPU 操作

模型本质上是由参数（也是张量）组成的，需要将模型的参数移动到GPU上才能进行GPU加速训练。

```python
import torch.nn as nn

class MyModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(10, 2)
        
model = MyModel()

# 将模型移动到设备上 (就地修改，但通常还是写成 model = model.to(device))
model = model.to(device)
```

**⚠️ 重要注意：** 
如果你要定义优化器 (Optimizer)，**必须在将模型移动到GPU之后**再定义优化器，否则优化器内部会追踪CPU上的参数，导致报错。

```python
# 正确顺序：
model = model.to(device)
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
```

## 5. 多设备下的数据对齐问题

在PyTorch中，**进行运算的两个张量（或模型与数据）必须在同一个设备上**。否则会触发经典错误：
`RuntimeError: Expected all tensors to be on the same device, but found at least two devices, cuda:0 and cpu!`

```python
# 错误示范：
a = torch.tensor([1, 2]).to("cuda")
b = torch.tensor([3, 4]).to("cpu")
# c = a + b  # 会报错！

# 正确处理：
c = a + b.to("cuda")
```
在训练循环中，通常需要将输入数据和标签都移动到相同的设备：
```python
for inputs, labels in dataloader:
    inputs = inputs.to(device)
    labels = labels.to(device)
    
    outputs = model(inputs)
    loss = criterion(outputs, labels)
```

## 6. 显存管理与清理

在训练过程中如果出现显存不足（OOM），可以手动清理不再使用的变量，并清空缓存：

```python
# 1. 删除不再需要的张量
del tensor_a
del tensor_b

# 2. 清空 PyTorch 的显存缓存
torch.cuda.empty_cache()
```
*注：`empty_cache()` 不会释放正在被占用的显存，只会释放PyTorch缓存的显存分配器中的空闲内存（供其他GPU程序使用）。要真正释放显存，必须确保没有变量引用那些张量（比如使用 `del`）。*

## 7. NumPy 转换注意
只有在 CPU 上的张量才能转换回 NumPy 数组。如果张量在 GPU 上，或者它带有计算图梯度（requires_grad=True），需要先进行处理：

```python
# x 此时在 GPU 上，且可能带有梯度
# 必须先 .detach() 剥离计算图，再 .cpu() 移动到CPU，最后 .numpy()
x_numpy = x.detach().cpu().numpy()
```
