---
layout: post
comments: true
title: (LeetCode Top Interview Questions-Easy) Single Number
category: Algorithms
shortinfo: LeetCode Top Interview Questions(Easy) Single Number 살펴보기
tags:
- leet code
---



## 136. Single Number

숫자로 이루어진 배열이 주어지고 개수가 한 개인 요소를 찾는 문제다. 

```
Given a non-empty array of integers, every element appears twice except for one. Find that single one.

Note:

Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

Example 1:

Input: [2,2,1]
Output: 1
Example 2:

Input: [4,1,2,1,2]
Output: 4
```

어떻게 풀어야 하나 고민하다가 예전에 봤던 파이썬 라이브러리 중 collections.Counter가 떠올라서 Counter를 사용했다.

```python
from collections import Counter

class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        d = Counter(nums)
        for k, v in d.items():
            if v == 1:
                return k
```

파이썬 라이브러리를 사용하지 않고 풀 수 있는 방법을 leet code에서 제공한 네 가지 방법에서 알아보았다.

##### List operation

no_duplicate_list라는 중복값이 없을 리스트를 만든다. 주어진 nums 리스트를 for loop를 돌리면서 no_duplicate_list 리스트에 없으면 넣고 리스트에 있다면 그 값을 지우도록 한다. 새로운 리스트를 만들어서 매우 간단하게 해결했다.

```python
class Solution:
    def singleNumber(self, nums):
        no_duplicate_list = []
        for i in nums:
            if i not in no_duplicate_list:
                no_duplicate_list.append(i)
            else:
                no_duplicate_list.remove(i)
        return no_duplicate_list.pop()
```

##### Hash Table

리스트와 마찬가지로 for loop를 돌면서 try-except 구문에서 hash_table이라는 딕셔너리에 값이 있으면 빼고(pop) 없으면 그 값을 key로 넣는다. 딕셔너리의 popitem() 메서드는 LIFO 순서로 (key, value) 쌍을 제거하고 돌려준다. 여기서는 key만 알면 되기 때문에 popitem()의 결과에서 0번째 값만 가져오도록 hash_table.popitem()[0]을 반환한다.

```python
class Solution:
    def singleNumber(self, nums):
        hash_table = {}
        for i in nums:
            try:
                hash_table.pop(i)
            except:
                hash_table[i] = 1
        return hash_table.popitem()[0]
```

##### Math

단 한 개의 요소를 제외하고 다른 요소는 모두 두 개씩 있기 때문에 nums에 있는 모든 숫자를 중복없이 더한 값에 2를 곱하고 nums의 모든 값을 더해서 빼면 하나만 있는 요소가 남게 된다.

> 2 * (a + b + c) - (a + a + b + b + c) = c  

```python
class Solution:
    def singleNumber(self, nums):
        return 2 * sum(set(nums)) - sum(nums)
```

##### Bit Manipulation

비트 연산자를 이용한 방법이다.  `^`연산자는 두 값이 같으면 0이 된다. 비트 연산자를 활용한 방법이 조금 헷갈렸는데 num에 있는 숫자를 2진수로 변환한 후 a ^= i 과정에 대입하니 과정을 이해할 수 있었다. 비트 연산자를 처음 공부할 때 언제 사용할 수 있을지 궁금했는데 멋진 해결 방법 중 하나라고 생각한다.

> a ^ 0 = a
>
> a ^ a = 0
>
> a ^ b ^ a = (a ^ a)^ b = 0 ^ b = b

```python
class Solution:
    def singleNumber(self, nums):
        a = 0
        for i in nums:
            a ^= i
        return a
```

