---
layout: post
comments: true
title: Express Application Methods - listen & get
shortinfo: Express의 Application Method <strong>listen</strong>과 <strong>get</strong> 알아보기
category: Node.js
permalink: /nodejs/:year/:month/:day/:title/
tags:
- express
- node.js

---

> YouTube cloning  강의를 들으며 배운 내용을 복습합니다.

## app.listen([port[, host[,backlog]]][,callback])
Node의 http.Server.listen() 메소드와 같은 역할을 한다. 만약 포트를 생략하거나 0일 경우,  임의로 사용되지 않은 포트를 지정한다. 



{% highlight js %}
var express = require('express');
var app = express();

app.listen(3000);
{% endhighlight %}



## app.get(path, callback [, callback ...])

callback 함수와 함께 지정한 path로 HTTP GET 요청을 라우팅한다.

|  전달인자  | 설명                                                         | 기본값 |
| :--------: | :----------------------------------------------------------- | :----: |
|   `path`   | 미들웨어 함수가 호출되는 path는 아래와 같다. <br>1. path를 나타내는 문자열<br>2. path 패턴<br> 3. path에 맞는 정규표현식<br> 4. 위의 조합에 해당되는 배열 <br> [예제](https://expressjs.com/ko/4x/api.html#path-examples) |  '/'   |
| `callback` | callback 함수는 아래와 같다.<br>1. 미들웨어 함수<br> 2. comma로 구분된 연속된 미들웨어 함수<br> 3. 미들웨어 함수의 배열<br> 4.위의 모든 것의 조합<br>[예제](https://expressjs.com/ko/4x/api.html#middleware-callback-function-examples) |  none  |



참조

[Express API reference](https://expressjs.com/en/4x/api.html#app)

