---
layout: post
comments: true
title: 파이썬을 이용한 클린 코드를 위한 테스트 주도 개발 7장
category: Python
shortinfo: 스타일 추가하여 테스트하고 정적 파일 배포 준비하기
tags:
- python
- django
- TDD
---



## 7장

### 레이아웃과 스타일 기능 테스트

레이아웃 테스트를 위해 기능 테스트에 함수를 추가한다. 윈도우 크기를 고정한 후에 입력 상자가 가운데에 배치됐는지 확인하기 확인하기 위해 입력 상자 너비와 화면 너비를 반 나눈 값을 오차 10 픽셀 내에 비교한다.

```python
# functional_tests/tests.py

class NewVisitorTest(LiveServerTestCase):
  # 생략
  
  def test_layout_and_styling(self):
    # 에디스 메인 페이지 방문
    self.browser.get(self.live_server_url)
    self.browser.set_window_size(1024, 768)

    # 입력 상자 가운데 배치
    inputbox = self.browser.find_element_by_id('id_new_item')
    self.assertAlmostEqual(inputbox.location['x'] + inputbox.size['width'] / 2, 512, delta=10)
```

테스트 결과는 실패다. home.html을 수정하고 다시 테스트하면 통과된다.

```html
<p style="text-align: center">
   <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
</p>
```

신규 작업 페이지에서도 입력 상자의 배치를 테스트하도록 테스트 코드를 추가한다.

```python
# functional_tests/tests.py

    def test_layout_and_styling(self):
        # 에디스 메인 페이지 방문
        self.browser.get(self.live_server_url)
        self.browser.set_window_size(1024, 768)

        # 입력 상자 가운데 배치
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertAlmostEqual(inputbox.location['x'] + inputbox.size['width'] / 2, 512, delta=10)

        # 새로운 리스트를 시작하고 입력 상자가 가운데에 배치 된 것을 확인
        inputbox.send_keys('testing\n')
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertAlmostEqual(inputbox.location['x'] + inputbox.size['width'] / 2, 512, delta=10)
```

스타일링을 위해 부트스트랩을 받도록 한다. `superlists/static/bootstrap` 폴더에 부트스트랩 css, js 파일을 받도록 한다.

### django 템플릿 상속

`diff lists/templates/home.html lists/templates/list.html` 명령어를 실행하면 두 파일의 다른 점을 확인할 수 있는데 공통된 부분은 템플릿을 만들어서 상속하도록 한다. home.html을 복사하여 base.html 파일을 만들고 상속한 템플릿이 수정할 수 있는 부분인 block을 추가한다.

```html
{% raw %}
<html>
<head>
    <title>To-Do lists</title>
</head>
<body>
<h1>{% block header_text %}{% endblock %}</h1>
<form method="post" action="{% block form_action %}{% endblock %}">
    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
    {% csrf_token %}
</form>
{% block table %}
{% endblock %}
</body>
</html>
{% rawend %}
```

추가한 block을 사용할 수 있도록 home.html과 list.html을 수정하도록 한다.

```django
{# home.html #}
{% raw %}
{% extends 'base.html' %}

{% block header_text %}Your To-Do list{% endblock %}

{% block form_action %}/lists/new{% endblock %}
{% endraw %}
```

```django
{# list.html #}
{% raw %}
{% extends 'base.html' %}

{% block header_text %}Your To-do list{% endblock %}

{% block form_action %}/lists/{{ list.id }}/add_item{% endblock %}

{% block table %}
    <table id="id_list_table">
        {% for item in list.item_set.all %}
            <tr>
                <td>{{ forloop.counter }}: {{ item.text }}</td>
            </tr>
        {% endfor %}
    </table>
{% endblock %}
{% endraw %}
```

기능 테스트를 실행하면 여전히 `AssertionError: 80.33333587646484 != 512 within 10 delta` 에러가 뜬다.

### 부트스트랩

부트스트랩을 적용하기 위해 base.html을 수정한다. 책에서는 \<div>의 class에 `col-md` 가 아닌 `col-md-6 col-md-offset-3` 을 썼지만 가운데 정렬이 되지 않아 변경했다.

```django
{% raw %}
<html>
<head>
    <title>To-Do lists</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="css/bootstrap.min.css" rel="stylesheet" media="screen">
</head>
<body>
<div class="container">
    <div class="row">
        <div class="col-md">
            <div class="text-center">
                <h1>{% block header_text %}{% endblock %}</h1>
                <form method="post" action="{% block form_action %}{% endblock %}">
                    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
                    {% csrf_token %}
                </form>
            </div>
        </div>
    </div>
</div>
<div class="row">
    <div class="col-md-6 col-md-offset-3">
        {% block table %}
        {% endblock %}
    </div>
</div>
</body>
</html>
{% endraw %}
```

