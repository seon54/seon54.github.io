---
layout: post
comments: true
title: (LeetCode Top Interview Questions-Easy) Roman to Integer
category: Algorithms
shortinfo: LeetCode Top Interview Questions(Easy) Roman to Integer 살펴보기
tags:
- leet code
---



## 13. Roman to Integer

Roman to Integer는 어떤 방법을 몇 시간을 붙들고 있어도 어떤 방법을 사용해야할지 감이 잡히지 않아 다른 사람들의 풀이를 참고했다.

```
Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M.

Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
For example, two is written as II in Roman numeral, just two one's added together. Twelve is written as, XII, which is simply X + II. The number twenty seven is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

I can be placed before V (5) and X (10) to make 4 and 9. 
X can be placed before L (50) and C (100) to make 40 and 90. 
C can be placed before D (500) and M (1000) to make 400 and 900.
Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.

Example 1:

Input: "III"
Output: 3
Example 2:

Input: "IV"
Output: 4
Example 3:

Input: "IX"
Output: 9
Example 4:

Input: "LVIII"
Output: 58
Explanation: L = 50, V= 5, III = 3.
Example 5:

Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

로마 숫자를 정수로 바꾸는 문제이다. IV, CD 같이 두번째 문자에서 첫번째 문자를 빼는 숫자를 주의해야 한다. dictionary 자료형을 사용하여 key에는 문자를 value에는 숫자가 들어가도록 했다. if, else에서 IV, CD 같은 부분을 확인한다.

```python
class Solution:
    def romanToInt(self, s: str) -> int:
        roman_dict = {"I": 1, "V": 5, "X": 10, "L": 50, "C": 100, "D": 500, "M": 1000}
        result = 0
        
        for i in range(len(s)-1):
            if roman_dict[s[i]] >= roman_dict[s[i+1]]:
                result += roman_dict[s[i]]
            else:
                result -= roman_dict[s[i]]
        
        result += roman_dict[s[-1]]
        return result
```

두번째 방법은 똑같이 dictionary를 사용하고 IV, CD 같은 부분을 해당 숫자를 의미하는 문자로 바꾼 후 문자열을 for문에서 돌면서 값을 더했다.

```PYTHON
class Solution:
    def romanToInt(self, s: str) -> int:
        translations = {
            "I": 1,
            "V": 5,
            "X": 10,
            "L": 50,
            "C": 100,
            "D": 500,
            "M": 1000
        }
        number = 0
        s = s.replace("IV", "IIII").replace("IX", "VIIII")
        s = s.replace("XL", "XXXX").replace("XC", "LXXXX")
        s = s.replace("CD", "CCCC").replace("CM", "DCCCC")
        for char in s:
            number += translations[char]
        return number
```

