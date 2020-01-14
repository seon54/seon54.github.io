---
layout: post
comments: true
title: 파이썬을 이용한 클린 코드를 위한 테스트 주도 개발 6장
category: Python
shortinfo: django test client를 이용한 테스트 
tags:
- python
- django
- TDD
---



## 6장

### 기능 테스트 내에서 테스트 격리

기능 테스트를 실행하면 앞 테스트의 목록 아이템이 데이터베이스에 남게 되며 이는 다음 테스트 결과에 영향을 미친다. 단위 테스트의 경우, 새로운 테스트 데이터베이스를 생성하며 개별 테스트를 할 때마다 데이터베이스가 초기화된다. django에서 제공하는 LiveServerTestCase 클래스를 이용하면 기능 테스트의 데이터베이스 문제를 해결할 수 있다. 기존의 `functional_tests.py`를 `superlists/functional_tests/tests.py`로 폴더를 생성하고 파일명을 바꾸도록 한다. functional tests 폴더는 파이썬 패키지 디렉터리로서 `__init__.py` 파일이 있어야 한다. 앞으로 기능 테스트를 실행할 때는 아래 명령어를 실행하도록 한다.

```shell
python manage.py test functional_tests
```

기능 테스트의 NewVisitorTest의 클래스를 LiveServerTestCase로 변경하고 로컬호스트를 8000번 포트로 접속하는 부분도 수정한다. 또한, 이제는 python functional_tests.py 로 실행하지 않기 때문에 `if __name__ == '__main__'` 으로 시작하는 부분은 지워도 된다.

```python
# functional_tests/tests.py

from django.test import LiveServerTestCase


class NewVisitorTest(LiveServerTestCase):
		# 생략
    
    def test_can_start_a_list_and_retrieve_it_later(self):
      # 에디스는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 듣고 웹사이트를 확인하러 간다.
      self.browser.get(self.live_server_url)
      
      # 생략
```

이제 `python manage.py test` 명령어를 실행하면 기능 테스트와 단위 테스트가 모두 실행된다. 기능 테스트만 실행하고 싶을 때는 뒤에 `functional_tests`를, 단위 테스트만 실행하고 싶을 때는 `lists`를 붙여주면 된다.

### TDD를 이용한 새로운 설계 반영하기

새로운 설계를 기능 테스트에 반영해보도록 한다. 에디스가 첫번째 아이템을 전송하면 새로운 목록을 만들어 신규 작업 아이템을 추가하고 목록에 접근할 수 있는 URL을 제공한다. 기능 테스트를 아래와 같이 수정한다. assertRegex() 함수는 지정한 정규표현과 문자열이 일치하는지 확인한다.

```python
# functional_test/tests.py

class NewVisitorTest(LiveServerTestCase):
  	# 생략
    
    def test_can_start_a_list_and_retrieve_it_later(self):
      # 생략
      
      # 엔터키를 치면 페이지가 갱신되고 작업 목록에
      # '1: 공작깃털 사기' 아이템 추가
      inputbox.send_keys(Keys.ENTER)
      edith_list_url = self.browser.current_url
      self.assertRegex(edith_list_url, '/lists/.+')
      self.check_for_row_in_list_table("1: 공작깃털 사기")
```

다른 사람이 접속할 경우 에디스의 목록이 보이지 않는 것을 확인하고 각 사용자마다 URL이 생성되었는지 확인하기 위해 마지막 부분도 수정한다.

```python
# functional_test/tests.py

class NewVisitorTest(LiveServerTestCase):
  	# 생략
    
    def test_can_start_a_list_and_retrieve_it_later(self):
      # 생략
      
      # 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보임
      self.check_for_row_in_list_table("1: 공작깃털 사기")
      self.check_for_row_in_list_table("2: 공작깃털을 이용해서 그물 만들기")

      ## 새로운 사용자 프란시스 접속
      ## 새로운 브라우저 세션을 이용해 에디스의 정보가 쿠키를 통해 유입되는 것 방지
      self.browser.quit()
      self.browser = webdriver.Firefox()

      # 프란시스가 홈페이지에 접속하고 에디스의 리스트는 보이지 않음
      self.browser.get(self.live_server_url)
      page_text = self.browser.find_element_by_tag_name('body').text
      self.assertNotIn('공작깃털 사기', page_text)
      self.assertNotIn('그물 만들기', page_text)

      # 프란시스가 새로운 작업 아이템 입력
      inputbox = self.browser.find_element_by_id('id_new_item')
      inputbox.send_keys('우유 사기')
      inputbox.send_keys(Keys.ENTER)

      # 프란시스가 전용 URL 취득
      francis_list_url = self.browser.current_url
      self.assertRegex(francis_list_url, '/lists/.+')
      self.assertNotEqual(francis_list_url, edith_list_url)

      # 에디스가 입력한 흔적이 없는지 확인
      page_text = self.browser.find_element_by_tag_name('body').text
      self.assertNotIn('공작깃털 사기', page_text)
      self.assertIn('우유 사기', page_text)
```

