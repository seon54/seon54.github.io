---
layout: post
comments: true
title: Pythonanywhere에서 배포하기
category: Python
shortinfo: Pythonanywhere에서 배포하는 방법을 알아봅니다.
tags:
- python
---



# Pythonanywhere에서 배포하기



## GitHub

먼저 GitHub에서 새 레포지토리를 만든다. 배포할 프로젝트에 `.gitignore` 파일을 만든 후 아래 내용을 추가한다. 또는 gitignore를 검색하면 각 언어마다 추가할 내용을 확인할 수 있다.

```python
*.pyc
*~
/venv
__pycache__
db.sqlite3
.DS_Store
```

마지막으로 settings.py 파일의 내용을 변경한다.

```python
# settings.py

DEBUG = False

ALLOWED_HOSTS = ['*']
```

터미널에서 아래의 깃 명령어로 코드를 업로드 한다. (yourname과 repo는 각자 깃헙 이름과 레포지토리)

```git
$ git add -A
$ git commit -m"your commit message"
$ git remote add origin https://github.com/yourname/repo.git
$ git push -u origin master
```



## Pythonanywhere

[pythonanywhere](https://www.pythonanywhere.com)에서 [Pricing & signup]을 누르면 요금제를 선택할 수 있으며 Create a Beginner account를 누르면 된다.  가입을 마친 후, [Dashboard]에서 New Cosole 밑에 `$Bash`를 누르면 온라인 콘솔이 나타난다. 

### 콘솔에서 배포 설정

```
$ pwd
```

위의 명령어를 통해 현재 경로가 `/home/아이디`인지 확인한다.

```
$ git clone https://github.com/yourname/repo.git
```

깃헙에서 소스를 다운받으면 레포지토리 이름의 폴더가 생기고 그 안에 배포하려는 프로젝트의 소스가 다운로드 되어 있다. 그 다음, 가상 환경을 만들기 위해 폴더를 이동하고 폴더 안에 가상 환경을 만들기 위해 virtualenv 명령어를 사용한다. venv은 가상환경 폴더명이고 뒤에는 파이썬 버전을 설정한다.

```
$ cd repo
$ virtualenv venv --python=python3.7
```

가상환경을 활성화하고 장고를 설치한다. 장고를 설치한 후에는 데이터베이스를 초기화하며 기존 데이터베이스를 그대로 사용하고 싶을 경우에는 파일을 업로드 해도 된다. 관리자 계정도 만들어준다.

```
$ source venv/bin/activate
$ pip install django
$ python manage.py migrate
$ python manage.py createsuperuser
```

### 웹 앱 설정

콘솔 화면 오른쪽 위의 메뉴의 [Web]을 눌러 이동하여 Add a new web app버튼을 누른다. Select a Python Web framework에서 Manual configuration (including virtualenvs)를 선택하고 Select a Python version은 위의 설정과 동일하게 3.7로 선택한다. 이후 Manual Configuration도 Next를 누르면 웹 앱 생성이 완료된다.

생성한 웹 앱이 업로드한 장고 어플리케이션 기준으로 동작하기 위한 설정을 해준다. [Code] 부분에서 WSGI configuration file의 링크를 클릭하여 파일 편집 창에 아래 코드를 입력 후 SAVE를 눌러 저장해준다. path의 id와 repo는 pythonanywhere의 아이디와 콘솔에서 만든 폴더이다.

```python
import os
import sys
path = "/home/id/repo"
if path not in sys.path:
    sys.path.append(path)
    
from django.contrib.staticfiles.handlers import StaticFileHandler
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTING_MODULE", "config.settings")
application = StaticFileHandler(get_wsgi_application())
```

[Virtualenv]에서 Enter path to a virtualenv, if desired을 클릭해 아래 경로를 입력한다. id와 repo는 위의 설정과 동일하다.

```
/home/id/repo/venv
```

모든 설정을 마친 후, 화면 상단의 [Reload id.pythonanywhere.com] 버튼을 누르고 그 위의 주소를 클릭한다. 메인 페이지에는 Not found 메시지가 뜨게 되고 uri에 repo/를 추가하면 프로젝트를 확인할 수 있다.

### 정적  파일 관리

WSGI 파일을 설정할 때 StaticFileHandler를 사용하였지만, 보통 서비스를 배포할 때는 잘 사용하지 않는 방법이다. 배포 상태일 때는 정적 파일은 STATIC_ROOT 변수에 설정한 경로에 모아두고 웹 서버가 파일을 찾도록 설정하는 것이 일반적이다. 이와 같이 설정하기 위해 settings.py 파일에서 STATIC_URL 밑에 아래 코드를 추가한다.

```python
STATIC_ROOT = os.path.join(BASE_DIR, 'static_files')
```

터미널에서 변경 사항을 깃헙에 업로드 한다.

```
$ git add -A
$ git commit -m"add s tatic_root"
$ git push -u
```

이제 pythonanywhere의 콘솔을 열고 repo 폴더로 이동한다. 소스 코드를 다운받은 후 설정을 사용하기 위해 정적 파일을 한곳에 모으는 명령어를 실행한다.

```
$ git pull
$ python manage.py collectstatic
```

collectstatic 명령어는 settings.py에 설정한 STATIC_ROOT의 경로에 정적 파일을 모은다. 설정을 적용하고 웹 서버가 찾을 수 있도록 하면 WSGI에서 StaticFileHandler를 사용하지 않아도 된다. 웹 앱 설정 화면으로 가서 [Static files] 부분에서 URL과 Directory를 각각 `/static/`, `/home/id/repo/static_files/`로 입력한다. id와 repo는 각각 pythonanywhere id와 깃헙 레포지토리명을 입력한다. 그 후 [Code] 부분의 WSGI configuration files을 눌러 아래와 같이 수정한다.

```python
import os
import sys
path = "/home/id/repo"
if path not in sys.path:
    sys.path.append(path)
    
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTING_MODULE", "config.settings")
application = StaticFileHandler(get_wsgi_application())
```

