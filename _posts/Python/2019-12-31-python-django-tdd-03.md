---
layout: post
comments: true
title: 파이썬을 이용한 클린 코드를 위한 테스트 주도 개발 4-5장
category: Python
shortinfo: django template을 이용한 POST 요청 테스트
tags:
- python
- django
- TDD
---



## 4장

### selenium을 이용한 사용자 반응 테스트

이전 장에 이어 functional_test.py에 테스트 할 내용을 아래와 같이 추가한다. selenium이 제공하는 다양헨 메서드로 tag 또는 id로 찾고자 하는 요소를 명확히 할 수 있다.

```python
import unittest
from selenium.webdriver.common.keys import Keys
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
        header_text = self.browser.find_element_by_tag_name('h1').text
        self.assertIn('To-Do', header_text)

        # 작업 추가
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertEqual(inputbox.get_attribute('placeholder'), '작업 아이템 입력')

        # '공작깃털 사기'라고 텍스트 상자에 입력
        inputbox.send_keys('공작깃털 사')

        # 엔터키를 치면 페이지가 갱신되고 작업 목록에
        # '1: 공작깃털 사기' 아이템 추가
        inputbox.send_keys(Keys.ENTER)

        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')
        self.assertTrue(any(row.text == '1: 공작깃털 사기' for row in rows), )

        # 추가 아이템을 입력할 수 있는 여분의 텍스트 상자 존재
        # 다시 '공작깃털을 이용해서 그물 만들기' 입력
        self.fail('Finish teh test!')

        # 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보임
        # 입력한 목록을 저장하는 URL 생성

        # 해당 URL에 접속하면 작업 목록 확인 가능


if __name__ == '__main__':
    unittest.main(warnings='ignore')
```

개발 서버를 실행 시킨 상태에서 `python functional_tests.py` 명령어로 테스트를 실행하면 h1 요소를 찾을 수 없다는 에러가 발생한다.

> selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: h1

### 상수와 템플릿

lists/tests.py의 단위 테스트를 보면 html 문자열을 확인하고 있다. 하지만 이는 html을 확인하는 좋은 방법이 아니다. html을 문자열로 테스트 하는 것처럼 상수를 테스트 하는 것도 효율적인 방법이 아니다. 단위 테스트는 **로직**, **흐름 제어** 그리고 **설정** 등을 테스트 하는 것이다. 더 나은 단위 테스트를 위해서는 템플릿을 이용하는 것이 나으며 django에서도 템플릿을 제공하므로 앞으로는 템플릿을 이용하여 단위 테스트를 하도록 한다.

lists/templates 폴더를 만들고 home.html 파일을 만들어 아래 내용을 추가하고 lists/views.py도 수정한다. HttpResponse를 반환하던 것을 render 함수를 이용하여 request와 home.html을 인수로 넣는다.

```html
<!-- lists/templates/home.html  -->

<html>
<title>To-Do lists</title>
</html>
```

```python
# lists/views/py


from django.shortcuts import render


def home_page(request):
    return render(request, 'home.html')
```

코드를 다 수정한 후 `python manage.py test` 명령어로 테스트를 하면 템플릿을 찾을 수 없다는 에러가 나온다.

> raise TemplateDoesNotExist(template_name, chain=chain)
> django.template.exceptions.TemplateDoesNotExist: home.html

 템플릿을 찾을 수 없던 이유는 아직 settings.py의 INSTALLED_APPS에 lists 앱을 등록하지 않았기 때문이다.

```python
# superlists/settings.py
...
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'lists',
]
...
```

앱을 등록한 후 다시 테스트를 하면 AssertionError가 발생하게 된다.

>  self.assertTrue(response.content.startswith(b'<html>'))
> AssertionError: False is not true

세 개의 assertion 중에 마지막 부분이 문제가 있다. 디버깅을 위해 `print(repr(response.content))` 를 추가한다

```python
# lists/tests.py
		
  	# 생략
    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        print(repr(response.content))
        
```