테스트를 실행하면 에러가 발생한다.

> AssertionError: Regex didn't match: '/lists/.+' not found in 'http://localhost:52142/'

위의 에러를 보면 두번째 아이템에 개별 URL과 식별자를 적용해야 함을 알 수 있다. lists/tests.py를 수정하도록 한다.

```python
# lists/tests.py


class HomePageTest(TestCase):
		# 생략
    
    def test_home_page_redirects_after_POST(self):
        request = HttpRequest()
        request.method = 'POST'
        request.POST['item_text'] = '신규 작업 아이템'

        response = home_page(request)

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], /lists/the-only-list-in-the-world/)
```

/lists/the-only-list-in-the-world/는 앞에서 설계한 URL이 아니지만 현재 하나의 목록만 지원하므로 하나의 URL만 이용하도록 하고 여러 목록과 메인 페이지를 위한 URL은 추후에 추가하도록 한다. 이에 맞춰 views.py도 수정한다.

```python
# lists/views.py


def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/lists/the-only-list-in-the-world/')

    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

책에서는 수정 후 기능 테스트를 하면 목록 테이블에 대한 에러가 나와야 하는데 계속 정규식에 대한 에러가 나와서 검색해보니 `time.sleep()` 을 추가해야 한다는 방법을 보았다. `inputbox.send_keys(Keys.ENTER)` 다음에 time.sleep()을 추가했고 시간 설정은 짧게 1로 설정했다. 그 후 기능 테스트를 하니 목록 테이블에 대한 에러를 확인할 수 있었다.

```python
# functional_test/tests.py


# 추가 아이템을 입력할 수 있는 여분의 텍스트 상자 존재
# 다시 '공작깃털을 이용해서 그물 만들기' 입력
inputbox = self.browser.find_element_by_id('id_new_item')
inputbox.send_keys('공작깃털을 이용해서 그물 만들기')
inputbox.send_keys(Keys.ENTER)
time.sleep(1)

# 프란시스가 새로운 작업 아이템 입력
inputbox = self.browser.find_element_by_id('id_new_item')
inputbox.send_keys('우유 사기')
inputbox.send_keys(Keys.ENTER)
time.sleep(1)
```

> selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [id="id_list_table"]

### Django 테스트 클라이언트를 이용한 view, templates, URL 테스트

django 테스트 클라이언트를 이용하면 URL, view 함수 호출, template 렌더링을 테스트 할 수 있다. lists/tests.py를 아래와 같이 LiveViewTest 클래스를 추가하도록 한다. 이전의 테스트에서는 view를 바로 호출했지만 여기서는 `TestCase` 의 속성인 테스트 클라이언트 `self.client` 를 사용하고 있다. 여기에 테스트할 URL을 get 메서드에서 호출한다. `self.assertIn('item 1', response.content.decode())`은 간단하게 `assertContains`로 response에 새로 만든 아이템이 있는지 확인하도록 한다.

```python
# lists/tests.py

# 생략

class ListViewTest(TestCase):

    def test_displays_all_items(self):
        Item.objects.create(text='item 1')
        Item.objects.create(text='item 2')

        response = self.client.get('/lists/the-only-list-in-the-world/')

        self.assertContains(response, 'item 1')
        self.assertContains(response, 'item 2')
```

테스트를 실행하면 에러가 발생한다.

> AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404 (expected 200)

아직 URL이 추가되지 않았기 때문에 superlists/urls.py에 추가하고 lists/views.py에도 해당 URL을 위한 새로운 함수를 작성한다.

```python
# superlists/urls.py

urlpatterns = [
    path('', views.home_page, name='home'),
    path('lists/the-only-list-in-the-world/', views.view_list, name='view-list'),
    path('admin/', admin.site.urls),
]
```

```python
# lists/views.py

def view_list(request):
    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

