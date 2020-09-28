---
title: "Code of GAT"
date: 2020-09-25T10:05:31+08:00
katex: true
---

# class GATConv(MessagePassing):

<div>
$$
\mathbf{x}^{\prime}_i = \alpha_{i,i}\mathbf{\Theta}\mathbf{x}_{i} +
        \sum_{j \in \mathcal{N}(i)} \alpha_{i,j}\mathbf{\Theta}\mathbf{x}_{j}
$$
</div>
where the attention coefficients :math: $\alpha_{i,j}$ are computed as

$$
\alpha_{i,j} =
        \frac{
        \exp\left(\mathrm{LeakyReLU}\left(\mathbf{a}^{\top}
        [\mathbf{\Theta}\mathbf{x}_i \, \Vert \, \mathbf{\Theta}\mathbf{x}_j]
        \right)\right)}
        {\sum_{k \in \mathcal{N}(i) \cup \{ i \}}
        \exp\left(\mathrm{LeakyReLU}\left(\mathbf{a}^{\top}
        [\mathbf{\Theta}\mathbf{x}_i \, \Vert \, \mathbf{\Theta}\mathbf{x}_k]
        \right)\right)}.
$$


## Initial Parameters

 negative_slope (float, optional): LeakyReLU angle of the negative slope. (default: :obj:`0.2`)
<font color="red">看不懂这个是干什么的</font>

有几个参数值得注意：
`self.lin_l`， `self.lin_r`， `self.att_l` ，`self.att_r`  
以及，`self._alpha`。

```python
self.lin_l = Linear(in_channels[0], heads * out_channels, False)
self.lin_r = Linear(in_channels[1], heads * out_channels, False)
```

在这个例子里，  
in_channels = 5; out_channels = 4, heads = 8.  
H = heads = 8
C = out_channels = 4

x.size = [3, 5]

lin_l(x) : 3x5 => 3x(4x8)

```python
x_l = x_r = self.lin_l(x).view(-1, H, C)
alpha_l = alpha_r = (x_l * self.att_l).sum(dim=-1)
```

对于$x: 3\times 8 \times 4$ 和 `nn.Parameter(1, 8, 4)`之积居然为 $out: 3\times 8 \times 4$   
所以，代码块中 `(x_l * self.att_l).sum(dim=-1) => (3, 8)`. 

## propagate

```python
out = self.propagate(
            edge_index, 
            x=(x_l, x_r), 
            alpha=(alpha_l, alpha_r), 
            size=size)
```
 x、alpha 在`**kwargs`参数中.

### coll_dict

```python
 self.__user_args__ = self.inspector.keys(
            ['message', 'aggregate', 'update']).difference(self.special_args)
```
在这里，`message`, `aggregate`,`update`的参数为：
```python
message( 
    x_j, 
    alpha_j, 
    alpha_i,
    index, ptr,
    size_i)
```
```python
aggregate(
    inputs, 
    index,
    ptr,
    dim_size)
```
```python
update(inputs)
```

```python
self.special_args : Set[str] = set([
        'edge_index', 'adj_t', 
        'edge_index_i', 'edge_index_j', 
        'size_i', 'size_j', 
        'ptr', 'index', 'dim_size'
    ])
```

于是，` self.__user_args__ `应该为：
- x_j
- alpha_j
- alpha_i
- inputs (aggregate & update)

所以对于如下...
```python
coll_dict = self.__collect__(
    self.__user_args__, 
    edge_index, 
    size,
    kwargs)

...

def __collect__(self, args, edge_index, size, kwargs):

```

`args = self.__user_args__ ` ==> `{'alpha_i', 'alpha_j', 'x_j'}`


<font color="#3300CC">后面就不解读了，主要是觉得这个代码跟论文里的公式出入很大，疑惑</font>