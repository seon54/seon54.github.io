---
layout: post
comments: true
title: PyCharm에서 프로젝트 시작하기
category: Python
shortinfo: PyCharm에서 프로젝트 시작하는 방법과 기본 프로젝트 구조
tags:
- python
- django
---


## 프로젝트 생성하기

파이썬과 파이참을 설치한 후,  `File > New Project` 에 들어가면 `Create Project` 창이 뜬다. `Location`에서 위치와 프로젝트 이름을 설정한다. `Project Interpereter: New Virtual environment`의 왼쪽 화살표를 누르면 가상환경을 설정할 수 있다. 프로젝트 생성 후, Terminal에서 `pip install django` 명령어로 장고를 설치한다.

설치 후 명령어로 장고 프로젝트를 만든다. `django-admin startproject config . `  명령어에서는 반드시 config를 쓴 후 한 칸 띄고 .을 붙여야 한다. 명령어를 실행한 후에는 config 폴더와 manage.py 파일이 생성된다. 그 후 데이터베이스를 생성하는 `python manage.py migrate` 명령어를 실행한다. 이 명령어를 통해 DB를 초기화 하고 DB 파일을 생성하고 프로젝트에 db.sqlite3 파일이 생긴 것을 확인할 수 있다.


## 관리자 계정 생성하기

`python manage.py createsuperuser`로 관리자를 생성할 수 있다. 계정명, 이메일, 비밀번호를 입력하게 되어 있고 이메일은 입력하지 않아도 되며 비밀번호를 입력할 때는 터미널에서는 빈칸으로 나온다. 그 후 `python manage.py runserver`로 서버를 실행하여 localhost:8000 또는 127.0.0.1:8000 주소로 들어가면 장고가 만들어주는 메인 화면을 볼 수 있다. 위의 주소에 /admin을 추가하면 관리자 페이지로 갈 수 있으며 관리자를 생성할 때의 계정과 비밀번호를 입력하면 로그인할 수 있다.


## 프로젝트 구조 살펴보기

### config 폴더

- 프로젝트 설정 파일과 웹 서비스 실행을 위한 파일이 있다.
- `django-admin startproject` 명령어로 생성
- `__init__.py ` : 파이썬 2.x 버전과 호환을 위해 만들어진 빈 파일
- `settings.py`: 프로젝트 설정
- `urls.py`: url을 설정하며 config 폴더 안의 urls 파일이 기준 url 파일. 기준 url 파일은 settings.py에서 변경 가능
- `wsgi.py`: 웹 서비스를 실행하기 위한 wsgi 관련 내용

### venv 폴더

- 프로젝트 구동에 필요한 가상환경

### db.sqlite3

- SQLite3 DB 파일. 임의 삭제 또는 변경 불가

### manage.py

- 장고의 다양한 명령어 실행

### settings.py

- config 폴더에 위치, 프로젝트에 관한 다양한 설정

- BASE_DIR: 프로젝트 루트 폴더, 설정 파일 또는 py 파일 등에서 프로젝트의 루트 폴더를 찾아 그 하위를 탐색하는 일 수행
- SECRET_KEY: 세션 값의 보호나 비밀번호 변경시 사용되는 보안 URL을 만드는 등의 일에 주로 사용
- DEBUG: True일 경우 다양한 오류 메시지 즉시 확인할 수 있으며 배포시 False로 설정
- ALLOWED_HOSTS: 현재 서비스의 호스트 설정, 개발할 때는 비어두고 사용하나 배포시 '*'나 실제 도메인 사용. Debug 모드가 False일 경우 값이 비어있으면 서비스 시작 불가
- INSTALLED_APPS: 프로젝트에서 사용하는 앱의 목록 관리
- MIDDLEWARE: 장고의 요청/응답 메세지 사이에 실행되는 프레임워크들
- ROOT_URLCONF: 기준 urls.py 경로 설정
- TEMPLATES: 템플릿 시스템 설정. 템플릿 해석 엔진과 템플릿 폴더 경로 변경
- WSGI_APPLICATION: 실행을 위한 wsgi 어플리케이션 설정
- DATABASES: DB 설정
- AUTH_PASSWORD_VALIDATORS: 비밀번호 검증 설정
- LANGUAGE_CODE: 다국어 설정

### wsgi.py

WSGI는 Web Server Gateway Interface의 줄임말로서 wsgi 어플리케이션 구동을 위해 사용되는 파일로서 웹 서버와 장고 어플리케이션 사이의 통신 역할을 한다. 웹 서버와 장고 웹 어플리케이션 사이에서 미들웨어처럼 동작하여 웹 서버가 요청이 있을 경우 정보와 콜백함수를 WSGI에 전달하고, 이 정보를 해석하여 장고 웹 어플리케이션에 전달한다. 장고 웹 어플리케이션은 파이썬 스크립트를 이용해 정보를  처리하고 결과를 WSGI에 다시 전달하여 이 정보를 콜백함수를 이용해 웹 서버에 다시 전달하며 동작한다. 
