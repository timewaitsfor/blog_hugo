---
title: "Code Analysis: deep Graph Matching Consensus"
date: 2020-10-22T17:04:40+08:00
katex: true
---

# Abstract
这篇博客是论文《Deep Graph Matching Consensus》 这篇论文的代码分析。
这篇论文来自，ICML2020，作者是 *Matthias Fey* ，这是写了 Pytorch Geomatric的大神，因此拜读下他的代码。
之前也研究过这个代码，但是没能看懂源码，这次是mou足了劲一定要看懂。
从2020/10/22上午开始一直到刚刚（2020-10-22 17:09:45）才算是完整的理解了这篇论文的代码，总共花费了18P。

# 思考和总结
1. 这个代码运行的超级快，基本上5分钟左右就可以迭代到非常好的结果
2. 代码写的很散，很随意，我自己迭代模型的时候也要注意，不需要在“辞藻”“格式”上太下功夫
3. 代码里有很多和论文中介绍不太一致的部分，需要仔细推敲
4. 代码里可以学到很多技巧关于pytorch，里面很多可以打包到我自己的工具包里的，注意采用
5. 我试了用手抄代码和写一个玩具模型的方式来尝试理解代码，后来总结感觉，直接创建一个玩具模型是最高效的，同时要做记录，用笔和纸记下所思所想，画下流程等等。

# 数据集
首先介绍下，我比较关注的实体对齐任务，数据集采用DBP15K。在我之前REGCN的实验中该数据集（来源于RDGCN）在zh-en分类下，source KG和target KG是合在一起的，总共有38960个entity. 而DGMC分开的，source KG有19388个entity，target KG有19572个entity. （合在一起确实是38960）。 <font color="red">这两个数据集的实体特征不知道用的是不是一样的</font>。但是有监督数据，RDGCN有15000个，而DGMC的训练集和测试集是分开的，train_y 有3205个，test_y有8788个，如果把RDGCN的训练集和测试集分开，则，训练集4500个，测试集10500个，DGMC的训练: 测试 = 0.36: 1，RDGCN的训练: 测试 = 0.42: 1。 <font color="red">训练集和测试集的比例不同</font>，不知道DGMC是怎么清洗的，这个划分比例应该效果更差点才对。

# Methodology

首先有2个GNN的模块：
```python
psi_1 = RelGNN(input: 300 -->  output: 256)
psi_2 = RelGNN(input: 32 -->  output: 32)
```
主模块为 DGMC()  
 stepin DGMC.forward()的代码

```python
h_s = self.psi_1(x_s, edge_index_s, edge_attr_s)
h_t = self.psi_1(x_t, edge_index_t, edge_attr_t)
```
先通过psi_1分别学习source KG 和 target KG中entity的embedding.
得到h_s和h_t，维度为 19388 x 256 和 19572 x 256.

然后获取h_s和h_t的**soft correspondences**, 在论文中表现为
$$
S^{(0)} = sinkhorn(\hat{S}^{(0)}) \in [0, 1]^{|v_s|\times |v_t|}\ with\ \hat{S}^{(0)} = H_s H_t^\intercal \in \mathbb{R}^{|v_s|\times |v_t|}
$$

```python
# ------ Sparse variant ------ #
S_idx = self.__top_k__(h_s, h_t)  # [B, N_s, k]
```
这里的`__top_k__`如下所示：
```python
def __top_k__(self, x_s, x_t):  # pragma: no cover
    r"""Memory-efficient top-k correspondence computation."""
    if LazyTensor is not None:
        x_s = x_s.unsqueeze(-2)  # [..., n_s, 1, d]
        x_t = x_t.unsqueeze(-3)  # [..., 1, n_t, d]
        x_s, x_t = LazyTensor(x_s), LazyTensor(x_t)
        S_ij = (-x_s * x_t).sum(dim=-1)
        return S_ij.argKmin(self.k, dim=2, backend=self.backend)
    else:
        x_s = x_s  # [..., n_s, d]
        x_t = x_t.transpose(-1, -2)  # [..., d, n_t]
        S_ij = x_s @ x_t
        return S_ij.topk(self.k, dim=2)[1]
```
即可以用KeOps包来快速求矩阵乘法，如果不用KeOps则可以用朴素的`else`中的代码。
这里有个有趣的东西，第一次知道Pytorch里用`@`可以直接表示矩阵乘法。  
而`torch.topk`表示取指定维度中最大的k个元素，返回两个值，第一个值为最大的k个元素的values，而第二个值为最大的k个元素的索引。

