---
layout: post
title: Express (1) 시작하기
shortinfo: Express 설치하기
tags:
- Node.js
---

## Express 설치

> YouTube cloning  강의를 들으며 배운 내용을 복습합니다

먼저 [Node.js](https://nodejs.org/ko/)가 설치되어 있어야 한다. Node.js를 설치하면 [npm](https://www.npmjs.com/)도 함께 설치된다. npm 홈페이지에는 JavaScript를 위한 패키지 매니저로서 코드를 설치, 공유, 배포할 수 있으며 dependency를 관리할 수 있다라고 설명이 되어 있다. 아직 많이 사용해보지는 않았지만 JavaScript와 관련된 것들을 `npm install` 명령어로 간단하게 설치할 수 있어서 익숙해지면 더 많이 활용할 수 있을 것 같다. 어플리케이션을 만들 폴더를 만들고 폴더에서 명령어를 실행한다.

`npm init`   명령어를 실행하면 [package.json](https://docs.npmjs.com/files/package.json) 파일이 폴더에 생성된다.
express를 설치하기 전에 먼저 실행해야 할 명령어이다. 명령어를 실행하면 터미널에서 자동으로 package.json의 항목들을 설정할 수 있다. package.json에는 나의 프로젝트가 의존하는 패키지들이 나열되어 있으며 **name**과 **version**이 반드시 있어야 한다. name은 소문자로 한 단어여야 하며 하이픈(-)이나 언더스코어(_)가 포함될 수 있다. version은 x.x.x의 형태여야 한다. `npm install express` 명령어를 실행 후 express가 설치가 되면서 package.json에서 dependencies에 추가된 것을 확인할 수 있다. version, main, scripts, author, license 등은 설정을 하지 않을 경우 아래와 같이 나오게 된다.

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "description": "Node.js app with express",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.4"
  }
}

```

