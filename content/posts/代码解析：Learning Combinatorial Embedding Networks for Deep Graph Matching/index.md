---
title: "代码解析：Learning Combinatorial Embedding Networks for Deep Graph Matching"
date: 2020-11-12T14:35:08+08:00
---

# 背景知识理解

## Dataset类
torch.utils.data.Dataset 是一个表示数据集的抽象类。任何自定义的数据集都需要继承这个类并覆写相关方法。  
所谓数据集，其实就是一个负责处理索引(index)到样本(sample)映射的一个类(class)。
自定义类大致是这样的：

```python
class CustomDataset(data.Dataset):#需要继承data.Dataset
    def __init__(self):
        # TODO
        # 1. Initialize file path or list of file names.
        pass
    def __getitem__(self, index):
        # TODO
        # 1. Read one data from file (e.g. using numpy.fromfile, PIL.Image.open).
        # 2. Preprocess the data (e.g. torchvision.Transform).
        # 3. Return a data pair (e.g. image and label).
        #这里需要注意的是，第一步：read one data，是一个data
        pass
    def __len__(self):
        # You should change 0 to the total size of your dataset.
        return 0
```

data pair 是指数据和标注？  

## eval 方法