단위 테스트는 모두 통과하고 기능 테스트는 아래의 에러 메세지가 나온다.

> AssertionError: '2: 공작깃털을 이용해서 그물 만들기' not found in ['1: 공작깃털 사기']

### Refactoring

단위 테스트에 있는 메소드를 살펴보기 위해 아래 명령어를 실행한다.

```shell
grep -E 'class|def' lists/tests.py
```

test_home_page_displays_all_list_items와 test_displays_all_items에서 모든 작업 아이템 목록을 확인하고 있다. test_home_page_displays_all_list_items 함수를 지우고 단위 테스트를 실행하면 7개의 테스트가 통과하는 것을 볼 수 있다.

### 목록을 위한 template

메인 페이지와 목록 뷰는 기능이 다르므로 별개의 html 템플릿을 가지는 것이 좋다.  home.html에는 하나의 입력상자를 가지고 새로 추가할 list.html에는 기존 아이템을 보여주는 테이블을 보여줄 것이다. 각각의 템플릿을 확인하는 테스트를 추가한다. 테스트를 실행하면 아직 list.html 파일을 만들지 않았기 때문에 에러가 나온다.

```python
# lists/tests.py

class LiveViewTest(TestCase):
		# 생략
    
    def test_uses_list_template(self):
        response = self.client.get('/lists/the-only-list-in-the-world/')
        self.assertTemplateUsed(response, 'list.html')
```

home.html의 내용을 복사해 lists/templates/list.html을 만들도록 한다. 이제 home.html에서는 아이템 출력이 필요없기 때문에 필요 없는 부분은 지우도록 하고 home_page 뷰에서도 아이템 목록을 보낼 필요가 없기 때문에 수정한다.

```python
# lists/views.py

def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/lists/the-only-list-in-the-world/')
    return render(request, 'home.html')


def view_list(request):
    items = Item.objects.all()
    return render(request, 'list.html', {'items': items})
```

```html
<!-- lists/templates/home.html -->

<html>
<head>
    <title>To-Do lists</title>
</head>
<body>
<h1>Your To-Do lists</h1>
<form method="post">
    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
    {% csrf_token %}
</form>
</body>
</html>
```

단위 테스트는 통과하지만 기능 테스트는 계속 두번째 아이템 입력에서 실패한다. list.html에서 \<form>에 action 속성이 없어 테스트를 할 때마다 동일 URL로 form이 전송되었다. list.html을 수정하고 다시 테스트를 한다.

```html
<!-- lists/templates/list.html -->
<html>
<head>
    <title>To-Do lists</title>
</head>
<body>
<h1>Your To-Do lists</h1>
<form method="post" action="/">
    <input name="item_text" id="id_new_item" placeholder="작업 아이템 입력">
    {% csrf_token %}
</form>
<table id="id_list_table">
    {% for item in items %}
        <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
    {% endfor %}
</table>
</body>
</html>
```

아이템을 새로 추가하게 되고 에디스와 프란시스의 URL을 확인하는 부분에서 에러가 발생한다.

> AssertionError: 'http://localhost:53377/lists/the-only-list-in-the-world/' == 'http://localhost:53377/lists/the-only-list-in-the-world/'

### 신규 목록 생성을 위한 테스트, URL, view

신규 작업 아이템을 추가하기 위해 새로운 URL을 추가할 것이다. lists/tests.py에도 이를 위한 새로운 클래스를 만든다. 기존의 test_home_page_can_save_a_POST_request 함수와 test_home_page_redirects_after_POST 함수를 가져와서 함수명을 수정하고 위에서 client를 사용하도록 수정한다.

```python
# lists/tests.py

# 생략

class NewListTest(TestCase):

    def test_saving_a_POST_request(self):
      self.client.post('/lists/new', data={'item_text': '신규 작업 아이템'})
      self.assertEqual(Item.objects.count(), 1)
      new_item = Item.objects.first()
      self.assertEqual(new_item.text, '신규 작업 아이템')

    def test_redirects_after_POST(self):
      response = self.client.post('/lists/new', data={'item_text': '신규 작업 아이템'})
      self.assertEqual(response.status_code, 302)
      self.assertEqual(response['location'], '/lists/the-only-list-in-the-world/')
```

테스트를 하면 위에서 추가한 두 개의 함수 모두 통과하지 못한다. post 요청을 보낸 신규 작업을 아직 데이터베이스에 저장하지 못하고 lists/new이 구축되지 않아서 404 응답을 받았다.