다시 테스트를 하면 마지막에 `\n` 이 추가된 것을 확인할 수 있다. 마지막 assertion을 수정하고 다시 테스트를 하면 드디어 OK를 볼 수 있다. 앞에서 상수를 테스트하지 않기 위해 템플릿을 사용한다는 이야기를 했다. 이를 위해 테스트를 수정하도록 한다. response.content 바이트 데이터를 파이썬 유니코드 문자열로 변환하기 위해 `decode()` 메서드를 이용했다. 또한, `render_to_string` 함수를 이용해서 세 개의 assertion을 대체했다. 이번에도 테스트 결과는 OK가 나온다.

```python
# lists/tests.py

from django.http import HttpRequest
from django.test import TestCase
from django.template.loader import render_to_string
from django.urls import resolve

from .views import home_page


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)

    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        expected_html = render_to_string('home.html')
        self.assertEqual(response.content.decode(), expected_html)
```

단위 테스트는 모두 통과했지만 기능 테스트는 실패한 상황이기 때문에 코드를 계속 수정하도록 한다. 마지막 기능 테스트의 결과는 h1 요소를 찾을 수 없다는 에러였다. home.html에  \<h1> 태그를 추가하도록 한다.

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
    <h1>Your To-Do lists</h1>
    </body>
</html>
```

테스트를 실행하면 id가 id_new_item인 요소를찾을 수 없다는 에러를 볼 수 있다.

>  raise exception_class(message, screen, stacktrace)
>selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [id="id_new_item"]

해당 id를 가지는 input을 추가해준다.

```html
    <h1>Your To-Do lists</h1>
    <input id="id_new_item">
```

다음은 AssertionError다.

>  self.assertEqual(inputbox.get_attribute('placeholder'), '작업 아이템 입력')
>AssertionError: '' != '작업 아이템 입력'
> 
>   \+ 작업 아이템 입력

에러 내용에 맞게 placeholder를 추가해준다.

```html
	<input id="id_new_item" placeholder="작업 아이템 입력">
```

다음은 id_list_table id를 가진 요소를 찾지 못했다.

> selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [id="id_list_table"]

해당 id를 가진 table을 추가한다.

```html
		<input id="id_new_item" placeholder="작업 아이템 입력">
    <table id="id_list_table"></table>
```

또 다시 AssertionError가 발생한다. 에러가 발생한 39번째 줄을 보면 assertTrue 함수를 사용하고 있는데 대부분의 assert 함수는 사용자가 정의한 메세지를 인수로 지정할 수 있다. 메세지를 추가한 후 다시 테스트를 해보도록 한다.

> File "functional_tests.py", line 39, in test_can_start_a_list_and_retrieve_it_later
>     self.assertTrue(any(row.text == '1: 공작깃털 사기' for row in rows), )
> AssertionError: False is not true

```python
# functional_tests.py

		# 생략
		table = self.browser.find_element_by_id('id_list_table')
    rows = table.find_elements_by_tag_name('tr')
		self.assertTrue(any(row.text == '1: 공작깃털 사기' for row in rows), "신규 작업이 테이블에 표시되지 않는다")

```

아까 나온 에러에서 추가한 메시지를 볼 수 있다.

> AssertionError: False is not true : 신규 작업이 테이블에 표시되지 않는다



## 5장

### POST 요청과 Form

마지막 기능 테스트에서는 신규 작업이 저장되지 않아 테이블에 표시되지 않았기 때문에 POST 요청을 통해 사용자 입력을 저장하도록 한다. POST 요청을 보내기 위해서 input에 name 속성도 추가해준다.

```html
...
<h1>Your To-Do lists</h1>
<form method="post">
    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
</form>
<table id="id_list_table"></table>
...
```

테스트를 하자 책과 다른 에러가 발생했다.

> selenium.common.exceptions.StaleElementReferenceException: Message: The element reference of <table id="id_list_table"> is stale; either the element is no longer attached to the DOM, it is not in the current frame context, or the document has been refreshed

우선 책의 디버깅 방법에 따라 time.sleep()을 추가하고 다시 테스트를 해본다.

```python
				...
				inputbox.send_keys(Keys.ENTER)

        import time
        time.sleep(10)

        table = self.browser.find_element_by_id('id_list_table')
        ...
