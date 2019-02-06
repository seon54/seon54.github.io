---
layout:post
title:Nodemon으로 Node.js 자동으로 재시작하기
shortinfo:Node application을 자동으로 재시작하는 nodemon
tags:
 - Node.js
---



## nodemon

> YouTube cloning 강의를 들으며 배운 내용을 복습합니다.



### nodemon?

파일에 변화가 있을 때 자동으로 node application을 재시작하는 것을 도와주는 도구이다. 사용법은 매우 간단하다. 코드나 개발의 방법에 변화를 줄 필요가 없이 스크립트를 실행할 때 명령어에서  `node` 를 `nodemon`으로 대체하면 된다.  

 `npm install --save-dev nodemon`  명령어를 입력하여 설치하면  package.json에서 'devDependencies'에 설치가 된 것을 확인할 수 있다. 



package.json의 **scripts**에서 nodemon을 사용하면 된다. scripts에서 여러 가지를 설정할 수 있다. 그 중, `start`의 명령어는 `npm start` 명령어를 통해 실행된다.  `--delay` 를 통해 재시작하는 것을 지연시킬 수 있다. 아래에서는 2ms 이후에 재시작하도록 설정했다. nodemon 설치 후 package.json에서 설정한 후에 `npm start` 명령어를 실행하고 나면 파일에 변화가 생길 때마다 자동으로 재시작한다.

```json
{
    "name": "myapp",
    "version": "1.0.0",   
    "dependencies": {
        ...
    },
    "scripts": {
        "start": "nodemon --exec app.js --delay 2"
    },
    "devDependencies": {
        "nodemon": "^1.18.9"
    }
}

```



참조  

[nodemon - npm](https://www.npmjs.com/package/nodemon)
