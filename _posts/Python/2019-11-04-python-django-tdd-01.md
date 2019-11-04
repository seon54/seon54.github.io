---
layout: post
comments: true
title: 파이썬을 이용한 클린 코드를 위한 테스트 주도 개발 1장
category: Python
shortinfo: django 프로젝트 생성, geckodriver 설치 및 git 레포지토리 생성
tags:
- python
- django
- TDD
---



## 1장

기존에 사용하던 anaconda에서 새로운 가상환경을 만들고 django와 selenium을 설치한다.

```shell
$ conda create -n tdd python=3.6
$ conda activate tdd
$ tdd> pip install django
$ tdd> pip install selenium
```

functional_tests.py 작성

```python
from selenium import webdriver

browser = webdriver.Firefox()
browser.get('http://localhost:8000')

assert 'Django' in browser.title
```

책에서 나온 AssertionError가 나오지 않고 `selenium.common.exceptions.WebDriverException: Message: 'geckodriver' executable needs to be in PATH. ` 에러가 발생하여 [geckodriver](https://github.com/mozilla/geckodriver/releases) 다운받아 functional_tests.py와 같은 경로에 설치하고 코드를 수정했다.

```python
import os
from selenium import webdriver

path = os.path.join(os.getcwd(), 'geckodriver')
browser = webdriver.Firefox(executable_path=path)
browser.get('http://localhost:8000')

assert 'Django' in browser.title
```

django 프로젝트를 생성 후, 개발 서버를 올린다. 다시 functional_tests.py를 실행하면 파이어폭스 창이 뜨고 커맨드에는 아무것도 뜨지 않는다.

```shell
$ tdd> django-admin startproject superlists
$ tdd> python manage.py runserver
$ tdd> python functinal_tests.py
```

functional_test.py를 superlists로 옮기고 git init을 실행한다. functional_tests.py 파일의 위치가 바뀌었기 때문에 path도 수정한다.


```shell
$ mv functional_tests.py superlists/
$ cd superlists
$ git init .
```

```python
import os
from selenium import webdriver

path = os.path.join(os.getcwd() + '/..', 'geckodriver')
browser = webdriver.Firefox(executable_path=path)
browser.get('http://localhost:8000')

assert 'Django' in browser.title
```

gitignore 파일을 만들어 git에 포함되지 않을 파일을 설정한다. Sqlite3 파일을 gitignore에 추가하고 모든 파일을 git에 추가한 후 상태를 확인하니 pyc 파일도 추가되었다.

```shell
$ echo 'db.sqlite3' >> .gitignore
$ git add .
$ git status
```

git에서 추가하지 않을 파일을 삭제하고 gitignore에도 추가한다. 상태 확인한 후 추가하고 커밋한다.

```shell
$ git rm -r --cached superlists/__pycache__
$ echo '__pycache__' >> .gitignore
$ echo '*.pyc' >> .gitignore
$ git status
$ git add .
$ git commit
```




