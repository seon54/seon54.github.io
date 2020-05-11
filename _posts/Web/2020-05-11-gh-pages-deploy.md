---
layout: post
comments: true
title: Github Pages에 배포하기
category: Web
shortinfo: 간단하게 Github Pages에 배포하는 방법 알아보기
tags:
- github
- deploy
---



노마드 코더에서 리액트 기초 강의를 듣으며 만든 movie app을 github pages에 배포했는데 정말 빠르고 간단했지만 잊어버릴지도 모를 미래의 나를 위해 기록하도록 한다.



### 1. gh-pages 패키지

먼저 관련 패키지를 받아주도록 한다. 

```shell
npm install gh-pages
```



### 2. Github repository

연결된 github repository가 없었기 때문에 react-movies라는 이름으로 만든 후, 안내된 명령어를 실행한다.

```shell
git init
git remote add origin https://github.com/{id}/{repository}.git
```



### 3. package.json

package.json에 스크립트와 홈페이지에 대한 설정을 추가해야 한다. `npm run build` 명령어를 실행하게 되면 build 폴더가 생성된다. `npm run deploy` 는 gh-pages를 이용해 배포하는데 build 디렉토리를 이용하며 이 명령어를 실행하게 되면 `predeploy`도 실행하게 된다. 그렇기 때문에 build 명령어를 하지 않아도 단계적으로 build 폴더가 생성되고 그 후에 배포로 진행된다.

```json
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "deploy": "gh-pages -d build",
    "predeploy": "npm run build"
  }
```

마지막으로 `hompage`에 주소를 적어주도록 한다. 주소의 형식은 아래와 같아야 한다.

```json
"homepage": "https://{id}.github.io/{repository}"
```



### 4. 배포

`npm run deploy` 명령어를 실행하고 터미널에서 Published를 확인한다. 배포가 되었으니 homepage에 입력한 주소에 가보면 내가 만든 앱을 확인할 수 있다.