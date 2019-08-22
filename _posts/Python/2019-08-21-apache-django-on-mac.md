---
layout: post
comments: true
title: Mac에서 apache & django 연동하기
category: python
shortinfo: Mac에서 apache httpd와 mod_wsgi 설치하고 django 연동하기
tags:
- django
- apache
---



##시행착오

`whereis httpd` 명령어로 기존에 설치되어 있는 것을 확인하고 `/usr/sbin/httpd -v`을 통해 버전도 확인했다.

```shell
$ whereis httpd
  /usr/sbin/httpd
$ /usr/sbin/httpd -v
  Server version: Apache/2.4.34 (Unix)
  Server built:   Feb 22 2019 20:20:11
```

mac에서 mod_wsgi 설치를 검색해서 [Installation On Mac OS](https://modwsgi.readthedocs.io/en/develop/user-guides/installation-on-macosx.html) 가이드를 따라 해보았다. xcode는 이미 설치되어 있어서 아래와 같은 결과가 나왔다.

```shell
$ xcode-select --install
  xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```

그 다음 과정은 시간이 좀 걸렸다. mac을 처음 사용하고 shell 또한 익숙치 않았기 때문에 마냥 `./configure` 명령어만 치고 no such file or directory: ./configure라는 결과만 보게 되었다. 검색을 하다가 mod_wsgi [github](https://github.com/GrahamDumpleton/mod_wsgi/releases)에서 mod_wsgi 4.6.5 버전을 받고 압축 해제한 파일에서 configure 파일을 확인할 수 있었다.압축 해제한 mod_wsgi 폴더에서 명령어를 실행하면 공식 홈페이지와 비슷한 결과가 나온다.

```shell
$ ./configure
```

하지만 자세히 보면 checking for apxs에서 no라는 결과가 나와 결국 설치에 실패하고 말았다.

```shell
checking for apxs2... no
checking for apxs... no
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables...
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking for prctl... no
checking Apache version... ./configure: line 2765: apxs: command not found
./configure: line 2765: apxs: command not found
./configure: line 2766: apxs: command not found
./configure: line 2769: /: is a directory
```

mac에 기존에 설치되어 있는 httpd를 찾지 못하는 것 같아 설치를 해주었다.

```shell
$ brew install httpd
```

설치 과정에서 나중에 설정할 때 필요한 파일인 httpd.conf의 위치(/usr/local/etc/httpd/httpd.conf)를 확인할 수 있었다. 

```shell
The default ports have been set in /usr/local/etc/httpd/httpd.conf to 8080 and in
/usr/local/etc/httpd/extra/httpd-ssl.conf to 8443 so that httpd can run without sudo.
```

httpd를 설치한 후 다시 `./configure` 명령어를 시도하면 마지막에 Makefile이 만들어졌다는 내용을 확인할 수 있다.

```shell
$ ./configure
	checking for apxs2... no
  checking for apxs... /usr/local/bin/apxs
  ...
  configure: creating ./config.status
  config.status: creating Makefile
```

 그 후에 차례대로 `make`, `sudo make instsall` 명령어를 실행하였으나 홈페이지에 나온 것처럼 `/usr/libexec/apache2`에 mod_wsgi.so 파일이 설치된 것이 아니라 `/usr/local/Cellar/httpd/2.4.41/lib/httpd/modules` 경로에 설치되었다.

```shell
$ make
$ sudo make install
```

`mod_wsgi-express start-server` 명령어도 실행되지 않고 `sudo make install LIBEXECDIR=/usr/libexec/apache2`로 설치 위치도 설정해 보았으나 에러가 발생하였다. 공식홈페이지에서 나온 방법을 포기하고 두번째 방법인 `pip install mod-wsgi` 명령어로 설치하는 것을 다시 시도해보았다. 설치 후, mod_wsgi 서버 실행도 localhost:8000에서 확인할 수 있었다.

```shell
$ pip install mod-wsgi
$ mod_wsgi-express start-server
```



#### 중간정리

1. apache httpd 설치
   - `brew install httpd`
2. apache 서버 실행
   - `sudo apachectl start` 실행 후, http://localhost에서 **It works!** 확인
3. mod_wsgi 설치
   - `pip install mod-wsgi` 
4. mod_wsgi 서버 실행
   - `mod_wsgi-express start-server` 실행 후, http://localhost:8000에서 확인



## conf 파일 설정

여러 가지 시도를 통해 httpd와 mod_wsgi를 설치하고 이제 httpd.conf와 httpd-vhosts.conf 파일을 수정해야 한다. 가상환경을 이용하기 때문에 httpd-vhosts.conf 파일도 수정한다. httpd를 설치할 때 터미널에서 확인했듯이 `/usr/local/etc/httpd` 경로에 httpd.conf 파일을 확인할 수 있다. `vi /user/local/etc/httpd/httpd.conf`로 수정하거나 visual studio code가 있다면 `code /user/local/etc/httpd/httpd.conf`로 수정할 수 있다. visual studio code가 더 편해서 두번째 방법을 사용하였다.



##### httpd.conf

먼저 httpd.conf 파일부터 수정한다. `Listen`에 포트가 8080으로 되어 있는데 80으로 수정하고 주석 처리가 되어 있는 가상환경 관련 설정을 해제해준다. 그리고 `ServerName`를 각자의 호스트명으로 지정해준다. 

```shell
Listen 80
ServerName localhost
LoadModule vhost_alias_module lib/httpd/modules/mod_vhost_alias.so
Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

 터미널에서 `mod_wsgi-express module-config` 를 입력하면 mod_wsgi 모듈에 대한 설정이 나온다. 이것 또한 추가하도록 한다. 결과는 아래와 비슷하며 환경에 따라 다르게 나온다. 

```shell
LoadFile "/Users/myuser/anaconda3/envs/newenv/lib/libpython3.6m.dylib"
LoadModule wsgi_module "/Users/myuser/anaconda3/envs/newenv/lib/python3.6/site-packages/mod_wsgi/server/mod_wsgi-py36.cpython-36m-darwin.so"
WSGIPythonHome "/Users/myuser/anaconda3/envs/newenv"
```



##### httpd-vhosts.conf

httpd-vhosts.conf에서는 가상환경에 대한 설정을 할 수 있다.  `ErrorLog`와 `CustomLog`에서 error log와 access log를 원하는 경로로 설정할 수 있다. [장고 공식 홈페이지](https://docs.djangoproject.com/en/2.2/howto/deployment/wsgi/modwsgi/#using-mod-wsgi-daemon-mode)에서는 윈도우 플랫폼이 아닌 곳에서 mod_wsgi를 실행할 때 daemon mode를 추천하며 `WSGIDaemonProcess`, `WSGIProcessGroup`, `WSGIScriptAlias` 에서 관련 설정을 해줄 수 있다. 먼저 `WSGIDaemonProcess` 와  `WSGIProcessGroup` 의 이름 그리고`WSGIScriptAlias` 의 process-group은 같은 값을 넣어준다. python-home에는 위의 `WSGIPythonHome`의 값과 같고 python-path에는 파이썬 라이브러리 경로를 적어준다. 

`WSGIScriptAlias` 에는 django의 wsgi.py 파일이 위치한 경로를 적어준다. 아래의 <Directory>에도 설정해준다. 아래의 두 `Alias` 에는 각각 static 폴더와 media 폴더에 대한 설정을 해주고 있다. settings.py에서 static 파일과 media 폴더에 대한 설정이 필요하며 `python manage.py collectstatic` 명령어를 실행해주면 static 폴더가 생기며 관련 파일이 모이게 된다.

```python
# settings.py

# STATIC
STATIC_URL = '/static/'
STATICFILE_DIRS = (
    os.path.join(BASE_DIR, 'static')
)
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

# MEDIA
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

```shell
<VirtualHost *:80>   
    ErrorLog "/Users/myuser/desktop/log/error.log" 
    CustomLog "/Users/myuser/desktop/log/access.log"  common
      
    WSGIDaemonProcess app1 python-home=/Users/myuser/anaconda3/envs/newenv python-path=/users/myuser/anaconda3/envs/newenv/lib/python3.6/site-packages
    WSGIProcessGroup app1

    WSGIScriptAlias /  "/Users/myuser/desktop/project1/django/server/wsgi.py" process-group=app1
    <Directory "/Users/myuser/desktop/project1/django/server">
        <Files wsgi.py>
            AllowOverride All
            Require all granted
        </Files>
    </Directory>

    Alias /static/ "/Users/myuser/desktop/project1/django/static/"
    <Directory "/Users/myuser/desktop/project1/django/static">
        AllowOverride All
        Require all granted
    </Directory>  
    
    Alias /media/ "Users/myuser/desktop/project1/django/media/"
    <Directory "Users/myuser/desktop/project1/django/media">
        AllowOverride All
        Require all granted
    </Directory> 
    
</VirtualHost>
```



## 실행

설정을 모두 마치고 httpd 서버를 실행할 차례다. 설치 후 테스트를 위해 실행했던 명령어를 다시 실행한다.

```shell
$ sudo apachectl start
```

ServerName을 localhost로 하고 포트를 80으로 설정했으므로 http://localhost에서 나의 장고 프로젝트를 확인할 수 있다. 



## Multiple Vhosts

###### *19.08.22 추가*

처음부터 여러 개의 장고 프로젝트를 아파치 서버로 올리는 것이 목적이었기 때문에 VirtualHost를 수정해주었다. 먼저 `WSGIScriptAlias` 이름을 수정해주어야 한다. 위에서 설정한 `SeverName` 에서 설정한 localhost에서 연결되는 부분이므로 각 app의 이름으로 구분하도록 하였다. 위에서는 `/` 로 설정한 것을 `/app1`,  `/app2` 로 수정했다. 각각 localhost/app1, localhost/app2의 식으로 적용된다. 그리고 static과 media도 수정해야 한다. 처음 설정한 것처럼 `/static/` 과 `/media/`  alias 이름이 같게 되면 첫번째것만 인식하는 것 같았다. 그래서 기존 장고의 settings을 변경하고 각 app의 alias도 수정했다.

```shell
<VirtualHost *:80>    
    ErrorLog "/Users/myuser/desktop/log/error.log" 
    CustomLog "/Users/myuser/desktop/log/access.log"  common
      
    # app1
    WSGIDaemonProcess app1 python-home=/Users/myuser/anaconda3/envs/newenv python-path=/users/myuser/anaconda3/envs/newenv/lib/python3.6/site-packages
    WSGIProcessGroup app1

    WSGIScriptAlias /app1  "/Users/myuser/desktop/project1/django/server/wsgi.py" process-group=app1
    <Directory "/Users/myuser/desktop/project1/django/server">
        <Files wsgi.py>
            AllowOverride All
            Require all granted
        </Files>
    </Directory>

    Alias /app1/static/ "/Users/myuser/desktop/project1/django/static/"
    <Directory "/Users/myuser/desktop/project1/django/static">
        AllowOverride All
        Require all granted
    </Directory>  
    
    Alias /app1/media/ "/Users/myuser/desktop/project1/django/media/"
    <Directory "/Users/myuser/desktop/project1/django/media">
        AllowOverride All
        Require all granted
    </Directory> 
    
    # app2
    WSGIDaemonProcess app2 python-home=/Users/myuser/anaconda3/envs/newenv python-path=/users/myuser/anaconda3/envs/newenv/lib/python3.6/site-packages
    WSGIProcessGroup app2

    WSGIScriptAlias /app2  "/Users/myuser/desktop/project2/django/server/wsgi.py" process-group=app2
    <Directory "/Users/myuser/desktop/project2/django/server">
        <Files wsgi.py>
            AllowOverride All
            Require all granted
        </Files>
    </Directory>

    Alias /app2/static/ "/Users/myuser/desktop/project2/django/static/"
    <Directory "/Users/myuser/desktop/project2/django/static">
        AllowOverride All
        Require all granted
    </Directory>  
    
    Alias /app2/media/ "/Users/myuser/desktop/project2/django/media/"
    <Directory "/Users/myuser/desktop/project2/django/media">
        AllowOverride All
        Require all granted
    </Directory> 
    
</VirtualHost>
```

```python
# project1/django/settings.py
...
# MEDIA_URL = '/media/'
MEDIA_URL = '/app1/media/'


# project2/django/settings.py
...
# MEDIA_URL = '/media/'
MEDIA_URL = '/app2/media/'
```

Httpd_vhosts.conf 파일과 장고 settings.py를 수정한 후 http://localhost/app1과 http://localhost/app2에서 각 프로젝트를 확인할 수 있다. 



### Commands

자주 사용했던 명령어

- `sudo apachectl start` : httpd 서버 실행
- `sudo apachectl restart` : httpd 서버 재실행
- `sudo apachectl stop` : httpd 서버 멈추기
- `apachectl configtest` : conf 파일 문법 확인
- `ps aux | grep httpd` : https pid와 실행 확인
- `whereis apachectl` : apachectl 경로 확인
- `brew list` : brew install 로 설치 목록 확인
- `:w !sudo tee % > /dev/null` : vi로 readonly 파일을 수정 후 저장



### Path

기존에 설치되어 있는 apache와 brew 명령어로 설치한 apache의 경로

##### apache(기존)

- 설정: /etc/apache2/httpd.conf
- 가상 호스트: /etc/apache2/extra/httpd-vhosts.conf
- 로그: /var/log/apache2
- 모듈: /usr/libexec/apache2
- Hosts: /etc/hosts

##### apache(설치)

- 설정 : /usr/local/etc/httpd/httpd.conf 
- 가상 호스트: /usr/local/etc/httpd/extra/httpd-vhosts.conf
- 로그: /usr/local/var/log/httpd
- 모듈: /usr/local/lib/httpd/modules



### Errors

apache와 django를 연동하며 겪은 에러들이다. 아파치 서버를 올린 후 에러가 났을 때는 `Debug=True` 로 설정한 후 로그를 확인하면 원인을 찾기 더 편하다. 

```shell
AH00557: httpd: apr_sockaddr_info_get() failed for myhost
```

​	➡️ `sudo vi /etc/hosts` 입력 후, 해당 호스트명 추가 `127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4 [호스트명]`



```shell
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using myhost. Set the 'ServerName' directive globally to suppress this message
```

​	➡️ /usr/local/etc/httpd/httpd.conf의 `ServerName` localhost 추가



```SHELL
httpd (mod_wsgi-express) : Syntax error on line 3 of /var/tmp/mod_wsgi-localhost:8000:501/httpd.conf: Cannot load /usr/local/lib/httpd/modules/mod_version.so into server: dlopen(/usr/local/lib/httpd/modules/mod_version.so, 10): image not found
```

​	➡️ 기존 mac에 설치된 httpd를 사용하려고 하니 mod_wsgi 모듈을 찾지 못하여 `brew install httpd` 로 설치 후 `mod_wsgi-express start-server` 실행 OK