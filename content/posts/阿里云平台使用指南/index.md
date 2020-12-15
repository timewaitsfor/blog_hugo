---
title: "阿里云平台使用指南"
date: 2020-12-15T09:23:15+08:00
draft: true
---

# 代码执行

`vcctl job list`
查看当任务

`vcctl job delete -N ea-exp`
删除当前任务

`vcctl job create -f yjz-ea-1.yaml`
新建任务

`kubectl logs ea-exp-task-0-0`
查看log

`kubectl get pods -o wide | grep ea`
查看进程详情


# exp

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