---
title: "Bug解决档案"
date: 2020-09-30T11:48:12+08:00
draft: true
---

# Pycharm远程debug时出现：
> ImportError: dlopen: cannot load any more object with static TLS

**解决办法**：
找到远程目录的*pydevd.py*文件，在文件开头加上：
```python
import torch
import sklean
```
在我的服务器上目录为：`/root/.pycharm_helpers/pydev/pydevd.py`