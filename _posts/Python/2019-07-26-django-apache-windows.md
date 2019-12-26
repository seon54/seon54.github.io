---
layout: post
comments: true
title: Windows에서 Django와 Apache 연동하기
category: python
shortinfo: Windows에서 Django와 Apache 연동하는 방법을 알아봅니다.
tags:
- Apache
- Django
---


## 설치
파이썬이 설치되어 있다고 가정하고 아래의 파일을 설치한다.
1. Apache - [다운로드](https://www.apachelounge.com/download/)
2. Visual Studio Build Tools - [다운로드](https://visualstudio.microsoft.com/ko/downloads/)
3. Virtualenv
4. mod_wsgi

가상환경에서 진행하기 위해 virtualenv를 설치한다. 파이썬이 설치되어 있다면 `pip virtualenv` 명령어로 설치할 수 있다. `C:/Users/python`  폴더에 가상환경을 만든다고 가정하고 `virtualenv newEnv` 명령어로 newEnv라는 새로운 가상환경을 만든다. 가상환경을 실행할 때는 `call newEnv/Scripts/activate`나 `C:/Users/python/newEnv/Scripts` 경로에 들어가서 cmd 창에서 `activate`라고 치면 가상환경이 실행된다. 가상환경을 실행한 후 `pip install django` 명령어로 django와 그 외 필요한 패키지들을 설치하도록 한다. 마지막으로 mod_wsgi를 사용하기 위해 `pip install mod_wsgi` 명령어로 설치해준다.

```shell
$ C:/Users/python>pip virtualenv
$ C:/Users/python>virtualenv newEnv
$ C:/Users/python>call newEnv/Scripts/activate
$ (newEnv) C:/Users/python>pip install django
$ (newEnv) C:/Users/python>pip install mod_wsgi
```



## wsgi.py

settings.py 파일이 있는 폴더에는 wsgi.py 파일이 존재한다. 여기서는 **wsgi_win.py** 파일명으로 새로운 wsgi 파일을 생성한다. 아래의 `C:/Users/python/newEnv`는 각자의 가상환경이 설치된 경로에 맞추고 `myapp` 또한 각자의 어플리케이션 이름으로 바꾸어주면 된다.

```python
# wsgi_win.py

activate_this = 'C:/Users/pytyon/newEnv/Scripts/activate_this.py'

exec(open(activate_this).read(),dict(__file__=activate_this))

import os
import sys
import site

# 가상환경의 패키지 추가
site.addsitedir('C:/Users/pytyon/newEnv/Scripts/Lib/site-packages')

# PYTHONPATH에 application directory 추가
path = os.path.abspath(__file__+'/../..')
if path not in sys.path:
    sys.path.append(path)

os.environ['DJANGO_SETTINGS_MODULE'] = 'myapp.settings'
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myapp.settings")

from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```



## settings.py

settings.py에서 static 파일과 media 파일에 대한 설정이 필요하다. 아래와 같이 설정한 후 `python manage.py collectstatic` 명령어를 실행한다. 명령어를 실행한 후에는 모든 static 파일들이 static 폴더에 저장되고 후에 저장되는 media 파일 또한 media 폴더에 저장된다. 그리고 urls.py에 `static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)`도 추가해준다.

```python
# myapp/settings.py
STATIC_URL = '/static/'
STATICFILE_DIRS = (
    os.path.join(BASE_DIR, 'static')
)
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# myapp/urls.py
from django.contrib import admin
from django.conf.urls import url
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
	...   	
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```



## Apache

압축 파일을 해제한 후, Apache24 폴더를 C 드라이브로 옮기고  `C:/Apache24/conf` 폴더에 있는 httpd.conf 파일을 나의 환경에 맞게 수정해 주어야 한다. 먼저 ServerName을 검색하여 www.example.com:80으로 되어 있는 주소를 각자 도메인과 포트에 맞게 수정한다.

```powershell
ServerName www.yourdomain.com:80
```

위의 가상환경이 실행된 cmd 창에서 `mod_wsgi-express module-config` 명령어를 실행했을 때 나오는 결과들을 httpd.conf 파일에 그대로 입력한다. 내용은 각자의 환경 설정마다 다르다.

```powershell
LoadFile "c:/users/myuser/appdata/local/programs/python/python37/python37.dll"
LoadModule wsgi_module "c:/users/newenv/lib/site-packages/mod_wsgi/server/mod_wsgi.cp37-win_amd64.pyd"
WSGIPythonHome "c:/users/newenv"
```

그 외에 추가로 설정해야 하는 것들은 아래와 같다.

```python
WSGIScriptAlias / "c:/users/python/myapp/myapp/wsgi_win.py"
WSGIPythonPath "c:/users/python/myapp/myapp"

<Directory "c:/users/python/myapp/myapp">
<Files wsgi_win.py>
Require all granted
</Files>
</Directory>

Alias /static/ c:/users/python/myapp/static/
<Directory c:/users/python/myapp/static/>
Require all granted
</Directory>

Alias /media/ c:/users/python/myapp/media/
<Directory c:/users/python/myapp/media/>
Require all granted
</Directory>
```

아파치 서버를 실행할 때 CORS 에러가 나서 아래의 설정을 추가해주었다. error.log에서 print()에 출력되는 내용을 보기 위해 `WSGIRestrictStdout Off`도 추가해준다.

```python
<Directory c:/users/python/myapp>
    Header set Access-Control-Allow-Origin "*"
</Directory>

WSGIPassAuthorization On
WSGIRestrictStdout Off
```

  `C:/Apache24/conf/extra` 폴더에 htpd-vhosts.conf 파일을 열어 가상환경 설정을 추가한다. 

```python
<VirtualHost *:80>
    ServerName localhost 
    WSGIPassAuthorization On
    Header set Access-Control-Allow-Origin "*"    
    ErrorLog "logs/error.log" common
    CustomLog "logs/access.log" common 
    WSGIScriptAlias /  "c:/users/python/myapp/myapp/wsgi_win.py"
    <Directory "c:/users/python/myapp/myapp">
        <Files wsgi_windows.py>
            Require all granted
        </Files>
    </Directory>

    Alias /static "c:/users/python/myapp/myapp/static"
    <Directory "c:/users/python/myapp/myapp/static">
        Require all granted
    </Directory>  
</VirtualHost>
```