기능 테스트를 해도 여전히 같은 에러가 발생한다. 부트스트랩의 CSS가 로딩되지 않기 때문이다. 정적 파일(static file)을 다룰 때 고려할 사항이 두 가지가 있다.

> 1. URL이 정적 파일을 위한 것인지 view에서 제공하는 html을 위한 것인지
> 2. 사용자가 원할 때 어디서 정적 파일을 찾을 수 있는지

Django에서는 정적 파일 URL을 설정할 수 있다. superlists/settings.py에서 `STATIC_URL = '/static/'` 으로 설정된 것을 확인할 수 있다. 이제 html 파일에서도 정적 파일을 찾을 때 이 URL에서도 찾도록 수정해야 한다.

```django
<link href="/static/bootstrap/css/bootstrap.min.css" rel="stylesheet" media="screen">	
```

책에서는 settings.py에서 STATIC_URL을 확인하고 \<link>의 주소를 바꿔주는 것으로 정적 파일 로딩에 대한 내용을 끝냈지만 이것으로는 정적 파일을 불러올 수 없었다. 먼저 settings.py에 정적 파일 폴더에 대한 설정을 하나 더 추가했다.

```django
STATICFILES_DIRS = [os.path.join(BASE_DIR, "static")]
```

그리고 base.html에는 django template language를 이용해서 정적 파일을 불러오도록 했다. \<link>의 href에서도 static 을 사용하여 settings.py에 설정된 STATIC_URL 이후의 주소를 추가하도록 해야한다.

```django
{% raw %}
{% load static %}
<html>
<head>
    <title>To-Do lists</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="{% static 'bootstrap/css/bootstrap.css' %}" rel="stylesheet" media="screen">
</head>
{% endraw %}
```

정적 파일을 불러올 수 있음에도 불구하고 여전히 기능 테스트를 실패하게 되는데 LiveServerTestCase 클래스가 정적 파일을 자동으로 찾을 수 없기 때문이다. StaticLiveServerTestCase 클래스로 변경하고 테스트를 실행하면 통과하게 된다.

```python
from django.contrib.staticfiles.testing import StaticLiveServerTestCase

class NewVisitorTest(StaticLiveServerTestCase):
  # 생략
```

이제 부트스트랩을 적용할 수 있으니 base.html에 스타일을 좀 더 추가하도록 한다. header와 form을 강조하기 위해 `jumbotron` 을 적용하고 input도 `form-control input-group-lg` 를 추가해 더 크게 만들어준다.

```html
{% raw %}
<div class="container">
    <div class="row">
        <div class="col-md jumbotron">
            <div class="text-center">
                <h1>{% block header_text %}{% endblock %}</h1>
                <form method="post" action="{% block form_action %}{% endblock %}">
                    <input name="item_text" id="id_new_item" class="form-control input-group-lg" placeholder="작업 아이템 입력"/>
                    {% csrf_token %}
                </form>
            </div>
        </div>
    </div>
</div>
{% endraw %}
```

list.html의 테이블도 `table` 클래스를 추가해준다.

```html
<table id="id_list_table" class="table">
  {% for item in list.item_set.all %}
  <tr>
    <td>{{ forloop.counter }}: {{ item.text }}</td>
  </tr>
  {% endfor %}
</table>
```

input에는 id_new_item이라는 id도 있다. static/base.css 파일을 만들어 스타일을 추가하도록 한다. 그리고 이를 사용할 수 있도록 base.html에 추가한다.

```css
#id_new_item {
    margin-top: 2ex;
}
```

```html
{% raw %}
<link href="{% static 'base.css' %}" rel="stylesheet" media="screen">
{% endraw %}
```

### collectstatic

정적 파일을 한 곳에 모아 배포용으로 만들 때 `python manage.py collectstatic` 명령어를 사용할 수 있다.  정적 파일이 모아지는 폴더를 설정하기 위해서는 settings.py에 `STATIC_ROOT` 를 추가해야 한다. 앞서 먼저 추가한 `STATICFILES_DIRS` 는 STATIC_ROOT로 지정한 폴더를 포함할 수 없기 때문에 주의해야 한다. 아래와 같이 추가하고 collectstatic 명령어를 실행하면 superlists가 있는 폴더에 static 폴더가 생기고 모든 정적 파일이 모아진다.

```python
# superlists/settings.py

STATIC_ROOT = os.path.abspath(os.path.join(BASE_DIR, '../static'))
```

