---
title: 使用手册：Pytorch
date: 2020-10-23T08:46:27+08:00
categories: [使用手册]
toc: true
---

## torch_scatter.scatter
这是pytorch geometric 中经常使用的一个函数，比如如下用例：
```python
tmp_t = r_s.view(B, N_s, 1, R_in) * S.view(B, N_s, k, 1)
tmp_t = tmp_t.view(B, N_s * k, R_in)
idx = S_idx.view(B, N_s * k, 1)
r_t = scatter_add(tmp_t, idx, dim=1, dim_size=N_t)
```
看几个例子就懂了，第一个例子：
```python
from torch_scatter import scatter

src = torch.randn(10, 6, 64)
index = torch.tensor([0, 1, 0, 1, 2, 1])

# Broadcasting in the first and last dim.
out = scatter(src, index, dim=1, reduce="sum")

print(out.size())
```
```sh
torch.Size([10, 3, 64])
```
也就是说，输出的out结果其它维度会和src一样，比如这里src的第0维是10，第2维是64，而由于`scatter`中指定了按第1维进行运算，其它维度都会自动Broadcasting. 于是，第一维按照`index` 里有 `0, 1, 2`三个维度，所以最终输出`out`的`size=(10, 3, 64)`.

再看一个例子：
```python
import torch
from torch_scatter import scatter_max

src = torch.tensor([[2, 0, 1, 4, 3], [0, 2, 1, 3, 4]])
index = torch.tensor([[4, 5, 4, 2, 3], [0, 0, 2, 2, 1]])

out, argmax = scatter_max(src, index, dim=-1)
```
这里的`argmax`先不管他，看一下按照之前的思路，src的维度是`(2, 5)`，而`scatter_max`指定了按第`-1`维进行运算也就是第`1`维，所以，其它维度自动Broadcasting，`out` 里最大维度为`5`，最小为`0`, 所以其维度理应为`x(2, 6)`. 输出一下看看：
```python
print(out)
tensor([[0, 0, 4, 3, 2, 0],
        [2, 4, 3, 0, 0, 0]])
```
每一个第`0`维度按照`index` 寻找最大值，以第`0`维的第一行为例，在`index` 中的第`0`维的第一行第`0`维的第一行为`[4, 5, 4, 2, 3]`，只含`2, 3, 4, 5` 四个元素，所以，`0, 1` 赋值为`0`，而其它`index` 元素找最大的，`2`出现1次，值为`4`，因此第2个index元素处赋值为4，`3`出现1次，值为`3`，因此第3个index元素处赋值为3，`4`出现2次，值分别为`2, 1`，因此第4个index元素处赋值为其中较大的那个，即`2`，`5`出现1次，值为`0`，因此第5个index元素处赋值为0.  
Over!

## pytorch.gather/scatter_
[pytorch.gather/scatter_的用法](https://zhuanlan.zhihu.com/p/101896024)

## 输出整个tensor的方法
```python
torch.set_printoptions(profile="full")
print(x) # prints the whole tensor
torch.set_printoptions(profile="default") # reset
print(x) # prints the truncated tensor
```

## torch.cumsum()

```python
a = torch.arange(0, 6).view(2, 3)
print(a)
b = a.cumsum(dim=0)
c = a.cumsum(dim=1)
print(b)
print(c)
```
对于二维输入a，dim=0(第1行不动，将第1行累加到其他行)；dim=1(进入最内层，转化成列处理。第1列不动，将第1列累加到其他列；
```sh
tensor([[0, 1, 2],
        [3, 4, 5]])
tensor([[0, 1, 2],
        [3, 5, 7]])
tensor([[ 0,  1,  3],
        [ 3,  7, 12]])
```
[torch.cumsum() 和 torch.cumprod()](https://blog.csdn.net/qq_30122359/article/details/102955570)
