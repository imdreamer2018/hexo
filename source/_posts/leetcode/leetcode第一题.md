title: leetcode第一题——两数之和
author: 追梦人

toc: true

date: 2019-03-25 18:13:00

categories: []

tags:

  - python
  - leetcode

---

给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 **两个** 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

**示例:**

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

<!--more-->

#### 暴力法

看到这个题的第一想法是暴力循环，采用两次循环，遍历每个元素x，查找是否存在两个值之和等于target。

```python
class Solution:
    def twoSum(self, nums, target):
        for i in range(len(nums) - 1):
            for j in range(i + 1, len(nums)):
                if nums[i] + nums[j] == target:
                    return [i,j]
```

**复杂度分析：**

- 时间复杂度为：O(n^2)。对于每个元素，我们试图通过遍历数组的其余部分来寻找它所对应的目标元素，这将耗费 O(n)的时间。因此时间复杂度为 O(n^2)

- 空间复杂度为：O(1)

#### 两遍哈希表

在python中哈希表就是字典，两遍哈希表，第一遍通过遍历将数组的值存入哈希表，其中key为数组的值，value为这个值对应的数组下标。在第二次迭代中，我们将检查每个元素所对应的目标元素(target-nums[i])是否存在于表中。注意，该目标元素不能是nums[i]本身！

```python
class Solution:
    def twoSum(self, nums, target):
        dict = {}
     	for i in range(len(nums)):
            dict[str(nums[i])] = i
        for i in range(len(nums)):
            if str(target-nums[i]) in dict and dict.get(str(target-nums[i])) != i:
                return [i,dict.get(str(target-nums[i]))]
```

**复杂度分析**

- 时间复杂度为：O(n)。我们把包含有 n 个元素的列表遍历两次。由于哈希表将查找时间缩短到 O(1) ，所以时间复杂度为

- 空间复杂度为：O(n)。所需的额外空间取决于哈希表中存储的元素数量，该表中存储了 n*n* 个元素

#### 一遍哈希表

事实证明，我们可以一次完成。在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回。

```python
class Solution:
    def twoSum(self, nums, target):
        dict = {}
        for i in range(len(nums)):
            if str(target-nums[i]) in dict:
                return [i,dict.get(str(target-nums[i]))]
        	dict[str(nums[i])] = i                               
```

**复杂度分析：**

- 时间复杂度：O(n)。我们只遍历了包含有 n 个元素的列表一次。在表中进行的每次查找只花费 O(1)的时间。

- 空间复杂度：O(n)， 所需的额外空间取决于哈希表中存储的元素数量，该表最多需要存储 n个元素。

#### 值得注意的情况

我们应该考虑程序的稳定性，如果这道题，输入一个空数组，那么这个程序就会马上崩溃，所以最好加上一个判断。

```python
class Solution:
    def twoSum(self, nums, target):
        if len(nums) == 0:
            return False
        ***
```

#### 总结

关于python中哈希表（字典）的用法

| Python字典内置方法                                           |                           |
| ------------------------------------------------------------ | ------------------------- |
| 创建一个新字典，以序列 seq 中元素做字典的键val 为字典所有键对应的初始值 | dict.fromkeys(seq[, val\] |
| 删除字典内所有元素                                           | dict.clear()              |
| 返回指定键的值，如果值不在字典中返回default值                | dict.get()                |
| 如果键在字典dict里返回true，否则返回false                    | key in dict               |
| 以列表返回可遍历的(键, 值) 元组数组                          | dict.items()              |
| 以列表返回一个字典所有的键                                   | dict.keys()               |
| 和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default | dict.setdefault()         |
| 把字典dict2的键/值对更新到dict里                             | dict.update()             |
| 以列表返回字典中的所有值                                     | dict.values()             |

