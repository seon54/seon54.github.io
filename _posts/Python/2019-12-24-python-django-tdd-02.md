---
layout: post
comments: true
title: 파이썬을 이용한 클린 코드를 위한 테스트 주도 개발 2-3장
category: Python
shortinfo: unittest를 이용한 기능테스트와 django.test를 이용한 단위 테스트
tags:
- python
- django
- TDD
---



## 2장

functional_tests.py 내용 추가

```python
from selenium import webdriver

browser = webdriver.Firefox()

# 에디스는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 듣고 웹사이트를 확인하러 간다.
browser.get('http://localhost:8000')

# 웹 페이지 타이틀과 헤더가 'To-Do'를 표시하고 있다.
assert 'To-Do' in browser.title

# 작업 추가

# '공작깃털 사기'라고 텍스트 상자에 입력

# 엔터키를 치면 페이지가 갱신되고 작업 목록에
# '1: 공작깃털 사기' 아이템 추가

# 추가 아이템을 입력할 수 있는 여분의 텍스트 상자 존재
# 다시 '공작깃털을 이용해서 그물 만들기' 입력

# 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보임
# 입력한 목록을 저장하는 URL 생성

# 해당 URL에 접속하면 작업 목록 확인 가능

browser.quit()
```

`python manage.py runserver` 명령어로 서버를 실행하고 또 다른 커맨드라인에서 `python functional_test.py` 명령어로 파일을 실행한다. 아래와 같은 에러가 발생한다.

> Traceback (most recent call last):
>   File "functional_tests.py", line 12, in <module>
>     assert 'To-Do' in browser.title
> AssertionError

AssertionError가 발생했지만 에러가 발생한 원인을 알 수 없으므로 아래와 같이 수정하면 수정된 에러메세지를 확인할 수 있다.

```python
assert 'To-Do' in browser.title, 'Browser title was ' + browser.title
```

> Traceback (most recent call last):
>   File "functional_tests.py", line 12, in <module>
>     assert 'To-Do' in browser.title, 'Browser title was ' + browser.title
> AssertionError: Browser title was Django: the Web framework for perfectionists with deadlines.



### unittest

파이썬의 기본 라이브러리인 unittest를 이용하여 코드를 수정하도록 한다. **unittest.TestCase** 를 상속하여 NewVisitorTest 클래스를 만든다. 클래스 안에 test로 시작하는 메서드가 있는데 test로 시작하는 메서드는 모드 테스트 메서드가 된다. setUp 메서드는 테스트 시작 전에, tearDown은 테스트 후에 실행된다. tearDown은 테스트에 에러가 발생해도 실행된다. 마지막에 `if __name__ == '__main__'` 은 파일이 다른 스크립트에 임포트된 것이 아닌 커맨드라인을 통해 실행됐다는 것을 확인한다.

```python
# functional_test.py

import os
import unittest
from selenium import webdriver


class NewVisitorTest(unittest.TestCase):

    def setUp(self) -> None:
        path = os.path.join(os.path.dirname(os.getcwd()), 'geckodriver')
        self.browser = webdriver.Firefox(executable_path=path)

    def tearDown(self) -> None:
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
        # 에디스는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 듣고 웹사이트를 확인하러 간다.
        self.browser.get('http://localhost:8000')

        # 웹 페이지 타이틀과 헤더가 'To-Do' 표시
        self.assertIn('To-Do', self.browser.title)
        self.fail('Finish the test!')

        # 작업 추가

        # '공작깃털 사기'라고 텍스트 상자에 입력

        # 엔터키를 치면 페이지가 갱신되고 작업 목록에
        # '1: 공작깃털 사기' 아이템 추가

        # 추가 아이템을 입력할 수 있는 여분의 텍스트 상자 존재
        # 다시 '공작깃털을 이용해서 그물 만들기' 입력

        # 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보임
        # 입력한 목록을 저장하는 URL 생성

        # 해당 URL에 접속하면 작업 목록 확인 가능


if __name__ == '__main__':
    unittest.main(warnings='ignore')
```

