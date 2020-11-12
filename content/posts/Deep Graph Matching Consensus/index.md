---
title: "Deep Graph Matching Consensus"
date: 2020-09-14T11:11:41+08:00
tags : ["doubt"]
draft: true
---

# abstract
Deep Graph Matching Consensus 的代码

# 模型初始化

```python
psi_1 = RelCNN(data.x1.size(-1), args.dim, args.num_layers, batch_norm=False,
               cat=True, lin=True, dropout=0.5)
psi_2 = RelCNN(args.rnd_dim, args.rnd_dim, args.num_layers, batch_norm=False,
               cat=True, lin=True, dropout=0.0)
```
$ \psi_1$ 参数`data.x1.shape =  19388 x 300`,  `args.dim = 256`. 

这里的 `args.rnd_dim`默认设置为32，按我的理解，应该这两个$ \psi$ 表示两个相同的GNN model 以进行source network和target network分别学习entity的表示。 

$ \psi_1$ 是 300 --> 256 

$ \psi_1$ 是 32 --> 32

## stepin RelCNN(.)
```python
...
self.convs = torch.nn.ModuleList()
self.batch_norms = torch.nn.ModuleList()
for _ in range(num_layers):
    self.convs.append(RelConv(in_channels, out_channels))
    self.batch_norms.append(BN(out_channels))
    in_channels = out_channels
...
```
### stepinin RelConv(MessagePassing)

```python
self.lin1 = Lin(in_channels, out_channels, bias=False)
self.lin2 = Lin(in_channels, out_channels, bias=False)
self.root = Lin(in_channels, out_channels)
```
这个GNN主要是由三个liner层组成的。  
由于这个函数继承自`MessagePassing(.)`

### stepinin MessagePassing(.)

<div>
$$
\mathbf{x}_i^{\prime} = \gamma_{\mathbf{\Theta}} \left( \mathbf{x}_i,
        \square_{j \in \mathcal{N}(i)} \, \phi_{\mathbf{\Theta}}
        \left(\mathbf{x}_i, \mathbf{x}_j,\mathbf{e}_{j,i}\right) \right)
$$
<div>

> where :math:$\square$ denotes a differentiable, permutation invariant
    function, *e.g.*, sum, mean or max, and :math:$\gamma_{\mathbf{\Theta}}$
    and :math:$\phi_{\mathbf{\Theta}}$ denote differentiable functions such as
    MLPs.

    Args:
        aggr (string, optional): 
            The aggregation scheme to use
            (:obj:`"add"`, :obj:`"mean"`, :obj:`"max"` or :obj:`None`).
            (default: :obj:`"add"`)
        flow (string, optional): 
            The flow direction of message passing
            (:obj:`"source_to_target"` or :obj:`"target_to_source"`).
            (default: :obj:`"source_to_target"`)
        node_dim (int, optional): 
            The axis along which to propagate.
            (default: :obj:`-2`)

在**MessagePassing** 类的主函数名和`__init__`方法之间有一块奇怪的代码段：
```python
special_args: Set[str] = set([
    'edge_index', 'adj_t', 'edge_index_i', 'edge_index_j', 'size_i',
    'size_j', 'ptr', 'index', 'dim_size'
])
```

在`__init__`中有一段看不懂的代码：
```python
self.inspector = Inspector(self)
self.inspector.inspect(self.message)
self.inspector.inspect(self.aggregate, pop_first=True)
self.inspector.inspect(self.message_and_aggregate, pop_first=True)
self.inspector.inspect(self.update, pop_first=True)
```
切入看到`Inspector.inspect`代码如下：
```python
def inspect(self, func: Callable,
            pop_first: bool = False) -> Dict[str, Any]:
    params = inspect.signature(func).parameters
    params = OrderedDict(params)
    if pop_first:
        params.popitem(last=False)
    self.params[func.__name__] = params
```


Constructs messages from node :math: $j$ to node :math: $i$
in analogy to :math: $\phi_{\mathbf{\Theta}}$ for each edge in
:obj: $edge\_index$.
This function can take any argument as input which was initially
passed to :meth: $propagate$.
Furthermore, tensors passed to :meth: $propagate$ can be mapped to the
respective nodes :math: $i$ and :math: $j$ by appending :obj: $_i$ or
:obj: $_j$ to the variable name, *.e.g.* :obj: $x_i$ and :obj: $x_j$.


看一下MessagePassing.aggregate
```
def aggregate(self, inputs: Tensor, index: Tensor,
                  ptr: Optional[Tensor] = None,
                  dim_size: Optional[int] = None) -> Tensor:
```

MessagePassing.message_and_aggregate
```
def message_and_aggregate(self, adj_t: SparseTensor) -> Tensor:
```
MessagePassing.update
```
def update(self, inputs: Tensor) -> Tensor:
```

总结下MessagePassing(.) 函数，没太看懂，其中inspector的调用比较神奇，可能是在保存类中所有方法的参数，并且做子类（RelConv）和父类（MessagePassing）的参数区分。  

**存疑**
1. 为什么要把第一个参数都摘出来，以使得很多参数都为空？
2. `self.__user_args__`  和  `self.__fused_user_args__`这两个参数去哪了？
3. `self.fuse = self.inspector.implements('message_and_aggregate')` 这行代码又是做什么的？

## stepin DGMC(.)
DGMC --> RelCNN(.) --> RelConv(MessagePassing)  
先调用了第一层`h_s = self.psi_1(x_s, edge_index_s, edge_attr_s)`

还没有完全看懂，先这样把！以后再看，而且，这里作者并没有使用sinkhorn normalization.