下面这个模块负责一个任务，注释里也写了，即除了“收集”topK个元素之外，还要随机采样k个元素，并且还要进行检查，如果这2k个元素里没有ground_truth，则在最后一维加上ground_truth.
```python
# In addition to the top-k, randomly sample negative examples and
# ensure that the ground-truth is included as a sparse entry.
if self.training and y is not None:
    rnd_size = (B, N_s, min(self.k, N_t - self.k))
    S_rnd_idx = torch.randint(N_t, rnd_size, dtype=torch.long,
                                device=S_idx.device)
    S_idx = torch.cat([S_idx, S_rnd_idx], dim=-1)
    S_idx = self.__include_gt__(S_idx, s_mask, y)
```
这样的话会得到一个新的`S_idx` 维度为：(1 x 19388 x 20), 这里的20就是新的k，既包含topK和random samples又包含ground_truth.  
继续往下看：
```python
k = S_idx.size(-1)
tmp_s = h_s.view(B, N_s, 1, C_out)
idx = S_idx.view(B, N_s * k, 1).expand(-1, -1, C_out)
tmp_t = torch.gather(h_t.view(B, N_t, C_out), -2, idx)
```
`idx`就是将`S_idx`展开，维度为（1 x 387760 x 256）
可以看一眼，`idx` 如下：（387760 = 19388 x 20）
```python
tensor([[[16492, 16492, 16492,  ..., 16492, 16492, 16492],
         [12231, 12231, 12231,  ..., 12231, 12231, 12231],
         [ 2364,  2364,  2364,  ...,  2364,  2364,  2364],
         ...,
         [13908, 13908, 13908,  ..., 13908, 13908, 13908],
         [16987, 16987, 16987,  ..., 16987, 16987, 16987],
         [12498, 12498, 12498,  ..., 12498, 12498, 12498]]], device='cuda:0')
```
而`S_idx`如下：
```python
tensor([[[16492, 12231,  2364,  ...,  8276, 12392,     0],
         [17302,  1189, 14384,  ..., 10488,  5975,     1],
         [14384,  7479, 12856,  ...,  2019, 11954,     2],
         ...,
         [14384,  2581, 12856,  ...,  9599, 15871,  3933],
         [14384, 11952, 12856,  ..., 18672,  3663, 12765],
         [16527,  9158, 14656,  ..., 13908, 16987, 12498]]], device='cuda:0')
```
将`S_idx`展开，同时每个元素重复256遍。  
`torch.gather` （关于这个函数的使用参考：传送门）会返回一个和`idx` 的size一模一样的tensor，可以将387760个entity在h_t中的embedding取出来。这里模块的设计和我平常用的很不一样，我比较习惯于用torch.index_select，不知道这两者效率差异有多少。

然后就可以计算新的**correspondences matrix**：
```python
S_hat = (tmp_s * tmp_t.view(B, N_s, k, C_out)).sum(dim=-1)
S_0 = S_hat.softmax(dim=-1)[s_mask]
```
`tmp_s`的维度是 (1, 19388, 1, 256)  
`tmp_t.view(B, N_s, k, C_out)` 为 (1, 19388, 20,  256)  
其实得到的是所有的source KG中的实体和`S_idx`中sample出来的examples之间的相似度（类比于$\sqrt{x \cdot y}$ 欧氏距离）  

后面的部分就相当于将这种相似度转化为稀疏矩阵的形式
```python
 S_L = S_hat.softmax(dim=-1)[s_mask]
S_idx = S_idx[s_mask]

# Convert sparse layout to `torch.sparse_coo_tensor`.
row = torch.arange(x_s.size(0), device=S_idx.device)
row = row.view(-1, 1).repeat(1, k)
idx = torch.stack([row.view(-1), S_idx.view(-1)], dim=0)
size = torch.Size([x_s.size(0), N_t])

S_sparse_0 = torch.sparse_coo_tensor(
    idx, S_0.view(-1), size, requires_grad=S_0.requires_grad)
S_sparse_0.__idx__ = S_idx
S_sparse_0.__val__ = S_0

S_sparse_L = torch.sparse_coo_tensor(
    idx, S_L.view(-1), size, requires_grad=S_L.requires_grad)
S_sparse_L.__idx__ = S_idx
S_sparse_L.__val__ = S_L
```
以上属于**local feature matching**的部分，可以得到一个还不错的**correspondences matrix**来确定哪些元素之间是对齐的。实验结果为0.68左右。

这篇论文的第二个部分在于，**Refine correspondence matrix**. 由于第一部分是对邻域特征进行aggregate，而这种聚合主要是对邻居特征进行聚合，没有考虑纯粹的网络结构特征。用作者的话说就是
