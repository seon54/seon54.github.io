---
layout: post
comments: true
title: JavaScript Array Method 1
category: JavaScript
shortinfo: filter, map, sort 짧은 정리
tags:
- javascript


---

## Array.prototype.filter()

`filter()`는 주어진 함수를 통과하는 요소를 새로운 배열로 만들어 반환한다.

예제에서는 filter를 통해 기존 numbers에 있는 숫자 중 홀수만 걸러서 새로운 배열을 만든다.

```javascript
const numbers = [1, 3, 4, 2, 9, 7, 8];

const oddNumbers = numbers.filter(number => number %2 === 1);

console.log(oddNumbers);
// [1, 3, 9, 7]
```



아래 예제에서는 continent가 'A'로 시작하는 결과만 새로운 배열로 만들었다.

```javascript
const countries = [
    {name : 'South Korea', continent : 'Asia'},
    {name : 'Canada', continent : 'North America'},
    {name : 'Thailand', continent : 'Asia'},
    {name : 'Morocco', continent : 'Africa'},
    {name : 'Venezuela', continent : 'South America'},
    {name : 'United Kingdom', continent : 'Europe'},
    {name : 'Greece', continent : 'Europe'},
]

const continentA = countries.filter(country => country.continent.substr(0,1) === 'A');

console.log(continentA);
/*
[	{name: "South Korea", continent: "Asia"},
	{name: "Thailand", continent: "Asia"},
	{name: "Morocco", continent: "Africa"}
]
*/
```



## Array.prototype.map()

`map()`에서는 주어진 함수의 대한 배열의 모든 요소의 결과를 새로운 배열로 반환한다.

예제에서는 numbers에 있는 숫자들의 거듭제곱을 새로운 배열로 만든다.

```javascript
const numbers = [1, 2, 3, 4, 5];

const square = numbers.map(number => Math.pow(number, 2));

console.log(square);
// [1, 4, 9, 16, 26]
```



아래 예제는 각 객체의 firstName과 lastName을 합쳐 새로운 배열을 만든다.

```javascript
const people = [
    { firstName : 'Jon', lastName : 'Snow' },
    { firstName : 'Tyrion', lastName : 'Lannister' },
    { firstName : 'Arya', lastName : 'Stark' },
    { firstName : 'Daenerys', lastName : 'Targaryen' }
];

const names = people.map(person => `${person.firstName} ${person.lastName}`);

console.log(names);
//  ["Jon Snow", "Tyrion Lannister", "Arya Stark", "Daenerys Targaryen"]
```



## Array.prototype.sort()

`sort()`은 배열의 요소를 정렬한 후 배열로 반환하며 기본 정렬 순서는 문자열의 유니코드 코드 포인트를 따른다.

```javascript
const numbers = [2, 4, 6, 23, 1, 7];
console.log(numbers.sort());
// [1, 2, 23, 4, 6, 7]

// 반환값이 0보다 작으면 a를 먼저 정렬
const ascending = numbers.sort((a, b) => a - b);
console.log(ascending);
// [1, 2, 4, 6, 7, 23]

// 반환값이 0보다 크면 b를 먼저 정렬
const descending = numbers.sort((a, b) => b - a);
console.log(descending);
// [23, 7, 6, 4, 2, 1]
```
