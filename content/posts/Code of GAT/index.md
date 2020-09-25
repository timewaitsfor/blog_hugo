---
title: "Code of GAT"
date: 2020-09-25T10:05:31+08:00
---

# class GATConv(MessagePassing):
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

```python
out = self.propagate(edge_index, x=(x_l, x_r), 
alpha=(alpha_l, alpha_r), size=size)
```