> self.assertEqual(Item.objects.count(), 1)
> AssertionError: 0 != 1
>
> self.assertEqual(response.status_code, 302)
> AssertionError: 404 != 302

이제 lists/new를 위한 url과 이를 위한 view를 추가한다.

```python
# superlists/urls.py

urlpatterns = [
    path('', views.home_page, name='home'),
    path('lists/the-only-list-in-the-world/', views.view_list, name='view-list'),
    path('lists/new', views.new_list, name='new-list'),
    path('admin/', admin.site.urls),
]
```

```python
# lists/views.py

def new_list(request):
    return redirect('/lists/the-only-list-in-the-world/')
```

테스트를 하면 AssertionError: 0 != 1 에러가 발생하므로 아이템을 생성하는 코드를 추가한다.

```python
# lists/views.py

def new_list(request):
  	Item.objects.create(text=request.POST['item_text'])
    return redirect('/lists/the-only-list-in-the-world/')
```

책에서는 `self.assertEqual(response['location'], '/lists/the-only-list-in-the-world/')` 에서 에러가 발생했지만 문제없이 넘어갔다. 책에서 이 에러를 해결하기 위해 수정한 내용은 아래와 같다.

```python
def test_redirects_after_POST(self):
  response = self.client.post('/lists/new', data={'item_text': '신규 작업 아이템'})
  self.assertRedirects(response, '/lists/the-only-list-in-the-world/')
```

아이템을 새로 추가하는 view를 작성했기 때문에 home_page 함수에 관련 코드와 이를 테스트 하는 test_home_page_only_saves_items_when_necessary 함수를 지우도록 한다.

```python
# lists/views.py

def home_page(request):
    return render(request, 'home.html')
```

새로 추가한 list/new URL을 home.html과 list.html의 \<form>에 추가하도록 한다.

```html
<form method="post" action="/lists/new">
```

### Model 수정

List라는 새로운 모델을 추가하고 단위테스트 또한 수정한다. 기존의 ItemModelTest의 클래스 이름을 바꾸고 List 모델에 대한 내용을 추가한다. 모델을 추가한 후에는 `python manage.py makemigrations` 명령어를 실행해야 한다.

```python
# lists/tests.py

from .models import Item, List

# 생략

class ListAndItemModelTest(TestCase):

    def test_saving_and_retrieving_items(self):
        list_ = List()
        list_.save()

        first_item = Item()
        first_item.text = '첫번째 아이템'
        first_item.list = list_
        first_item.save()

        second_item = Item()
        second_item.text = '두번째 아이템'
        second_item.list = list_
        second_item.save()
        saved_list = List.objects.first()
        self.assertEqual(saved_list, list_)

        saved_items = Item.objects.all()
        self.assertEqual(saved_items.count(), 2)

        first_saved_item = saved_items[0]
        second_saved_item = saved_items[1]
        self.assertEqual(first_saved_item.text, '첫번째 아이템')
        self.assertEqual(first_item.list, list_)
        self.assertEqual(second_saved_item.text, '두번째 아이템')
        self.assertEqual(second_item.list, list_)
```

```python
# lists/models.py

from django.db import models

class List(models.Model):
    pass

class Item(models.Model):
    text = models.TextField(default='')
    list = models.TextField(default='')
```

원래 단위 테스트를 실행하면 self.assertEqual(first_item.list, list_) 에서 에러가 발생하는 게 책에서 의도한 상황이었지만 django에서 버전이 업데이트 되면서 수정된 게 있는지 first_item.list도 List 객체로 저장되었다. 하지만 책의 의도대로 외래키에 대한 내용을 추가하고 마이그레이션 명령어도 실행한다.

```python
# lists/models.py

from django.db import models

class List(models.Model):
    pass
  
class Item(models.Model):
    text = models.TextField(default='')
    list = models.ForeignKey(List, on_delete=models.CASCADE, default=None)
```

그 후에 단위 테스트를 실행하면 3개의 에러가 발생한다. 에러가 발생한 함수는 다르지만 내용은 모두 같다. Item과 List의 관계에 대한 내용이 없어서 발생한 에러다.

> ERROR: test_displays_all_items (lists.tests.ListViewTest)
>
> sqlite3.IntegrityError: NOT NULL constraint failed: lists_item.list_id
>
> ERROR: test_redirects_after_POST (lists.tests.NewListTest)
>
> sqlite3.IntegrityError: NOT NULL constraint failed: lists_item.list_id
>
> ERROR: test_saving_a_POST_request (lists.tests.NewListTest)
>
> django.db.utils.IntegrityError: NOT NULL constraint failed: lists_item.list_id