```

브라우저에서 CSRF 에러 화면이 나오고 커맨드는 책과 같은 에러가 나왔다.

> selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [id="id_list_table"]

템플릿에 csrf token을 추가하도록 한다.

```html
<form method="post">
    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
    {% csrf_token %}
</form>
```

다시 테스트하면 다시 AssertionError가 발생하는데 이는 POST 요청을 아직 서버와 연결하지 않았기 때문이다.

> AssertionError: False is not true : 신규 작업이 테이블에 표시되지 않는다

### 서버에서 POST 요청 처리

home_page 뷰에 적용할 새로운 단위 테스트를 추가하도록 한다. 새로 추가한 함수는 POST 요청 처리와 반환된 HTML이 신규 아이템 텍스트를 포함하고 있는지 확인한다.

```python
# lists/tests.py
    def test_home_page_returns_correct_html(self):
        # 생략

    def test_home_page_can_save_a_POST_request(self):
        request = HttpRequest()
        request.method = 'POST'
        request.POST['item_text'] = '신규 작업 아이템'
        response = home_page(request)
        self.assertIn('신규 작업 아이템', response.content.decode())
```

테스트를 하자 책과 다르게 3개의 테스트 중 2개가 실패했다고 나왔다. csrf token을 추가하면서 기존의 test_home_page_returns_correct_html 테스트도 문제가 생긴 것이다.

> FAIL: test_home_page_can_save_a_POST_request (lists.tests.HomePageTest)
> Traceback (most recent call last):
>   File "/Users/superlists/lists/tests.py", line 39, in test_home_page_can_save_a_POST_request
>     self.assertIn('신규 작업 아이템', response.content.decode())
> AssertionError: '신규 작업 아이템' not found in '<html>\n<head>\n    <title>To-Do lists</title>\n</head>\n<body>\n<h1>Your To-Do lists</h1>\n<form method=\n    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">\n    <input type="hidden" name="csrfmiddlewaretoken" value="oKBO0wvUR78So0mfpakTccTUcTKCw9mEkDKD2DfFdDFiD8PrgDK6sbtKd">\n</form>\n<table id="id_list_table"></table>\n</body>\n</html>\n'
>
> ======================================================================
> FAIL: test_home_page_returns_correct_html (lists.tests.HomePageTest)
> Traceback (most recent call last):
>   File "/Users/superlists/lists/tests.py", line 32, in test_home_page_returns_correct_html
>     self.assertEqual(response.content.decode(), expected_html)
> AssertionError: '<htm[180 chars]n    <input type="hidden" name="csrfmiddleware[141 chars]l>\n' != '<htm[180 chars]n    \n</form>\n<table id="id_list_table"></ta[20 chars]l>\n'

다행히 google groups에서 이 책의 그룹을 찾아 해결책도 찾을 수 있었다. 템플릿에 추가한 {% csrf_token %} 템플릿 태그는 \<input type='hidden'>으로 변환되는데 이 input 태그를 없애는 함수를 추가했다.

```python
import re
from django.http import HttpRequest
from django.test import TestCase
from django.template.loader import render_to_string
from django.urls import resolve
from .views import home_page

def remove_csrf_tag(text):
    return re.sub(r'<[^>]*csrfmiddlewaretoken[^>]*>', '', text)


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)

    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        expected_html = render_to_string('home.html')
        self.assertEqual(
            remove_csrf_tag(response.content.decode()),
            remove_csrf_tag(expected_html)
        )
```

그러자 남은 에러는 책과 같이 '신규 작업 아이템'을 찾을 수 없다는 AssertionError이다. views.py에서 if로 POST 요청에 대한 처리를 추가하면 3개의 테스트를 모두 통과하는 것을 확인할 수 있다.

```python
from django.http import HttpResponse
from django.shortcuts import render


