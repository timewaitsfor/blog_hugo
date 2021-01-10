---
title: "实体对齐比赛文档"
date: 2020-12-23T14:29:35+08:00
# draft: true
---

# 前置工作
```sh
# 解压压缩包：
$ tar -xvf ea-task-submit.tar.gz
# 进入文件夹：（此后所有操作均在此文件夹下）
$ cd ea-task-submit
```

# 测试cuda环境
```
nvcc -V
```
如果类似：
{% asset_img index/2020-12-23-14-33-06.png %}
则说明环境为cuda环境为：V10.1.168

# 更新C++的编译环境
```sh
#安装scl
$ sudo yum install centos-release-scl
#安装想要的gcc版本，如gcc7
$ sudo yum install devtoolset-7
# gcc 路径加入环境变量
$ scl enable devtoolset-7 bash
# 测试下gcc是否配置正确
$ gcc -v
```
如果正确，输出结果如下：
{% asset_img index/2020-12-24-11-10-18.png %}
（图中为7.3.1)
gcc版本>=7即可。

# 安装cmake
```sh
#安装cmake
$ sudo yum install cmake
#安装cmake3
$ sudo yum install cmake3
#配置cmake默认为cmake3 Part 1
$ sudo alternatives --install /usr/local/bin/cmake cmake /usr/bin/cmake 10 \
--slave /usr/local/bin/ctest ctest /usr/bin/ctest \
--slave /usr/local/bin/cpack cpack /usr/bin/cpack \
--slave /usr/local/bin/ccmake ccmake /usr/bin/ccmake \
--family cmake
# 配置cmake默认为cmake3 Part 2
$ sudo alternatives --install /usr/local/bin/cmake cmake /usr/bin/cmake3 20 \
--slave /usr/local/bin/ctest ctest /usr/bin/ctest3 \
--slave /usr/local/bin/cpack cpack /usr/bin/cpack3 \
--slave /usr/local/bin/ccmake ccmake /usr/bin/ccmake3 \
--family cmake
# 启动下环境
$ source ./bashrc
# 测试cmake环境
$ cmake -version
```
如果正确，输出结果如下：
{% asset_img index/2020-12-24-11-19-08.png %}
这一步务必保证cmake>= 3.10

# 安装anaconda
anaconda安装包已下载在mo目录下，执行以下命令进行安装：
```sh
$ bash Anaconda3-2020.11-Linux-x86_64.sh
```
一路[ENTER]回车向下，接下来会出现一堆的License许可声明，出现如下文字，输入yes：
{% asset_img index/2020-12-23-14-44-18.png %}
继续一路[ENTER]回车，需要初始化环境时，输入yes
{% asset_img index/2020-12-23-14-45-29.png %}
如果正确，输出结果如下：
{% asset_img index/2020-12-23-14-46-02.png %}

## 配置python环境
```sh
#启动下环境，执行之后命令行前面会多一个（base）的标记
$ source ~/.bashrc
#安装虚拟环境，期间需要配合输入y继续进行下去
$ conda create -n eatask python=3.6.9
#激活环境
$ conda activate eatask 
```
如果正确，输出结果如下：
{% asset_img index/2020-12-24-11-30-44.png %}
确保命令行出现出现（eatask）标记再继续下一步操作。

# 安装pytorch
pytorch安装包已下载在mo目录下，执行以下命令进行安装：
```sh
$ pip install torch-1.6.0+cu101-cp36-cp36m-linux_x86_64.whl
```

# 安装Keops
```sh
$ pip install pykeops
```
##测试keops包是否安装成功
运行在mo目录下的测试代码：keops_exp.py
执行指令：
```sh
$ python keops_exp.py
```
{% asset_img index/2020-12-25-13-58-02.png %}z

# 安装pytest
```sh
$ pip install pytest-runner
```

# 安装pytorch geometric
```sh
#查看pytorch版本，理论上为1.6.0
$ python -c "import torch; print(torch.__version__)"
#查看cuda版本，理论上为10.1
$ python -c "import torch; print(torch.version.cuda)"
```
如果确认pytorch=1.6.0 且 cuda=10.1则运行如下命令
```sh
$ pip install torch-scatter -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ pip install torch-sparse -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ pip install torch-cluster -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ pip install torch-spline-conv -f https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ pip install torch-geometric
```

# 安装transformers
```sh
$ pip install transformers
```

# 数据预处理
```sh
$ python preprocessing.py
```

# 训练和测试模型
```sh
$ nohup bash run_all.sh &
```
最终结果保存在result.txt中。