실행 후 파이어폭스 창이 닫히고 좀 더 형태가 갖춰진 에러 메세지를 볼 수 있다.


>  F
>
> FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
>
> Traceback (most recent call last):
> File "functional_tests.py", line 20, in test_can_start_a_list_and_retrieve_it_later
>  self.assertIn('To-Do', self.browser.title)
> AssertionError: 'To-Do' not found in 'Django: the Web framework for perfectionists with deadlines.'
>
> ----------------------------------------------------------------------
> Ran 1 test in 4.462s
>
> FAILED (failures=1)



### 암묵적 대기

setUp에 implicitly_wait를 추가한다. implicitly_wait에 지정한 시간(초 단위)만큼 동작을 대기 상태로 둘 수 있다.

```python
import os
import unittest
from selenium import webdriver


class NewVisitorTest(unittest.TestCase):

    def setUp(self) -> None:
        path = os.path.join(os.path.dirname(os.getcwd()), 'geckodriver')
        self.browser = webdriver.Firefox(executable_path=path)
        self.browser.implicitly_wait(3)

    def tearDown(self) -> None:
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
        # 에디스는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 듣고 웹사이트를 확인하러 간다.
        self.browser.get('http://localhost:8000')

        # 웹 페이지 타이틀과 헤더가 'To-Do' 표시
        self.assertIn('To-Do', self.browser.title)
        self.fail('Finish the test!')

        # 작업 추가

        # '공작깃털 사기'라고 텍스트 상자에 입력

        # 엔터키를 치면 페이지가 갱신되고 작업 목록에
        # '1: 공작깃털 사기' 아이템 추가

        # 추가 아이템을 입력할 수 있는 여분의 텍스트 상자 존재
        # 다시 '공작깃털을 이용해서 그물 만들기' 입력

        # 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보임
        # 입력한 목록을 저장하는 URL 생성

        # 해당 URL에 접속하면 작업 목록 확인 가능


if __name__ == '__main__':
    unittest.main(warnings='ignore')

```



## 3장

`python manage.py startapp lists` 명령어로 lists 앱을 만든다. superlists 폴더 안에 lists, superlists 폴더를 확인할 수 있다. 

단위테스트와 기능테스트는 각각 사용자 관점에서 앱 외부를 테스트 하는 것과 개발자 관점에서 내부를 테스트 하는 것으로 나눌 수 있다.



### Django에서 단위 테스트

lists/tests.py를 보면 장고가 제공하는 TestCase를 확인할 수 있다. 이는 unittest.TestCase를 확장한 것으로 django 특화 기능이 추가되어 있다. tests.py에 실패 테스트를 만들고 `python manage.py test` 명령어로 확인하도록 한다.

```python
from django.test import TestCase


class SmokeTest(TestCase):

    def test_bad_maths(self):
        self.assertEqual(1 + 1, 3)
```

> Creating test database for alias 'default'...
> System check identified no issues (0 silenced).
>
> F
>
> FAIL: test_bad_maths (lists.tests.SmokeTest)
>
> Traceback (most recent call last):
>   File "/Users/jlk/Desktop/github/django-tdd/superlists/lists/tests.py", line 7, in test_bad_maths
>     self.assertEqual(1 + 1, 3)
> AssertionError: 2 != 3
>
> ----------------------------------------------------------------------
> Ran 1 test in 0.001s
>
> FAILED (failures=1)
> Destroying test database for alias 'default'...



### Django의 MVC

Django로 대체로 MVC의 패턴을 따르며 처리 흐름은 다음과 같다.

1. 특정 URL에 대한 HTTP request를 받음
2. django는 특정 규칙을 이용해 해당 request에 어떤 view gkatnfmf tlfgodgkfwl rufwjd
3. view의 function이 request를 처리하고 HTTP response로 반환

여기서 테스트 해야할 것은 아래 두 가지이다.

- URL의 사이트 루트(/)를 해석해서 특정 view function에 매칭할 수 있는가
- 이 view function이 특정 HTML을 반환하게 해서 기능 테스트를 통과할 수 있는가

