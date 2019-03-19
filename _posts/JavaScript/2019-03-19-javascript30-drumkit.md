---
layout: post
title: JavaScript30 - Drum Kit
category: JavaScript
shortinfo: wesbos의 JavaScript30을 듣고 정리한 내용
tags:
- JavaScript
---



# JavaScript30 - Drum Kit

> wesbos의 JavaScript30(https://javascript30.com/)을 듣고 공부한 내용을 정리합니다.



`keydown`  이벤트는 키가 눌릴 때 발생하며 `keypress`와 다르게 모든 키에 적용된다. 

```javascript
// 강의 예제
window.addEventListener('keydown', playSound);

function playSound(e) {
    ...
}
```

playSound에서 console.log(e)로 확인하면 KeyboardEvent에서 여러 프로퍼티를 확인할 수 있다. key에서는 내가 누른 키를 a, Enter식으로 확인할 수 있고 강의에서 사용한 keyCode로 각 키의 코드를 알 수 있다.



`transitioned` 이벤트는 CSS transition이 완료될 때 발생한다.

```javascript
function removeTransition(e) {    
                
	if (e.propertyName !== 'transform') return; 
	...
}

const keys = document.querySelectorAll('.key');
keys.forEach(key => key.addEventListener('transitionend', removeTransition));
```

removeTransition에서 e를 확인하면 TransitionEvent에서 각 transition 정보를 알  수 있다. propertyName에서 box-shadow, transform 등으로 어떤 transition인지 확인할 수 있다.





