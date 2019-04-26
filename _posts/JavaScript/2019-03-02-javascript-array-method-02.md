---
layout: post
comments: true
title: JavaScript Array Method 2
category: JavaScript
shortinfo: some, every, find, findIndex 짧은 정리
tags:
- JavaScript

---



# JavaScript Array Method 2 



## Array.prototype.some()

`some()`는 배열의 어떤 요소라도 주어진 함수를 통과하면 true를 반환한다.

예제에서는 food에 burger가 포함되어 있지 않기 때문에 false를 반환한다.

```javascript
const food = ['bread', 'latte', 'sushi', 'pasta'];

const hasBurger = food.some(val => val === 'burger');

if (hasBurger) {
    console.log('I love 🍔!');
} else {
    console.log('You need 🍔');
}
// You need 🍔
```

배열에 object를 넣어서 pets라는 배열에 type이 cat인 요소가 하나라도 있으면 true를 반환한다.

```javascript
const pets = [
    { name : 'honggu', 	type : 'dog' },
    { name : 'ssong', 	type : 'cat' },
    { name : 'donggu', 	type : 'dog' }
]

const hasCat = pets.some(pet => pet.type === 'cat');

if (hasCat) {
    console.log('Meow 🐱');
}
// Meow 🐱
```



## Array.prototype.every()

`every()`에서는 `some()`가 다르게 배열의 모든 요소가 주어진 함수를 통과해야만 true를 반환한다.

예제에서는 numbers에 있는 숫자들이 모두 짝수이면 true를 그렇지 않으면 false를 반환한다.

```javascript
const numbers = [2, 4, 6, 8, 10];

const isEvenNumber = numbers.every(number => number % 2 === 0);

console.log(isEvenNumber);	// true
```



## Array.prototype.find()

`find()`는 판별 함수를 충족하는 첫 번째 요소를 반환하고 없다면 undefined를 반환한다.

예제에서는 like에 type이 dessert인 요소가 두 개가 있지만 `console.log(likeDessert)` 의 결과로 그 중 첫 번째 요소만 반환한다. `console.log(likeEspresso)`를 할 경우 type이 espresso인 요소가 없으므로 undefined를 반환한다.

```javascript
const like = [
    { name : "cat", 		type : "animal"},
    { name : "ice cream", 	type : "dessert"},
    { name : "latte", 		type : "coffee"},
    { name : "Ghibli", 		type : "animation"},
    { name : "macaron", 	type : "dessert"}
];

const likeDessert = like.find(val => val.type === 'dessert');
const likeEspresso = like.find(val => val.type === 'espresso');

console.log(likeDessert); 	// {name: "ice cream", type: "dessert"}
console.log(likeEspresso);	// undefined
```



## Array.prototype.findIndex()

`findIndex()`는 판별 함수를 충족하는 배열의 첫 번째 요소의 인덱스를 반환하며 없을 경우 -1을 반환한다.

예제에서는 exam에서 grade의 값이 60보다 작은 객체가 존재하므로 그 인덱스인 4를 반환한다.

```javascript
const exam = [
    { subject: 'Korean', 	grade: 95 },
    { subject: 'English', 	grade: 80 },
    { subject: 'History', 	grade: 73 },
    { subject: 'Math', 		grade: 86 },
    { subject: 'Science', 	grade: 53 }
];

const underSixty = exam.findIndex(val => val.grade < 60);

console.log(underSixty);		// 4
console.log(exam[underSixty]);	// {subject: "Science", grade: 53}
```

