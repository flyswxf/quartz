# torch.utils.data.Dataset

`Dataset` 是 PyTorch 中表示数据集的抽象基类。任何自定义的数据集都需要继承这个类，并重写其中的两个核心方法。

### 核心方法

要自定义数据集，必须实现以下两个方法：
1. `__len__(self)`: 返回数据集的大小（样本总数）。
2. `__getitem__(self, idx)`: 支持索引访问，通过索引 `idx` 返回数据集中的第 `idx` 个样本（通常是一个包含数据和标签的元组）。

### 📝 举例说明

假设有一个包含图像文件路径和对应标签的列表，需要将其封装为 PyTorch 的 `Dataset`。

```python
import torch
from torch.utils.data import Dataset

class CustomImageDataset(Dataset):
    def __init__(self, img_paths, labels):
        """
        初始化数据集，可以在这里进行数据的加载或预处理参数的设置。
        """
        self.img_paths = img_paths
        self.labels = labels

    def __len__(self):
        """
        返回数据集中样本的总数。
        """
        return len(self.img_paths)

    def __getitem__(self, idx):
        """
        根据索引 idx 获取一个样本。
        """
        # 1. 根据索引获取对应的数据
        img_path = self.img_paths[idx]
        label = self.labels[idx]
        
        # 2. 模拟读取图像和预处理 (这里用随机张量代替)
        # image = read_image(img_path)
        image = torch.randn(3, 224, 224) 
        
        # 3. 返回数据和标签
        return image, label

# 测试自定义数据集
paths = ["img1.jpg", "img2.jpg", "img3.jpg"]
targets = [0, 1, 0]

dataset = CustomImageDataset(paths, targets)
print("数据集大小:", len(dataset))

# 获取第一个样本
img, label = dataset[0]
print("第一个样本的图像形状:", img.shape, "标签:", label)
```