def home_page(request):
    if request.method == 'POST':
        return HttpResponse(request.POST['item_text'])
    return render(request, 'home.html')
```

### 템플릿에 변수 전달하기

템플릿 구문을 이용하면 views.py에 있는 변수를 템플릿에 전달할 수 있다. 템플릿에 파이썬 객체를 추가하고 테스트 코드도 이에 맞춰 수정하도록 한다. render_to_string 함수 두번째 인수에 변수명과 값을 추가하였다. 

```html
<!-- 생략 -->
<form method="post">
    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
    {% csrf_token %}
</form>
<table id="id_list_table">
    <tr>
        <td>
            {{ new_item_text }}
        </td>
    </tr>
</table>
...
```

```python
# lists/tests.py
	
  # 생략
  
  def test_home_page_can_save_a_POST_request(self):
        request = HttpRequest()
        request.method = 'POST'
        request.POST['item_text'] = '신규 작업 아이템'
        response = home_page(request)
        self.assertIn('신규 작업 아이템', response.content.decode())
        expected_html = render_to_string('home.html', {'new_item_text': '신규 작업 아이템'})
        # self.assertEqual(remove_csrf_tag(response.content.decode()), remove_csrf_tag(expected_html))
        self.assertEqual(response.content.decode(), expected_html)
```

테스트 결과는 아직 views.py에서 new_item_text에 대한 처리를 하지 않았기 때문에 실패한다. views.py도 수정하도록 한다.

```python
def home_page(request):    
    return render(request, 'home.html', {'new_item_text': request.POST['item_text']})
```

테스트를 하자 책과는 다른 에러가 발생했다.

> raise MultiValueDictKeyError(key)
> django.utils.datastructures.MultiValueDictKeyError: 'item_text'

우선 책의 방법대로 수정해보도록 한다. item_text 값을 가져오도록 하고 없으면 빈 값이 들어가도록 해서 테스트를 통과했다. 

```python
def home_page(request):
    return render(request, 'home.html', {'new_item_text': request.POST.get('item_text', '')})
```

funtional_tests.py를 다시 시도하면 같은 에러메세지가 나와서 에러 메세지를 좀 더 구체적으로 수정하도록 한다.

```python
				# 생략

  			table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')
        self.assertTrue(any(row.text == '1: 공작깃털 사기' for row in rows),
                        f"신규 작업이 테이블에 표시되지 않는다 - 해당 텍스트:{table.text}")
```

혹은 assertTrue를 assertIn으로 수정하는 방법도 있다.

> self.assertIn('1: 공작깃털 사기', [row.text for row in rows])
> AssertionError: '1: 공작깃털 사기' not found in ['공작깃털 사기']

화면에는 공작깃털 사기만 나오고 '1:'이 포함되어 있지 않아서 에러가 발생한 것이다. 가장 빨리 테스트를 통과하는 방법은 템플릿을 수정하는 것이다.

```html
<td>1: {{ new_item_text }}</td>
```

하지만 두번째 아이템을 추가하자 다시 에러가 발생한다. `StaleElementReferenceException` 에러가 발생했기 때문에 중간에 time.sleep()을 추가하였다.

```python
				# 생략
  
  			self.assertIn('1: 공작깃털 사기', [row.text for row in rows])
        
        # 추가 아이템을 입력할 수 있는 여분의 텍스트 상자 존재
        # 다시 '공작깃털을 이용해서 그물 만들기' 입력
        inputbox = self.browser.find_element_by_id('id_new_item')
        inputbox.send_keys('공작깃털을 이용해서 그물 만들기')
        inputbox.send_keys(Keys.ENTER)

        import time
        time.sleep(1)

        # 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보임
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')
        self.assertIn('1: 공작깃털 사기', [row.text for row in rows])
        self.assertIn('2: 공작깃털을 이용해서 그물 만들기', [row.text for row in rows])

        # 입력한 목록을 저장하는 URL 생성
        self.fail('Finish the test!')