먼저 ListViewTest를 수정한다

```python
# lists/tests.py

class ListViewTest(TestCase):

    def test_displays_all_items(self):
        list_ = List.objects.create()
        Item.objects.create(text='item 1', list=list_)
        Item.objects.create(text='item 2', list=list_)
				
        # 생략
```

남은 두 에러는 new_list 뷰에서 POST 할 때 발생하는 에러이므로 views.py를 수정한다.

```python
# lists/views.py

def new_list(request):
    list_ = List.objects.create()
    Item.objects.create(text=request.POST['item_text'], list=list_)
    return redirect('/lists/the-only-list-in-the-world/')
```

### 각 목록을 위한 하나의 고유 URL

목록을 위한 고유 식별자로 데이터베이스가 자동으로 생성하는 id를 사용할 수 있다. LiveViewTest의 test_displays_all_items 함수를 수정하여 특정 리스트에서만 출력되는 아이템을 확인하도록 한다.

```python
# lists/tests.py

class ListViewTest(TestCase):

    def test_displays_only_items_for_that_list(self):
        correct_list = List.objects.create()
        Item.objects.create(text='item 1', list=correct_list)
        Item.objects.create(text='item 2', list=correct_list)

        other_list = List.objects.create()
        Item.objects.create(text='other list item 1', list=other_list)
        Item.objects.create(text='other list item 2', list=other_list)

        response = self.client.get(f'/lists/{correct_list.id}/')

        self.assertContains(response, 'item 1')
        self.assertContains(response, 'item 2')
        self.assertNotContains(response, 'other list item 1')
        self.assertNotContains(response, 'other list item 2')
```

해당 URL을 추가하지 않았기 때문에 테스트를 실행하면 404 코드와 관련된 에러가 발생한다. 책에서 사용한 `url` 대신 `path`를 사용하고 정규식 표현을 위해서는 `re_path`를 사용했다.

```python
# superlists/urls.py

from django.contrib import admin
from django.urls import path, re_path
from lists import views


urlpatterns = [
    path('', views.home_page, name='home'),
    re_path(r'lists/(.+)/$', views.view_list, name='view-list'),
    path('lists/new', views.new_list, name='new-list'),
    path('admin/', admin.site.urls),
]
```

`view_list() takes 1 positional argument but 2 were given` 에러가 발생한다. view_list에 파라미터를 추가해준다.

```python
# lists/views.py

def view_list(request, list_id):
  #  생략
```

`AssertionError: 1 != 0 : Response should not contain 'other list item 1'` 에러를 해결하기 위해 뷰에서 어떤 아이템을 템플릿에 보낼지 처리하는 부분을 추가한다.

```python
# lists/views.py

def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    items = Item.objects.filter(list=list_)
    return render(request, 'list.html', {'items': items})
```

그 후에는 /lists/the-only-list-in-the-world/ URL을 테스트 하는 부분에서 에러가 발생한다. test_redirects_after_POST 함수와 new_list 뷰를 수정하도록 한다.

```python
# lists/tests.py

class NewListTest(TestCase):
  
    def test_redirects_after_POST(self):
        response = self.client.post('/lists/new', data={'item_text': '신규 작업 아이템'})
        new_list = List.objects.first()
        self.assertRedirects(response, f'/lists/{new_list.id}/')        
```

```python
# lists/views.py

def new_list(request):
    list_ = List.objects.create()
    Item.objects.create(text=request.POST['item_text'], list=list_)
    return redirect(f'/lists/{list_.id}/')
```

책에서는 다루지 않았지만 test_uses_list_template 함수에 대한 에러가 아직 남아 있어서 아래와 같이 수정하였다.

```python
# lists/tests.py

def test_uses_list_template(self):
  list_ = List.objects.create()
  response = self.client.get(f'/lists/{list_.id}/')
  self.assertTemplateUsed(response, 'list.html')
```

### 기존 목록에 아이템 추가하기

기능 테스트를 실행하면 하나의 목록에 아이템이 추가되지 않고 모든 POST마다 새로운 아이템을 만들고 있음을 알 수 있다. 기존 목록에 새로운 아이템을 추가하는 URL과 뷰, 테스트를 추가한다.

