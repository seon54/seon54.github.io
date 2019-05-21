---
layout: post
comments: true
title: Custom user model 만들기
category: Python
shortinfo: AbstractBaseUser를 상속하여 cumsom user model을 만들어봅니다.
tags:
- Python
- Django
---


# Custom user model 만들기

### AbstractBaseUser
AbstractBaseUser는 해시 비밀번호와 토큰화된 비밀번호 리셋을 포함한다. 

##### USERNAME_FIELD 
​	user model에서 고유한 식별자로 사용되는 필드명을 적는다. username, email 또는 그 외의 것들도 사용 가능하다. 이 필드는 항상 `unique=True`이어야 한다. 

##### EMAIL_FIELD 
​	user model에서 이메일 필드명을 적는다. 이 값은 `get_email_field_name()`의 리턴값으로 사용된다.

##### REQUIRED_FIELDS
​	python manage.py createsuperuser`로 사용자를 만들 때 필수로 입력하게 되는 필드 리스트이다. USERNAME_FIELD의 필드와 password는 항상 입력되기 때문에 포함시키지 않는다.

```python
from django.contrib.auth.models import AbstractBaseUser


class CustomUser(AbstractBaseUser):
    name = models.CharField(max_length=20, unique=True)
   	email = models.EmailField(max_length=254)
    date_of_birth = models.DateField()
   	    
    USERNAME_FIELD = 'name'
    EMAIL_FIELD = 'email'
    REQUIRED_FIELDS = ['email', 'date_of_birth']
```



### **BaseUserManager**
user model을 위해 custom manager 또한 정의해야 한다. 장고의 기본 user 모델처럼 username, email, is_staff, is_active, is_superuser, last_login, date_joined 필드를 정의한다면 장고의 `UserManager`를 사용하면 된다. 다른 필드를 정의했을 경우, `BaseUserManager`를 상속한다. BaseUserManager는 `create_user`와 `create_superuser` 메소드를 제공한다.

##### create_user(*username_field*, password=None, **other_fields)
```python
def create_user(self, email, date_of_birth, password=None):
    # username 필드와 REQUIRED_FIELDS의 필드가 모두 포함되어야 한다.
    # 여기서는 email이 username으로 사용되었고 date_of_birth가 필수 필드로 포함되었다.
```

##### create_superuser(*username_field*, password, **other_fields)
```python
def create_superuser(self, email, date_of_birth, password):
    # username 필드와 REQUIRED_FIELDS의 필드가 모두 포함되어야 한다.
    # 여기서는 email이 username으로 사용되었고 date_of_birth가 필수 필드로 포함되었다.
```



### django.contrib.admin

custom user model이 admin에서도 작동하기 위해서는 몇 가지 속성과 메서드를 정의해야 한다.

 **is_staff** : 사용자가 관리자 사이트 접근이 허용될 경우 True
 **is_active** : 사용자 계정이 현재 active 할 경우 True
 **has_perm(perm, obj=None)** : 사용자가  지정된 권한(permission)을 가질 경우 True. obj가 제공되면, 특정 객체 인스턴스에 대해 권한을 확인해야 한다.
 **has_module_perm(app_label)** : 사용자가 주어진 앱의 모델에 접근 권한을 가질 경우 True



### 예제

아래 예제에서는 email을 username으로 사용하고 date_of_birth가 필수 필드이다. 권한 확인은 하지 않는다.

```python
# customauth/models.py

from django.db import models
from django.contrib.auth.models import BaseUserManager, AbstractBaseUser


class MyUserManager(BaseUserManager):
    def create_user(self, email, date_of_birth, password=None):
        if not email:
            raise ValueError('Users must have an email address')
            
        user = self.model(
        	email=self.normalize_email(email),
            date_of_birth=date_of_birth,
        )
        
        user.set_password(password)
        user.save(using=self._db)
        return user
    
    def create_superuser(self, email, date_of_birth, password):
        user = self.create_user(
        	email,
            password=password,
            date_of_birth=date_of_birth,
        )
        user.is_admin = True
        user.save(using=self._db)
        return user
    
    
class MyUser(AbstractBaseUser):
    email = models.EmailField(verbose_name='email address', max_length=255, unique=True,)
    date_of_birth = models.DateField()
    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)
    
    objects = MyUserManager()
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['date_of_birth']
    
    def __str__(self):
        return self.email
    
    def has_perm(self, perm, obj=None):
        return True
    
    def has_module_perm(self, app_label):
        return True
    
    @property
    def is_staff(self):
        return self.is_admin
```
model에서 model과 manager를 작성 후, settings.py에서 사용자 모델을 등록해준다.
```python
# settings.py

...
AUTH_USER_MODEL = 'customauth.MyUser'
```



참조

[Django - Specifying a custom user model](<https://docs.djangoproject.com/en/2.2/topics/auth/customizing/#specifying-a-custom-user-model>)

