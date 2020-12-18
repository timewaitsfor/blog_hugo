---
title: "使用手册：阿里云平台使用指南"
date: 2020-12-15T09:23:15+08:00
draft: true
categories: [使用手册]
---

# 阿里云平台的使用

    IP: 39.101.128.205

ssh root@39.101.128.205

`vcctl job list`
查看当任务

`vcctl job delete -N ea-exp`
删除当前任务

`vcctl job create -f yjz-ea-1.yaml`
新建任务

`kubectl get pods -o wide | grep ea`
查看进程详情

`kubectl logs ea-exp-task-0-0`
查看log

`kubectl exec ea-exp-task-0-0 -it -- bash`
可以查看显存使用情况，直接进到docker里

`kubectl describe pod ea-exp-task-0-0`
查看pod的信息

## TODO: 编写yaml文件

# docker使用

1. ` docker pull docker pull pytorch/pytorch:1.6.0-cuda10.1-cudnn7-devel`  
首先拉取在 https://hub.docker.com/r/pytorch/pytorch 上的镜像
2. `docker run -it pytorch/pytorch:1.6.0-cuda10.1-cudnn7-devel`  
运行该镜像
3. 配置环境：  
```sh
apt-get update

# 装Keops
apt-get install cmake
pip install pykeops 

# 装pytorch-geometric
pip install torch-scatter -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
pip install torch-sparse -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
pip install torch-cluster -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
pip install torch-spline-conv -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
pip install torch-geometric
```
4. `docker ps -a`  
查看有哪些可用的镜像
{% asset_img index/2020-12-15-16-28-27.png %}
比如第一个`028c43b647d0`就是我的镜像，
5. `docker commit 028c43b647d0 yjz-dev`  
起个名字，commit一下
6. `docker tag yjz-dev registry.cn-zhangjiakou.aliyuncs.com/iie-lct/yjz-dev:4.0`  
将镜像tag成阿里云镜像的格式
7. `docker push registry.cn-zhangjiakou.aliyuncs.com/iie-lct/yjz-dev:4.0`
上传镜像

这时候就完成docker镜像的制作了

# 比赛记录

1. 按照属性进行对齐，比如`placeOfBirth`, 比如金正恩的家乡是`平壤`，把家乡都是平壤的进行对齐

2. 加了异常检测

3. 融合了2种类型的特征，现要加入第三种

4. 主动学习策略

5. 小数据集上跑的，大的数据集很慢


# 服务器信息
141.164.45.210
root
F?4yjP?UPRzVH8Ns



# 其他信息
验证了一下，发现用那个跨网络的embedding，相似度很低