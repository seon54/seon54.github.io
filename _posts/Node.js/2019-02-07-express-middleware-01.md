---
layout: post
title: Express Middleware - 1
category: Node.js
shortinfo: Express Middleware 시작하기, morgan & helmet
tags:
- Express
- Node.js
---



# Express Middleware (1)

> YouTube cloning  강의를 들으며 배운 내용을 복습합니다.



## Middleware?

Middleware 함수는 어플리케이션의 요청-응답 주기 내에서 request, response 그리고 다음의 middleware 함수에 대한 접근 권한을 가진다. 각각은 req, res, next로 사용한다.  

Middleware 함수는 아래의 기능을 가진다.

- 코드 실행
- request와 response 객체에 대한 변경 실행
- requiest-response 주기 종료
- 스택 내에서 다음 middleware 함수 호출

주의할 점은 현재 middleware 함수가 요청-응답 주기를 끝내지 않을 경우 **next()**를 호출해 다음 middleware 함수로 제어를 전달하도록 해야한다는 것이다. 



## morgan

morgan은 Node.js를 위한 HTTP request logger 미들웨어이다. `npm install morgan` 명령어로 설치한다. 강의에서는 Babel을 사용하였기 때문에 ES6의 문법을 사용했다. (참조 : [Babel](https://seon54.github.io/tool/2019/02/05/babel-01/)) 



{% highlight js %}

import express from "express";

import morgan from "morgan";



var app = express();



app.use(morgan('dev'));

app.listen(4000);

app.get("/", (req, res) => res.send("Hello World"));

{% endhighlight %}



morgan은 combined, common, dev, short, tiny 총 5개의 포맷을 선택할 수 있다.  설정할 때 나오는 결과는 아래와 같다.  

**combined**

`:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length] ":referrer" ":user-agent"`

**common**

`:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length]`

**dev**

`:method :url :status :response-time ms - :res[content-length]`

**tiny**

`:method :url :status :res[content-length] - :response-time ms`

**short**

`:remote-addr :remote-user :method :url HTTP/:http-version :status :res[content-length] - :response-time ms`



위의 코드를 실행하면 포맷이 dev이므로 터미널에서  `GET / 304 2.982 ms - -`  이 결과를 통해 메소드, url, 상태 코드, 응답 시간 등을 확인할 수 있다.



## helmet

helmet은 다양한 HTTP 헤더를 설정함으로써 Express application을 보호한다. `npm install helmet` 명령어로 설치한 후 아래와 같이 사용한다. helmet은 미들웨어 스택에서 먼저 사용하는 것이 좋다.



{% highlight js %}

import express from "express";

import helmet from "helmet";



var app = express();



app.use(helmet());

...

{% endhighlight %}



참조  
[Writing Middleware](https://expressjs.com/en/guide/writing-middleware.html)  
[morgan - npm](https://www.npmjs.com/package/morgan)  
[helmet - npm](https://www.npmjs.com/package/helmet)  

