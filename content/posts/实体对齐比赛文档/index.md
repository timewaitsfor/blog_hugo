---
title: "实体对齐比赛文档"
date: 2020-12-23T14:29:35+08:00
draft: true
---

# 测试cuda环境
```
nvcc -V
```
如果类似：
{% asset_img index/2020-12-23-14-33-06.png %}
则说明环境为cuda环境为：V10.1.168

# 安装anaconda

## 下载anaconda
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2020.02-Linux-x86_64.sh

## 安装anaconda
`bash Anaconda3-2020.02-Linux-x86_64.sh`
{% asset_img index/2020-12-23-14-42-50.png %}
输入[ENTER]回车
接下来会出现一堆的License许可声明，一路[ENTER]回车向下，出现如下文字，输入yes：
{% asset_img index/2020-12-23-14-44-18.png %}
输入`yes`
继续一路[ENTER]回车
{% asset_img index/2020-12-23-14-45-29.png %}
输入yes

出现如下
{% asset_img index/2020-12-23-14-46-02.png %}
代表安装成功

## anaconda启动环境
```
source ~/.bashrc
```
这时候输入一次`python`
{% asset_img index/2020-12-23-14-47-25.png %}
表示安装正确。

# 安装pytorch

输入`pip install torch torchvision torchaudio`


# 安装pytorch geometric
{% asset_img index/2020-12-23-14-51-51.png %}
输入`python -c "import torch; print(torch.__version__)"`
查看pytorch版本，如图为1.7
输入`python -c "import torch; print(torch.version.cuda)"`
查看cuda版本，如图为10.2

pip install torch-scatter -f https://pytorch-geometric.com/whl/torch-1.7.0+cu101.html
pip install torch-sparse -f https://pytorch-geometric.com/whl/torch-1.7.0+cu101.html
pip install torch-cluster -f https://pytorch-geometric.com/whl/torch-1.7.0+cu101.html
pip install torch-spline-conv -f https://pytorch-geometric.com/whl/torch-1.7.0+cu101.html
pip install torch-geometric
