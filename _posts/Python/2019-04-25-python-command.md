---
layout: post
comments: true
title: Python 기본 명령어 알아보기
category: Python
shortinfo: Python 기본 명령어를 알아봅니다.
tags:
- Python
- Django

---



# Python 기본 명령어



django-admin startproject projectname` : 장고  프로젝트를 만드는 명령어

`python manage.py startapp appname` : 프로젝트에 기능 단위 앱을 생성하는 명령어

`python manage.py makemigrations` : 어플리케이션에 변경 사항을 추적해 DB에 적용할 내용을 정리하며 주로 앱의 model에 변경사항이 있을 때 사용

`python manage.py sqlmigrate` : 실행할 SQL 명령문 출력

`python manage.py migrate` : 변경 사항 DB 반영

`python manage.py showmigrations` : 프로젝트의 DB 변경사항 목록과 상태 출력

`python manage.py runserver` : 테스트 서버 실행

`python manage.py dumpdata` : 현재 DB 백업

`python manage.py loaddata` : 백업 파일에서 DB 복구

`python manage.py flush` : DB 테이블의 내용만 삭제

`python manage.py shell` : 장고 쉘 실행. 작성한 모델 테스트 가능

`python manage.py dbshell` : DB에 직접 접근할 수 있는 쉘 실행. SQL 구문을 이용해 직접 수정 가능

`python manage.py createsuperuser` : 관리자 계정 생성

`python manage.py changepassword` : 계정 비밀번호 변경  