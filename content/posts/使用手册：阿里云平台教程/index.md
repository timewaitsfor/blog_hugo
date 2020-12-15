---
title: "使用手册：阿里云平台教程"
date: 2020-12-15T09:23:15+08:00
draft: true
categories: [使用手册]
---

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [如何通过GPU管理器提交训练](#如何通过gpu管理器提交训练)
      * [0. 创建Docker镜像](#0-创建docker镜像)
      * [1. 两台服务器](#1-两台服务器)
      * [2. 编写模型代码](#2-编写模型代码)
      * [3. 上传代码与训练集](#3-上传代码与训练集)
      * [4. 编写作业描述文件(.yaml)](#4-编写作业描述文件yaml)
      * [5. 提交、删除作业](#5-提交删除作业)
      * [6. 查看作业执行状态](#6-查看作业执行状态)
      * [7. 下载日志及模型](#7-下载日志及模型)
      * [Appendix A. 错误排查](#appendix-a-错误排查)
         * [任务运行失败常见的原因](#任务运行失败常见的原因)
         * [挂载错误排查](#挂载错误排查)
      * [Appendix B. docker镜像相关](#appendix-b-docker镜像相关)
         * [安装docker](#安装docker)
         * [拉取并运行镜像](#拉取并运行镜像)
         * [测试环境](#测试环境)
         * [保存环境](#保存环境)
         * [推送镜像到仓库](#推送镜像到仓库)
         * [其他](#其他)





# 如何通过GPU管理器提交训练

​		GPU管理器的目的是为了对阿里云GPU资源进行统一管理，提高资金和资源利用效率，避免浪费。用户无需自己创建ECS节点并配置环境，只需通过命令行接口提交训练作业，待作业完成后，从存储中下载日志和训练完的模型。GPU管理器还提供了作业状态监控接口，可随时查看作业状态。

本文档以mnist为例，说明了如何使用GPU管理器。

## 0. 创建Docker镜像

例如安装完成时容器终端是 (base) root@d453d6161939:/workspace# ，那d453d6161939就是我们需要的id，此时退出容器，运⾏

```
docker commit d453d6161939 [⾃定义容器名称]:[版本号] 
# 例如 
docker commit d453d6161939 lzj-dev:1.0
```

本地运行docker镜像(docker其他基本操作网上很多)

如果你有已经创建好的镜像，能满足你的使用需求，可以直接使用已有的镜像，省去重新打包上传的麻烦。如果需要自己打包上传镜像，请继续往下看。

不同的用户有自己的训练环境。用户可以定制自己的Docker镜像，这样可以解决环境搭建问题。创建具体镜像的步骤省略，可参考网上教程。接下来详细介绍如何将自己的镜像上传到阿里云镜像仓库中。

（1）登录阿里云Docker Registry （在群里联系集群管理同学）

```root
sudo docker login --username=虎嵩林 registry.cn-zhangjiakou.aliyuncs.com
```

密码：***

(2)  上传自己的镜像到阿里云镜像仓库

```rust
// 将镜像tag成阿里云镜像的格式
docker tag [ImageId] registry.cn-zhangjiakou.aliyuncs.com/iie-lct/[姓名-镜像名-版本号] 
# 例如
docker tag sxh-dev:1.0 registry.cn-zhangjiakou.aliyuncs.com/iie-lct/lzj-dev:1.0

// 上传镜像
docker push registry.cn-zhangjiakou.aliyuncs.com/iie-lct/[姓名-镜像名-版本号]
#例如
docker push registry.cn-zhangjiakou.aliyuncs.com/iie-lct/lzj-dev:1.0
```

## 1. 两台服务器

我们已经提前配置好了两台服务器环境，一台用来打包 docker 镜像，一台用来提交作业。

(1) 打包镜像的服务器（这个ip会变，需要时请在群里联系集群管理同学）

(2) 提交作业的服务器

    IP: 39.101.128.205

登录的账户和密码，可以联系平台组老师同学获得，或者询问阿里云平台讨论群里的其他同学。

## 2. 编写模型代码

​	训练日志、模型的保存以及其它需要持久化存储的内容，都需要自己在代码中实现，可以参考pytorch.7z压缩包中的pytorch.py，或 pytorch 等框架的教程文件。务必要修改代码中读取和保存数据的语句所使用的路径为你的作业目录（/nfs/testdata/code/[姓名]/[训练作业名]）。

​    代码编写完成后，先自行测试通过，再进行后续步骤。

​      

## 3. 上传代码与训练集

​	将代码和数据拷贝到/nfs/testdata/code/[姓名]/[训练作业名]目录下。由于数据实际存储在远程的NAS中（实际保存在NAS的/code目录下），上传过程需要等待一定时间才能完成。

PS：挂载错误排查

 https://help.aliyun.com/document_detail/129698.html?spm=a2c4g.11186623.2.23.62ebaa3d5K90K2#concept-1614284 



## 4. 编写作业描述文件(.yaml)

​	提交作业使用下面的yaml模板，需根据自己的作业情况进行修改。**作业名请用姓名首字母开头，并加入简要的描述信息，方便区分。例如，sxh-mnist-1。作业名重复将导致提交失败。**

**复制下面的 yaml 模板进行修改时，请删除注释，否则可能提交时报错**

```yaml
jobname: xxxx          # 必须，修改成自己job的名字，作业名不能重复
tasks:                 # 必须
  - amount: 1          # 可选, 默认为 1
  	name:  task-0      # 可选，任务名称，如果是分布式训练可能会有多个任务，通常可以不动
    workingDir:        # 可选
    command: xxxx      # 可选
    args: xxxx         # 可选
    image:  xxxx       # 必须
    resources:         # 可选，cpu,gpu,memory等选项，若不需要，将对应字段整行删除即可
      cpu: "500m"	   # 字段值可修改，若不需要则将此行删除
      memory: "1024Mi" # 字段值可修改，若不需要则将此行删除
      gpu: "1"         # 字段值可修改，若不需要则将此行删除
    ports:             # 可选，name只能有数字和字母，且必须有一个字母
      number:
      name:
```

同学们可以使用的最简单的作业描述示例：

```yaml
jobname: test_job1      
tasks:                 
  - command: xxxx    
    image:  xxxx       
    resources:         
      gpu: "1"     
```

command是容器内执行的命令，有两种方式：

- 直接写运行命令，如果带着命令行参数，那参考下面的例子

    ```rust
    command: ["python", "pytorch.py"]
    args: ["-train", "-batch_size", "8"]
    ```

    这种方式有个问题，就是一次只能执行一个命令，不够灵活，如果依赖项在conda之类的虚拟环境中需要激活自己的conda环境，可以采用下面这种方式。

- 编写shell，例如(这里conda 虚拟环境的名字为work)：

    ```shell
    #!/usr/bin/env bash
    # >>> conda initialize >>>
    # !! Contents within this block are managed by 'conda init' !!
    __conda_setup="$('/home/user/miniconda/envs/py36/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__conda_setup"
    else
        if [ -f "/home/user/miniconda/envs/work/etc/profile.d/conda.sh" ]; then
            . "/home/user/miniconda/envs/work/etc/profile.d/conda.sh"
        else
            export PATH="/home/user/miniconda/envs/work/bin:$PATH"
        fi
    fi
    unset __conda_setup
    # <<< conda initialize <<<
    conda activate work
    
    # run command here
    python pytorch.py -train -batch_size 8
    ```

    在1-13行初始化conda环境，在16行激活自己的conda环境，19行开始可以连续写很多命令，会依次执行。


## 5. 提交、删除作业

（1）进入 作业的 yaml文件的同级目录，假设job的Yaml文件命名为： job.yaml

（2）使用下面的命令进行创建

```rust
vcctl job create -f job.yaml
```

出现下面的显示则创建成功

submit job test-job1 successfully

（3）使用下面的命令删除作业，在作业出错，或者大改的情况下删除作业再重新提交。

```
vcctl job delete -N test-job1
```

PS：如果创建失败，也会出现显示具体错误原因。一般原因是job命名冲突，或者yaml格式错误（有严格的缩进）


## 6. 查看作业执行状态

注：这一节的详细内容，请参考"提交训练作业时的常见问题及排查方法.md"。

使用

```shell
 vcctl job list
```

作业目前有三个正常的状态：**Pending（等待调度）, Running(作业运行中)，Completed（已完成）**。



## 7.查看、下载日志及模型

**查看日志方法1**：

使用vcctl job logs命令：

```shell
vcctl job logs -N jobname
```

若想查看限定行数的日志，即使用：

```shell
vcctl job logs -N jobname -L lines
```

lines指行数，如想查看最近20行的日志，把lines换成20即可，若想重定向到文件中可参照如下命令：

```shell
vcctl job logs -N jobname -L lines  >  filename
```

filename为指定的文件名称。

**查看日志方法2**：

进入/nfs/testdata/code/[姓名]/[作业名] (自己代码中指定的位置) 即可查看日志文件和模型。



## Appendix A. 错误排查

**注：这一节的详细内容，请结合"提交训练作业时的常见问题及排查方法.md"一起阅读。**

如果出现 job 处于pending状态超过10分钟。使用下面的命令查看具体原因：

```shell
vcctl job logs -N jobname   
```

 执行vcctl job  delete -N jobname 命令可删除错误作业。

### 任务运行失败常见的原因

（1) 文件或者数据找不到

这是由于NAS文件还在上传，等待一段时间即可

（2）文件权限错误

修改本地目录（本来例子中是/nfs/testdata）的权限

例如： chmod 777 /nfs/testdata -r 



### 挂载错误排查

 https://help.aliyun.com/document_detail/129698.html?spm=a2c4g.11186623.2.23.62ebaa3d5K90K2#concept-1614284 



## Appendix B. docker镜像相关

目前为大家建立了docker镜像，按照收集到的requirement文件，不过在建立的过程中由于阿里云的pip源不全，以及部分库可能包含其他依赖（C库）导致安装失败，所以大家还是需要拉取自己的镜像做微调，同时也方便在本地验证代码的正确性，实验室在阿里云上建立了镜像仓库，仓库地址

```
registry.cn-beijing.aliyuncs.com/iielct/
username: ***
password: ***
```
为大家建立的docker镜像列表（发送requirements的同学），名称为名字首字母小写-dev:版本号

```shell
registry.cn-beijing.aliyuncs.com/iielct/zy-dev     1.0                 9b361a82db03
registry.cn-beijing.aliyuncs.com/iielct/sp-dev     1.0                 b0c9611c3b63
registry.cn-beijing.aliyuncs.com/iielct/qwh-dev    1.0                 e50e80d383b3
registry.cn-beijing.aliyuncs.com/iielct/wlw-dev    1.0                 737a62dfa3b3
registry.cn-beijing.aliyuncs.com/iielct/sxh-dev    1.0                 a1c9f304359f
registry.cn-beijing.aliyuncs.com/iielct/wdj-dev    1.0                 3004884c6286
registry.cn-beijing.aliyuncs.com/iielct/ycy-dev    1.0                 f98022bd2f07
registry.cn-beijing.aliyuncs.com/iielct/zwb-dev    1.0                 8d2527648642
registry.cn-beijing.aliyuncs.com/iielct/zl-dev     1.0                 d53a58641632
registry.cn-beijing.aliyuncs.com/iielct/lyx-dev    1.0                 9479281f9b62
registry.cn-beijing.aliyuncs.com/iielct/mqw-dev    1.0                 d37f5a74d882
registry.cn-beijing.aliyuncs.com/iielct/lsw-dev    1.0.1               531ac9403099
registry.cn-beijing.aliyuncs.com/iielct/lmm-dev    1.0.1               3cad087707ee
registry.cn-beijing.aliyuncs.com/iielct/nlp-base   cuda-10.0           4d578c17a968
```

所有镜像都是基于ubuntu 16.06 64位系统，conda里面有两个虚拟环境，一个是$work$ ，一个是$env$，work是之前测试过程中建立的，基本包含了自然语言处理常用的类库，env是根据大家的requirements建立的，不过一部分包安装失败了。不想折腾的同学可以直接激活work环境试试是否可以满足要求。

### 安装docker

略

### 拉取并运行镜像

```shell
docker run -it [自己的镜像完整路径]:[版本号] /bin/bash
```

比如

```
docker run -it registry.cn-beijing.aliyuncs.com/iielct/sxh-dev:1.0 /bin/bash
```

命令会拉取镜像并运行，直接进入终端，默认激活了env环境。

### 测试环境

可以通过挂载目录的方式让docker共享本地代码文件夹，本地文件夹需要指定绝对路径，例如我希望本地的/home/user/code/test/指定到docker的/app目录下，可以执行

```
docker run -it -v /home/user/code/test:/app registry.cn-beijing.aliyuncs.com/iielct/sxh-dev:1.0 /bin/bash
```

之后就可以在docker里测试代码和补全开发环境了，修改完成记得保存（见下一节）。

### 保存环境

使用docker的commit命令保存容器为新的镜像，docker run的时候终端会有个容器的id，在终端可以看到，例如运行docker的终端名字

```shell
(env) user@ad4bce6db5fd:/app$ 
```

其中ad4bce6db5fd就是容器id，假设此时已经完成了对容器的修改，exit命令退出回到本地终端，运行

```shell
docker commit ad4bce6db5fd [新镜像名称]
```

我们为大家的镜像做了版本号，为了区分方便，每次修改建议给不同的版本号，初始默认是1.0，修改之后可以分配为1.0.1，例如

```shell
docker commit ad4bce6db5fd registry.cn-beijing.aliyuncs.com/iielct/sxh-dev:1.0.1
```

### 推送镜像到仓库

```shell
docker push [镜像名]
```

例如

```shell
docker push registry.cn-beijing.aliyuncs.com/iielct/sxh-dev:1.0.1
```

这样就完成了自己的镜像的更新。后续任务可以指定更新后的镜像了。

### 其他

未发送依赖库的同学可以以仓库任意镜像为基类（建议用nlp-base）安装前述步骤创建自己的镜像并推送。有问题可以群里问。