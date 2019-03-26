title: leetcode第二题——整数反转
author: 追梦人
tags:

  - python

  - leetcode

  categories: []
  date: 2019-03-26 14:46:00

---

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

**示例 1:**

```
输入: 123
输出: 321
```

 **示例 2:**

```
输入: -123
输出: -321
```

**示例 3:**

```
输入: 120
输出: 21
```

**注意:**

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−2^31,  2^31 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

<!-- more -->

#### 暴力取余法

以前这种题在C预言中，一般采用取余取整的方法，利用while循环，反转整数。下面是我开始写的方法，由于在python中不用考虑溢出的现象，但是还是需要做出一个判断。

```python
class Solution:
    def reverse(self,x):
        result = 0
        if x < -2**31 or x > 2**31-1 or x == 0:
            return 0
        if x >= 0:
            while x != 0:
                result = result*10 + x % 10
                x = x // 10
                if result < -2**31 or result > 2**31-1:
                    return 0
            return result
        else:
            x = -x
            while x != 0:
                result = result*10 + x % 10
                x = x // 10
                if result < -2**31 or result > 2**31-1:
                    return 0
            return -result
```

后来考虑到这种方法，代码太冗余，就改成下面这种。

```python
class Solution:
    def reverse(self,x):
        result = 0
        if x < -2**31 or x > 2**31-1 or x==0:
            return 0
        a = abs(x)
        while a != 0:
            if (result >= 2**31) // 10:
                return 0
            result = result*10 + a % 10
            a = a // 10
        result = result * (x // abs(x))
        return result
```

**复杂度分析：**

时间复杂度：O(lnx)

空间复杂度：O(1)

#### 切片法

可以利用python的切片机制来进行反转。

```python
class Solution:
    def reverse(self,x):
        if x == 0 or x < -2 ** 31 or x > 2 ** 31 - 1 :
            return 0
        x = (x // abs(x)) * int(str(abs(x))[::-1])
        return x if -2 ** 31 < x < 2 ** 31 - 1 else 0
```

在python切片中，str[:]可以表示整个字符串，str[::-1]，后面加上:-1表示从倒数每一个进行分片，就可以实现字符串的反转。