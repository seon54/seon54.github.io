---
layout: post
comments: true
title: (LeetCode Top Interview Questions-Easy) Remove Duplicates From Sorted Array
category: Algorithms
shortinfo: LeetCode Top Interview Questions(Easy) Remove Duplicates From Sorted Array 살펴보기
tags:
- leet code
---



## 26. Remove Duplicates From Sorted Array

정렬된 배열이 주어지면 중복을 없앤, 각 요소가 한 번씩 나올 때의 전체 길이를 반환한다. 이 문제에서 중요한 점은 다른 배열을 위한 공간을 할당하지 않도록 주의해야 하는 것이다.

```
Given a sorted array nums, remove the duplicates in-place such that each element appear only once and return the new length.
Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

Example 1:
Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.
It doesn't matter what you leave beyond the returned length.

Example 2:
Given nums = [0,0,1,1,1,2,2,3,3,4],

Your function should return length = 5, with the first five elements of nums being modified to 0, 1, 2, 3, and 4 respectively.
It doesn't matter what values are set beyond the returned length.
```

처음에 중복을 없애야 한다고 해서 set()을 떠올렸다. 집합으로 중복을 없애고 다시 리스트에 넣으면 되지 않을까라고 생각했다.

```python
class Solution:
  def removeDuplicates(self, nums):
    nums = list(set(nums))
    return len(nums)
```

하지만 이 방법은 추가 공간을 할당하지 말라는 주의사항을 어겨서 테스트를 넘기지 못했고 리스트 또한 정렬되지 않았다. 그러다 슬라이싱을 이용한 방법을 보게 되었다. 위의 풀이로 할 때 nums를 선언하기 전과 후에 `print(id(nums))` 로 해보면 다른 값이 나온다. 처음 나온 값은 removeDuplicates 함수의 매개변수로 들어간 nums의 값이고 두번째로 나온 값은 함수 내에서 새로 생성한 변수 nums의 값이다. 
아래 방법은 같은 방법으로 id() 결과값을 보면 같은 값이 나온다. 매개변수로 받은 nums를 그대로 이용해 전체 배열에 set()으로 중복된 값을 없애고 sorted()로 정렬한 배열을 할당했기 때문이다.

```python
class Solution:
  def removeDeuplicates(self, nums):
    nums[:] = sorted(set(nums))
    return len(nums)
```

두번째 방법은 굳이 중복된 요소를 지우지 않고 중복되지 않은 요소의 개수만 세서 반환한다. i는 0번째 인덱스부터, for loop의 j는 1번째 인덱스부터 시작하여 앞 뒤의 값을 비교한다. 값이 중복되지 않을 때마다 i를 1씩 증가하고 i번째 요소를 j번째 요소로 바꾼다. for loop가 끝나면 i가 0부터 시작했기 때문에 1을 더해서 반환한다.

```python
class Solution:
  def removeDuplicates(self, nums):
    i = 0
    for j in range(1, len(nums)):
      if nums[j] != nums[i]:
        i += 1
        nums[i] = nums[j]
    return i + 1
```