먼저 첫번째를 테스트 하기 위해 코드를 수정한다.

```python
from django.test import TestCase
from django.urls import resolve
from .views import home_page


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)
```

위의 코드를 실행하면 `ImportError` 가 발생한다. home_page라는 view를 아직 작성하지 않았기 때문이다.



### 애플리케이션 코드 작성

lists/views.py에 가서 코드를 작성한다. 어처구니 없는 코드지만 작성하고 나서 `python manage.py test` 명령어를 실행한 후 에러 메세지를 확인한다. Traceback을 잘 살펴보고 에러를 파악하도록 해야한다. '/' 매핑을 찾지 못해 404 에러가 발생한 것을 알 수 있다.

```python
from django.shortcuts import render


home_page=None
```

> Creating test database for alias 'default'...
> System check identified no issues (0 silenced).
>
> E
>
> ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest)
>
> Traceback (most recent call last):
>   File "/Users/jlk/Desktop/github/django-tdd/superlists/lists/tests.py", line 9, in test_root_url_resolves_to_home_page_view
>     found = resolve('/')
>   File "/Users/jlk/anaconda3/envs/tdd/lib/python3.6/site-packages/django/urls/base.py", line 24, in resolve
>     return get_resolver(urlconf).resolve(path)
>   File "/Users/jlk/anaconda3/envs/tdd/lib/python3.6/site-packages/django/urls/resolvers.py", line 567, in resolve
>     raise Resolver404({'tried': tried, 'path': new_path})
> django.urls.exceptions.Resolver404: {'tried': [[<URLResolver <URLPattern list> (admin:admin) 'admin/'>]], 'path': ''}
>
> ----------------------------------------------------------------------
> Ran 1 test in 0.001s
>
> FAILED (errors=1)
> Destroying test database for alias 'default'...



### urls.py

django는 urls.py에서 URL을 view 함수에 매핑하는 것을 정의한다. superlists/superlists 폴더의 urls.py 파일을 보면 아래와 같다. 책은 django 이전 버전이라 path 대신 url을 사용하고 있다. 

```python
from django.contrib import admin
from django.urls import path

urlpatterns = [    
    path('admin/', admin.site.urls),
]

```

path를 하나 더 추가하여 테스트하도록 한다. 책에서는 superlists.views를 import 할 수 없다는 ImportError가 발생하였지만 TypeError가 발생하였다. 버전의 차이인 것 같다.

```python
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('', 'superlists.views.home', name='home'),
    path('admin/', admin.site.urls),
]

```

> Creating test database for alias 'default'...
> Destroying test database for alias 'default'...
> Traceback (most recent call last):
>   File "manage.py", line 21, in <module>
>     main()
>  ......
>   File "/Users/jlk/Desktop/github/django-tdd/superlists/superlists/urls.py", line 5, in <module>
>     path('', 'superlists.views.home', name='home'),
>   File "/Users/jlk/anaconda3/envs/tdd/lib/python3.6/site-packages/django/urls/conf.py", line 73, in _path
>     raise TypeError('view must be a callable or a list/tuple in the case of include().')
> TypeError: view must be a callable or a list/tuple in the case of include().

path를 아래와 같이 수정하고 다시 테스트 명령어를 실행한다. 에러메세지는 위와 같고 책의 에러메세지와 완전히 같지는 않지만 view가 callable 하지 않다는 내용은 비슷하다.

```python
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('', 'lists.views.home', name='home'),
    path('admin/', admin.site.urls),
]
```

다시 lists/views.py로 돌아가서 수정한 후 테스트 해보았지만 책에서 테스트를 패스한 것과 달리 위에서 나온 에러가 또 발생하여 urls.py를 수정하였다.

```python
# lists/views.py

from django.shortcuts import render


def home_page():
    pass
  
# superlists/urls.py

from django.contrib import admin
from django.urls import path
from lists import views


urlpatterns = [
    path('', views.home_page, name='home'),
    path('admin/', admin.site.urls),
]
```

