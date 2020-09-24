---
title: "今日算法：20200924"
date: 2020-09-24T15:37:12+08:00
katex: true
---

# 第1题：  

>将$n(n>1)$个整数存放到一维数组$R$中。设计一个算法，将$R$中的序列循环左移$P(0<P<n)$个位置,即将R中的数据由$X_0, X_1,...,X_{n-1}$变换为$X_p, X_{p+1},...,X_{n-1}, X_0, X-1, ..., X_{p-1}$。

这道题还算比较简单，但是参考答案是比较牛逼的！

```python
def shift_lift01(n, p):
    arr = np.array(range(n))
    new_arr = np.concatenate((arr, arr))
    return new_arr[p:len(arr) + p]
```
第一种解法，我构造了一个长数组，直接读数组来左移，优势在于代码简洁。


```python
def shift_lift02(n, p):
    arr = np.array(range(n))
    new_arr = np.zeros(n, dtype=int)
    new_arr[-p:] = arr[:p]
    new_arr[:-p] = arr[p:]
    return new_arr
```
第二种解法就更简单了，直接将固定p位置的数字复制到指定位置，简单直接。


```python
def shift_lift00(n, p):
    arr = np.array(range(n))
    arr[:p] = arr[:p][::-1]
    arr[p:] = arr[p:][::-1]
    arr = arr[::-1]
    return arr
```
参考答案，这种思路不知道怎么想的，牛逼！但是我怀疑，这种算法的时间复杂度高。


# 第2题：
> 在有序表(12,24,36,48,60,72,84)中二分查找关键字72时所需进行的关键字比较次数是多少？