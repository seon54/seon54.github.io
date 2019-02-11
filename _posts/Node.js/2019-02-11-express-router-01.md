---
layout: post
title: Express Router 1
category: Node.js
shortinfo: Express Router의 기본 형태와 express.Router 살펴보기
tags:
- Node.js
- Express

---



# Express Router
> YouTube cloning  강의를 들으며 배운 내용을 복습합니다.

Routing은 URI 또는 특정 HTTP 요청 메소드(GET, POST 등)인 특정 엔드포인트의 클라이언트 요청에  어떻게 어플리케이션이 응답하는지를 결정하는 것을 말하며 각 라우트는 하나 이상의 handler 함수를 가질 수 있고 라우트가 일치할 때 실행된다.

기본 형태는 `app.METHOD(PATH, HANDLER)`이다. 

- app :  express instance
- METHOD : HTTP 요청 메소드
- PATH : 서버 상의 경로
- HANDLER : 라우트가 일치할 때 실행되는 함수



예제 
```javascript
// 애플리케이션 홈페이지에서 GET 요청에 'Welcome'으로 응답
app.get('/', (req, res) => res.send('Welcome!'))

// 애플리케이션 홈페이지에서 root 라우트의 POST 요청에 응답
app.post('/', (req, res) => res.send('Use post method'))
```



## express.Router

Router 클래스는 모듈의 마운팅이 가능한 라우트 핸들러를 만들기 위해 사용한다. Router 인스턴스는 완전한 미들웨어이자 라우팅 시스템이다. Router 클래스를 사용하면 아래 예제처럼 라우터를 분리할 수 있다. router.js에서 '/', '/join' 그리고 useRouter.js에서는 '/profile', '/change' 두 경로에 대한 라우트를 정의하고 app.js에서 두 라우터를 import 한 후 use 메소드를 사용하여 이용한다.  
'/', '/join'의 URI는 router.js에서 처리하고 '/user/profile', '/user/change'는 userRouter.js에서 처리하게 된다.

```javascript
// app.js
import express from "express";
import router from "./router";
import userRouter from "./userRouter";

var app = express();
...
app.use('/', router);
app.use('/user', userRouter);

```

```javascript
// router.js
import express from "express";

const router = express.Router();

router.get('/', (req, res) => res.send('Welcome home!'));
router.get('/join', (req, res) => res.send('Join'));

export default router;

```

```javascript
// userRouter.js
import express from "express";

const userRouter = express.Router();

userRouter.get('/profile', (req, res) => res.send('Check user profile'));
userRouter.get('/change', (req, res) => res.send('Change profile'));

export default userRouter;
```



참조  
[Express Routing](https://expressjs.com/en/guide/routing.html)  