```

테스트를 하자 아래의 에러가 발생했다.

> self.assertIn('1: 공작깃털 사기', [row.text for row in rows])
> AssertionError: '1: 공작깃털 사기' not found in ['1: 공작깃털을 이용해서 그물 만들기']

 반복되는 코드가 있기 때문에 helper 함수를 만들어 처리하도록 한다. `StaleElementReferenceException` 에러 때문에 함수 안에 time.sleep()을 추가했다. 테스트 결과는 아까와 동일하다.

```python
    def check_for_row_in_list_table(self, row_text):
    		time.sleep(1)
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_element_by_tag_name('tr')
        self.assertIn(row_text, [row.text for row in rows])

    def test_can_start_a_list_and_retrieve_it_later(self):
        # 생략
        
```

### Django ORM & Model

객체 관계형 매핑(Object RElational Mapper)은 DB의 테이블, 레코드, 칼럼 형태로 저장되어 있는 데이터를 추상화한 것이다. ORM을 사용하여 단위 테스트를 작성하도록 한다.

```python
# lists/tests.py

from .models import Item

# HomePageTest 생략

class ItemModelTest(TestCase):

    def test_saving_and_retrieving_items(self):
        first_item = Item()
        first_item.text = '첫번째 아이템'
        first_item.save()
        second_item = Item()
        second_item.text = '두번째 아이템'
        second_item.save()

        saved_items = Item.objects.all()
        self.assertEqual(saved_items.count(), 2)

        first_saved_item = saved_items[0]
        second_saved_item = saved_items[1]
        self.assertEqual(first_saved_item.text, '첫번째 아이템')
        self.assertEqual(second_saved_item.text, '두번째 아이템')
```

테스트를 실행하면 `ImportError: cannot import name 'Item'` 에러메세지를 볼 수 있다. models.py에 가서 Item 클래스를 추가하도록 한다.

```python
# lists/models.py

from django.db import models


class Item(models.Model):
    pass
```

Item 클래스를 추가했는데도 `django.db.utils.OperationalError: no such table: lists_item` 에러가 발생한다. models.py에 클래스를 추가한 다음에는 migration을 해야하는데 `python manage.py makemigrations` 명령어로 models.py에 작성한 내용을 DB에 적용하도록 한다. 명령어 실행 후 새로운 에러를 볼 수 있다.

> self.assertEqual(first_saved_item.text, '첫번째 아이템')
> AttributeError: 'Item' object has no attribute 'text'

Item 클래스에 text를 추가하도록 한다. 추가한 후에는 DB에 적용되도록 다시 마이그레이션을 해야한다. 드디어 테스트를 모두 통과했다.

```python
from django.db import models


class Item(models.Model):
    text = models.TextField(default='')
```

### POST요청 데이터베이스에 저장하기

test_home_page_can_save_a_POST_request 함수에서 POST 요청을 DB에 저장하도록 수정한다. 

```python
# lists/tests.py

  def test_home_page_can_save_a_POST_request(self):
        request = HttpRequest()
        request.method = 'POST'
        request.POST['item_text'] = '신규 작업 아이템'

        response = home_page(request)

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, '신규 작업 아이템')

        self.assertIn('신규 작업 아이템', response.content.decode())
        expected_html = render_to_string('home.html', {'new_item_text': '신규 작업 아이템'})
        self.assertEqual(remove_csrf_tag(response.content.decode()), remove_csrf_tag(expected_html))

```

에러가 발생했다.

> self.assertEqual(Item.objects.count(), 1)
> AssertionError: 0 != 1

views.py에서 Item을 만들어 text를 추가하도록 수정한다.

```python
from django.shortcuts import render
from .models import Item


def home_page(request):
    item = Item()
    item.text = request.POST.get('item_text', '')
    item.save()

    return render(request, 'home.html', {'new_item_text': request.POST.get('item_text', '')})
```

테스트를 통과했다. 마지막으로 views.py의 render 함수 뒷 부분을 조금 수정한다.

```python
def home_page(request):
    item = Item()
    item.text = request.POST.get('item_text', '')
    item.save()

    return render(request, 'home.html', {'new_item_text': item.text})
