---
title: "刷LeetCode"
date: 2020-09-18T11:21:55+08:00
draft: true
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

```python
def binary_search(arr, tar, l_index, r_index):
    mid_index = l_index + int((r_index - l_index)/2)
    mid_value = arr[mid_index]

    if tar < mid_value:
        return binary_search(arr, tar, l_index, mid_index-1)
    elif tar == mid_value:
        return mid_index
    else:
        return binary_search(arr, tar, mid_index+1, r_index)
```
二分查找的代码

# 第3题：
> 归并排序

```python
def merge_sort_recursive(arr):

    arr_len = len(arr)
    mid = int(arr_len/2)

    # 排序好左边
    if len(arr[:mid]) == 1:
        l_arr = arr[:mid]
    else:
        l_arr = merge_sort_recursive(arr[:mid])

    # 排序好右边
    if len(arr[mid:]) == 1:
        r_arr = arr[mid:]
    else:
        r_arr = merge_sort_recursive(arr[mid:])

    merge_arr = np.zeros(arr_len)
    l_i = 0
    r_i = 0
    for m_i, m_v in enumerate(merge_arr):
        l_v = l_arr[l_i]
        r_v = r_arr[r_i]
        if l_v < r_v:
            merge_arr[m_i] = l_v
            l_i += 1
            if l_i == len(l_arr):
                merge_arr[m_i+1:] = r_arr[r_i:]
                break
        else:
            merge_arr[m_i] = r_v
            r_i += 1
            if r_i == len(r_arr):
                merge_arr[m_i+1:] = l_arr[l_i:]
                break

    return merge_arr

arr = np.array([0,4,3,0,10,7,9])
arr_sorted = merge_sort_recursive(arr)
print(arr_sorted)
```
结果显示如下：
```sh
[ 0.  0.  3.  4.  7.  9. 10.]
```

# 第4题：LeetCode-1

> 两数之和:给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例：
```
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```
## 我的解法：笨方法

```python
class Solution:
    def twoSum(self, nums, target):
        for i, i_v in enumerate(nums):
            j_v = target - i_v
            fake_j = self.plus_search(nums, j_v, i+1, len(nums))
            if fake_j == None:
                continue
            else:
                return [i, fake_j]


    def plus_search(self, arr, tar, l_index, r_index):
        for j in range(l_index, r_index):
            if arr[j] == tar:
                return j
        return None


s = Solution()

arr = [3,2,3]
tar = 6

res = s.twoSum(arr, tar)
print(res)
```
这种方法即，直接从后续的数组中搜索指定元素。