```python
# lists/tests.py

class NewItemTest(TestCase):
    def test_can_save_a_POST_request_to_an_existing_list(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        self.client.post(f'/lists/{correct_list.id}/add_item', data={'item_text': '기존 목록에 신규 아이템'})
        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, '기존 목록에 신규 아이템')
        self.assertEqual(new_item.list, correct_list)

    def test_redirects_to_list_view(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        response = self.client.post(f'/lists/{correct_list.id}/add_item', data={'item_text': '기존 목록에 신규 아이템'})
        self.assertRedirects(response, f'/lists/{correct_list.id}/')
```

테스트를 하면 두 개의 에러가 발생한다. 404 에러가 나올 것 같았지만 301이 나왔다. 이는 urls.py의 정규표현식 때문이다.  `(.+)`가 `{correct_list.id}/add_item` 에 매칭되어 django는 URL에서  `/`가 빠진 것으로 보는 것이다. URL에서 숫자만 추출하도록 수정하고 다시 테스트를 한다. 

> AssertionError: 0 != 1
>
> AssertionError: 301 != 302 : Response didn't redirect as expected: Response code was 301 (expected 302)

```python
# superlits/urls.py

urlpatterns = [
    path('', views.home_page, name='home'),
    re_path(r'lists/(\d+)/$', views.view_list, name='view-list'),
    path('lists/new', views.new_list, name='new-list'),
    path('admin/', admin.site.urls),
]
```

`404 != 302`로 에러메세지가 바꼈다. 이제 기존 목록에 신규 아이템을 추가하는 URL과 뷰를 추가한다.

```python
# superlits/urls.py

urlpatterns = [
    path('', views.home_page, name='home'),
    re_path(r'lists/(\d+)/$', views.view_list, name='view-list'),
    re_path(r'lists/(\d+)/add_item$', views.add_item, name='add-item'),
    path('lists/new', views.new_list, name='new-list'),
    path('admin/', admin.site.urls),
]
```

```python
# lists/views.py

def add_item(request, list_id):
    list_ = List.objects.get(id=list_id)
    return redirect(f'/lists/{list_.id}/')
```

`AssertionError: 0 != 1` 에러를 해결하기 위해 list_와 연관된 아이템을 생성하는 코드를 추가한다.

```python
# lists/views.py

def add_item(request, list_id):
    list_ = List.objects.get(id=list_id)
    Item.objects.create(text=request.POST['item_text'], list=list_)
    return redirect(f'/lists/{list_.id}/')
```

이제 새로 생성한 URL을 form에서 사용할 수 있도록 수정할 차례이다. 현재 작업중인 목록에 새로운 아이템을 추가하는 URL을 얻기 위해 다음과 같이 수정한다. `list.id`를 사용하기 위해서는 뷰에서 템플릿에 전달해야 한다. 이를 테스트 하는 함수를 추가한다.

```html
<form method="post" action="/lists/{{ list.id }}/add_item">
```

```python
# lists/tests.py

    def test_passes_correct_list_to_template(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()
        response = self.client.post(f'/lists/{correct_list.id}/')
        self.assertEqual(response.context['list'], correct_list)
```

아직 뷰에서 list를 전달하고 있지 않기 때문에 `KeyError: 'list'` 에러가 발생한다. vew_list를 수정한다.

```python
# lists/views.py

def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    return render(request, 'list.html', {'list': list_})
```

list.html에서 사용하고 있는 items를 못 받고 있어서 에러가 발생하게 된다. 이제 list를 보내주기 때문에 list를 이용해서 list와 관련된 item을 가져올 수 있다. 드디어 단위 테스트와 기능 테스트 모두 통과할 수 있게 됐다.

```django
<!-- 생략 -->

{% for item in list.item_set.all %}
        <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
{% endfor %}
```

마지막으로 lists/urls.py을 만들고 superlists/urls.py에는 `include`를 이용하여 urls.py를 추가하도록 한다. 

```python
# lists/urls.py

from django.urls import path, re_path
from lists import views


urlpatterns = [
    re_path(r'(\d+)/$', views.view_list, name='view-list'),
    re_path(r'(\d+)/add_item$', views.add_item, name='add-item'),
    path('new', views.new_list, name='new-list'),
]
```

```python
# superlists/urls.py

from django.contrib import admin
from django.urls import path, include
from lists import views

urlpatterns = [
    path('', views.home_page, name='home'),
    path('lists/', include('lists.urls')),
    path('admin/', admin.site.urls),
]
```

