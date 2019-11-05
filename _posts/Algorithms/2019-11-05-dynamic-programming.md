---
layout: post
comments: true
title: 알고리즘 - 다이나믹 프로그래밍(Dynamic Programming)
category: Algorithms
shortinfo: 다이나믹 프로그래밍에 대한 설명과 간단한 예제
tags:
- algorithms

---



## 다이나믹 프로그래밍

- 큰 문제를 작은 문제로 나눠서 푸는 알고리즘

- Dynamic은 아무 의미가 없으며 처음 사용한 Richard Bellman이 멋있어보여 사용

- 큰 문제를 작은 문제로 나눌 때, 작은 문제는 중복 발생

  

두 가지 속성을 만족해야 다이나믹 프로그래밍으로 문제를 풀 수 있다.

**Overlapping Subproblem**(중복되는 부분 문제)

- 큰 문제를 작은 문제로 나누고 작은 문제가 중복이 된다. (예: 피보나치 수)

**Optimal Substructure**(최적 부분 구조)

- 문제의 정답이 작은 문제의 정답을 통해 구할 수 있다.
  - 예제: 서울에서 부산을 가는 가장 빠른 길이 대전과 대구를 순서대로 거쳐야 한다면, 대전에서 부산을 가는 가장 빠른 길은 대구를 거치게 된다.

- Optimal Substructure를 만족한다면, 문제의 크기에 상관없이 어떤 한 문제의 정답은 일정하다.
  - 예제: 10번째 피보나치 수를 구하면서 구한 4번째 피보나치 수와 5번째 피보나치 수를 구하면서 구한 4번째 피보나치 수는 동일



다이나믹 프로그래밍에서 각 문제는 한 번만 풀어야 하며 Optimal Substructure를 만족하므로 같은 문제는 구할 때마다 정답이 같다. 구한 답을 배열에 저장하여 사용할 수 있는데 이를  메모이제이션(Memoization)이라 한다.



```python
# 시간복잡도 O(n²)

def fibonacci(n):
	if n <= 1:
    return n
  else:
    return fibonacci(n-1) + fibonacci(n-2)
  
# memoization 추가, 시간복잡도 O(n)

memo = []
def fibonacci2(n):
  if n <= 1:
    return n
  else:
    if memo[n] > 0:
      return memo[n]
    memo[n] = fibonacci2(n-1) + fobonacci2(n-2)
    return memo[n]
```



구현 방식

1. Top-down: 큰 문제부터 문제를 쪼개가며 작은 문제를 만들어 합쳐가며 문제 해결(재귀)
2. Bottom-up: 작은 문제를 모아서 큰 문제를 만드는 것을 반복(반복문)

 두 방법의 시간 차이는 알 수 없다. 파이썬에서는 재귀를 사용할 때 스택오버플로우가 발생할 수 있어 주의해야 한다.

```python
# bottom-up

d = []
def fibonacci(n):
  d[0] = 0
  d[1] = 1
  for i in range(2, n+1):
    d[i] = d[i-1] + d[i-2]
  return d[n]
```





