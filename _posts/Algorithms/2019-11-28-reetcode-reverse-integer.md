---
layout: post
comments: true
title: (LeetCode Top Interview Questions-Easy) Reverse Integer
category: Algorithms
shortinfo: 
tags:
- leet code
---



매일 코딩 테스트 문제를 하나씩 풀어야지 마음 먹었지만 꾸준히 하는 것은 쉽지 않았다. 앞으로 LeetCode 문제 중 약 50개 정도의 Top Interview Questions의 난이도 easy 문제를 풀어보려고 한다. 50일보다 시간이 더 걸리더라도 모든 문제를 끝까지 풀 수 있기를!



## 7. Reverse Integer

```
Given a 32-bit signed integer, reverse digits an integer.
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−2^31,  2^31 − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

Example 1:
Input: 123
Output: 321

Example 2:
Input: -123
Output: -321

Example 3:
Input: 120
Output: 21
```

부호있는 32비트 정수가 주어지면 정수로 뒤집는 문제이다. 정수의 범위가 정해져 있고, 뒤집은 결과가 그 범위에 맞지 않으면 0을 반환하게 된다.

주어진 정수 x가 0보다 큰 지 아닌지 구분해서 문자로 바꿔서 슬라이싱을 하고 다시 정수로 바꾼 값을 result에 할당했다. 그리고 한 번 더 if문에서 result가 주어진 범위에 있는지 확인하였다. 

```python
class Solution:
    def reverse(self, x: int) -> int:
        if x < 0:
            result = int(str(x)[:0:-1]) * -1
        else:
            result = int(str(x)[::-1])

        if result not in range(-(pow(2, 31), pow(2, 31))):
            return 0

        return result
```

문자로 바꿔서 슬라이싱을 하고 다시 정수로 바꾼 과정을 좀 더 깔끔하고 보기 좋게 할 수 있는 방법을 알고 싶어 다른 풀이를 찾아보았다.

첫번째 풀이에서는 sign 변수로 음수, 양수를 처리하는데 `[x < 0]`에서 False이면 0, True이면 1이라는 점을 잘 활용했다.  결과값 rst가 범위 안에 있는지 확인하는 것도 return 할 때 if로 처리해서 코드를 더 줄일 수 있었다.

```python
class Solution:
    def reverse(self, x: int) -> int:       
        sign = [1,-1][x < 0]
        rst = sign * int(str(abs(x))[::-1])
        return rst if -(2**31)-1 < rst < 2**31 else 0
```

두번째 풀이는 몫과 나머지를 잘 활용한 경우였다. 내가 잘 사용하지 못하는 방법이기 때문에 더 멋지게 느껴졌던 풀이였다. while문 안에서 10으로 나눈 나머지를 결과값에 더하고 또 결과값에 10을 곱하면서 마지막 숫자부터 result에 들어가게 된다. 풀이에는 pop, push로 설명이 되어 있다. result에 값을 할당한 후에 x에 10으로 나눈 몫을  할당해야 하는데 `/=`  연산자가 아닌 `//=` 연산자를 사용하도록 주의해야 한다. while문을 돌면서 정수 x의 값에서 10을 나눈 나머지가 result에 push 되는 과정이기 때문에 `/=` 연산자를 사용하면 한 자리 수가 남을 때 결과가 음수가 된다.

```python
class Solution:
    def reverse(self, x: int) -> int:
        result = 0
        sign = 1
        
        if x < 0:
            sign = -1
            x *= -1
        
        while x:
            result = result * 10 + x % 10
            x //= 10
        
        return 0 if result > pow(2, 31) or result < -pow(2, 31) else result * sign
```