수정한 후에야 책과 같은 결과가 나왔다.

> Creating test database for alias 'default'...
> System check identified no issues (0 silenced).
>  .
> ----------------------------------------------------------------------
> Ran 1 test in 0.000s
>
> OK
> Destroying test database for alias 'default'...



### view를 위한 단위 테스트

view를 위한 단위 테스트를 할 때는 HTML 형식의 실제 응답을 반환하는 함수를 작성해야 한다. lists/tests.py를 아래와 같이 수정한다.

HttpRequest 객체를 생성하고 home_page view에 전달한 후 response의 content와 title을 확인한다.

```python
from django.test import TestCase
from django.http import HttpRequest
from django.urls import resolve

from .views import home_page


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)

    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        self.assertTrue(response.content.startswith(b'<html>'))
        self.assertIn(b'<title>To-Do lists</title>', response.content)
        self.assertTrue(response.content.endswith(b'</html>'))
```
테스트를 하면 home_page()에는 전달인자를 갖지 않지만 1개의 전달인자가 들어있다는 에러를 확인할 수 있다.

> Creating test database for alias 'default'...
> System check identified no issues (0 silenced).
> 
>E.
> 
>ERROR: test_home_page_returns_correct_html (lists.tests.HomePageTest)
> 
>Traceback (most recent call last):
> File "/Users/jlk/Desktop/github/django-tdd/superlists/lists/tests.py", line 16, in test_home_page_returns_correct_html
>    response = home_page(request)
>    TypeError: home_page() takes 0 positional arguments but 1 was given
> 
>----------------------------------------------------------------------
> Ran 2 tests in 0.001s
> 
>FAILED (errors=1)
> Destroying test database for alias 'default'...

TDD 단위 테스트는 실패 테스트를 수정하기 위해 최소한의 코드를 수정하도록 한다. views.py를 수정하고 테스트 하는 과정을 반복한다.

```python
def home_page(request):
  pass
```

> self.assertTrue(response.content.startswith(b'<html>'))
> AttributeError: 'NoneType' object has no attribute 'content'

다시 views.py를 수정하여 HttpResponse()를 추가하고 테스트 한다.

```python
from django.http import HttpResponse


def home_page(request):
    return HttpResponse()
```

> Traceback (most recent call last):
>   File "/Users/jlk/Desktop/github/django-tdd/superlists/lists/tests.py", line 17, in test_home_page_returns_correct_html
>     self.assertTrue(response.content.startswith(b'<html>'))
> AssertionError: False is not true

```python
from django.http import HttpResponse


def home_page(request):
    return HttpResponse('<html>')
```

> Traceback (most recent call last):
>   File "/Users/jlk/Desktop/github/django-tdd/superlists/lists/tests.py", line 18, in test_home_page_returns_correct_html
>     self.assertIn(b'<title>To-Do lists</title>', response.content)
> AssertionError: b'<title>To-Do lists</title>' not found in b'<html>'

```python
from django.http import HttpResponse


def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title>')
```

> Traceback (most recent call last):
>   File "/Users/jlk/Desktop/github/django-tdd/superlists/lists/tests.py", line 19, in test_home_page_returns_correct_html
>     self.assertTrue(response.content.endswith(b'</html>'))
> AssertionError: False is not true

```python
from django.http import HttpResponse


def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title></html>')
```

> Ran 2 tests in 0.002s
>
> OK

드디어 단위 테스트 수정을 마쳤다. 개발 서버를 킨 상태에서 다시 기능 테스트를 해보도록 한다. `self.fail('Finish the test!')`가 에러 메세지로 나왔다. 단지 작업 완료 메세지를 위해 self.fail에 메세지를 넣은 것이어서 테스트는 성공이다.

>F
>
> FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
>
> Traceback (most recent call last):
>   File "functional_tests.py", line 22, in test_can_start_a_list_and_retrieve_it_later
>     self.fail('Finish the test!')
> AssertionError: Finish the test!
>
> ----------------------------------------------------------------------
> Ran 1 test in 4.247s
>
> FAILED (failures=1)