```

모든 request에서 비어 있는 것은 저장하지 않도록 테스트가 필요하다.  단위 테스트에 테스트를 하나 더 추가하도록 한다.

```python
# lists/tests.py

	def test_home_page_can_save_a_POST_request(self):
     # 생략
    
  def test_home_page_only_saves_items_when_necessary(self):
     request = HttpRequest()
     home_page(request)
     self.assertEqual(Item.objects.count(), 0)
```

`AssertionError: 1 != 0` 에러 메세지를 확인할 수 있다. views.py에서 좀 더 수정이 필요하다. POST 요청일 때는 item_text의 값을 가져와서 create() 메서드로 새로운 Item 객체를 생성한다. 이와 동시에 text에 가져온 값도 저장한다. 수정 후 테스트를 통과한다.

```python
def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']
        Item.objects.create(text=new_item_text)
    else:
        new_item_text = ''

    return render(request, 'home.html', {'new_item_text': new_item_text})
```

### POST 후 redirection

`new_item_text = ''` 로 처리한 부분이 찝찝하다. 하지만 POST 후에는 redirection하라는 말에 따라 테스트와 views.py를 수정하여 이를 보완하도록 한다.

```python
# lists/tests.py

    def test_home_page_can_save_a_POST_request(self):
        request = HttpRequest()
        request.method = 'POST'
        request.POST['item_text'] = '신규 작업 아이템'

        response = home_page(request)

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, '신규 작업 아이템')

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/')
```

```python
# lists/views.py

from django.shortcuts import render, redirect
from .models import Item


def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/')

    return render(request, 'home.html')
```

test_home_page_can_save_a_POST_request 테스트가 두 가지의 기능을 테스트 하고 있다. 단위 테스트는 각 테스트가 하나의 테스트만 하는 것이 좋기 때문에 테스트를 나누도록 함수를 하나 더 추가하도록 한다.

```python
# lists/tests.py

		def test_home_page_can_save_a_POST_request(self):
        request = HttpRequest()
        request.method = 'POST'
        request.POST['item_text'] = '신규 작업 아이템'

        response = home_page(request)

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, '신규 작업 아이템')

    def test_home_page_redirects_after_POST(self):
        request = HttpRequest()
        request.method = 'POST'
        request.POST['item_text'] = '신규 작업 아이템'

        response = home_page(request)

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/')
```

### 템플릿에 있는 아이템 렌더링

템플릿이 여러 아이템을 출력할 수 있는지 확인하는 테스트를 새로 추가한다.

```python
# lists/tests.py

    def test_home_page_displays_all_list_items(self):
        Item.objects.create(text='item 1')
        Item.objects.create(text='item 2')

        request = HttpRequest()
        response = home_page(request)

        self.assertIn('item 1', response.content.decode())
        self.assertIn('item 2', response.content.decode())
```

`AssertionError: 'item 1' not found` 에러메세지와 함께 테스트가 실패한다. 템플릿에서 for loop로 리스트를 처리하도록 한다. 그리고 템플릿으로 객체를 전달하도록 views.py 또한 수정한다.

```html
<table id="id_list_table">
    {% for item in items %}
        <tr><td>1: {{ item.text }}</td></tr>
    {% endfor %}
</table>
```

```python
def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/')
    
    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

단위 테스트는 통과했지만 기능 테스트는 `AssertionError: 'To-Do' not found in 'OperationalError at /'` 메세지와 함께 실패한다. http://localhost:8000 에 들어가면 `no such table: lists_item `이라는 디버그 메세지를 확인할 수 있다. 마이그레이션은 했지만 진짜 데이터베이스를 만들지 않았기 때문이다. `python manage.py migrate` 명령어를 실행하도록 한다. 이제 아까와 같은 에러는 뜨지 않고 아이템 번호만 수정하면 된다. 템플릿 태그의 `forloop.counter` 를  사용하도록 한다.

```html
    {% for item in items %}
        <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
    {% endfor %}
```

