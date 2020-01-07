---
layout: post
title: Django 15 - 사용자 인증(회원가입/로그인/로그아웃)
category: Django
tags: [Django]
comments: true
---



# 사용자 인증

> 회원 가입 및 로그인 등 사용자 인증에 관한 코드를 Django에서 구현해보도록 하자. 이들은 전적으로 새로운 기능으로, 이 기능만 담당하는 App을 새로 만드는 것이 적절함. (App 이름은 accounts로 명명함)
> - setttings.py - INSTALLED_APPS에 accounts 추가
> - accounts 앱 내에 urls.py 생성 및 INSTA 프로젝트 urls.py 내에 accounts 앱 경로 재 설정


### 회원가입

회원가입은 하나의 어떤 유저를 만드는 것. 이는 CRUD 로직 중 Create를 구현하는 것과 동일하다. Django에서는 user모델(이름, 아이디, 비밀번호 등)과 모델폼이 이미 구현되어 있기 때문에, 이들을 import 하여 views.py에 뿌려주기만 하면 된다. 기본적인 골자는 모델폼 기반의 create  함수에 대한 코드를 따른다.
- `UserCreationForm` 은 `django.contrib.auth.form` 내 구현되어있는 회원가입 모델 폼으로 import

```python
#views.py
from django.shortcuts import render, redirect
from django.contrib.auth.forms import UserCreationForm

def signup(request):
    #HTTP Method가 POST 인 경우
    if request.method == 'POST';
    	signup_form = UserCreationForm(request.POST)
        #모델폼의 유효성 검증이 valid할 경우, DB에 저장
        if signup_form.is_valid():
            signup_form.save()
        return redirect('posts:list')
    
    #HTTP Method가 GET 인 경우
    else:
        signup_form = UserCreationForm()
    
    return render(request, 'accounts/signup.html', {'signup_form':signup_form})
```

- signup.html
  - accounts\templates\accounts 내에 html 문서 생성

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>회원 가입</h1>

<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    {% raw %}{{ signup_form }}{% endraw %}
    <input type="submit" name="Submit"/>
</form>

{% raw %}{% endblock %}{% endraw %}
```



### 로그인 & 세션

세션이란, 웹 서버에서 임시로 클라이언트의 데이터를 갈무리하는 것을 뜻한다. 쿠키와 비슷한 역할이나,  쿠키는 클라이언트 측에 데이터를 갈무리 하는 반면에, 세션은 서버측에 데이터를 갈무리한다는 차이점이 있다. 주로 로그인, 온라인 쇼핑몰의 장바구니 등에 쓰인다 ([나무위키](https://namu.wiki/w/%EC%84%B8%EC%85%98))

웹 브라우저와 서버가 HTTP 프로토콜을 통해서 하는 모든 커뮤니케이션은 무상태(stateless)라고 합니다. 프로토콜이 무상태라는 뜻은 클라이언트와 서버 사이의 메시지가 완벽하게 각각 독립적이라는 뜻입니다.  여기엔 이전 메시지에 근거한 행동이나 순서(sequence)라는 것이 존재하지 않습니다. 결국, 만약 사이트가 클라이언트와 계속적인 관계를 유지하는 것을 당신이 원한다면, 당신이 직접 그 작업을 해야합니다.

세션이라는 것은 Django 그리고 대부분의 인터넷에서 사용되는 메카니즘으로, 사이트와 특정 브라우저 사이의 "state"를 유지시키는 것입니다. 세션은 당신이 매 브라우저마다 임의의 데이터를 저장하게 하고, 이 데이터가 브라우저에 접속할 때 마다 사이트에서 활용될 수 있도록 합니다. 세션에 연결된 각각의 데이터 아이템들은 "key"에 의해 인용되고, 이는 또다시 데이터를 찾거나 저장하는 데에 이용됩니다. ([장고 튜토리얼](https://developer.mozilla.org/ko/docs/Learn/Server-side/Django/Sessions))

장고 서버는 우리가 접속하고 있는 브라우저의 정보를 임시로 가지고 있음. 따라서 어떤 클라이언트가 어떤 페이지를 보고 있는지 등의 정보를 장고에서 알 수 있음. 로그인의 경우, 마치 파이썬에서 사용한 글로벌 변수처럼, 일반적으로 페이지를 이동하더라도 유지되어야 함.  우리는 세션을 이용하여 페이지가 이동하더라도 유지되도록, 로그인 기능을 구현할 수 있음.

- `AuthenticationForm` : 유저가 존재하는지를 검증하는 Django 내장 모델 폼. 사용자가 로그인 폼에 작성한 정보가 유효한지를 검증함
- `auth_login` (default: `login`)
  - 유저 정보를 세션에 생성 및 저장하는 역할을 하는 Django 내장 함수.
  - `login_form.get_user()` 를 통해 `login_form`에 저장된 유저 정보를 갖고와 세션에 유저 정보를 생성함
  - ***`AuthenticationForm` 은 유저가 존재하는지를 검증할 뿐, 세션과는 무관함을 유의하자.***

```python
#views.py

# AuthenticationForm 모델폼 import
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm

# jango.contrib.auth 내 함수 login을 import함.
# (views.py 내 정의한 함수 login과 구분하기 위해 auth_log로 재 명명함)
from django.contrib.auth import login as auth_login

#Session Create
def login(request):
    if request.method == 'POST':
        login_form = AuthenticationForm(request, request.POST)
        if login_form.is_valid():
            auth_login(request, login_form.get_user())
        return redirect('posts:list')
    
    else:
        login_form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'login_form' : login_form})


#urls.py
urlpatterns = [
    path('login/', views.login, name='login'),    
]
```

- login.html

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}
<h1>로그인</h1>

<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    
    {% raw %}{{ login_form }}{% endraw %}
    <input type="submit" value="Submit"/>
</form>
{% raw %}{% endblock %}{% endraw %}
```



### 로그아웃

- `django.contrib.auth` 내 `logout` 함수를 import 하여 세션에 저장된 유저 정보를 제거

```python
#views.py
from django.contrib.auth import logout as auth_logout

def logout(request):
    auth_logout(request)
    return redirect('posts:list')

#urls.py
path('logout/', views.logout, name='logout')
```



<br>

## AuthenticationForm에 대한 고찰

- `AuthenticationForm` 은 유저가 존재하는지를 검증하는 모델폼임.
- `AuthenticationForm` 의 생성자의 메서드를 살펴보면, 아래와 같이 정의되어있음. 생성자 메서드의 첫번째 인자는 반드시 request 객체가 들어가야하며, request 객체 명 또는 None값을 적용할 수 있음.
  - 브라우저의 쿠키 support를 받아 쿠키와 세션간의 유효성 검증을 하고 싶은 경우, request 객체를 첫번째 인자로 넣어줌.
  - 유효성 검증을 위한 form data는 키워드 인자 `data=  ` 에 들어감

```python
    def __init__(self, request=None, *args, **kwargs):
        """
        The 'request' parameter is set for custom auth use by subclasses.
        The form data comes in via the standard 'data' kwarg.
        """
        self.request = request
        self.user_cache = None
        super(AuthenticationForm, self).__init__(*args, **kwargs)
```

- 이를 통해 우리가 구현할 수 있는 `AuthenticationForm` 포맷 예제는 다음과 같음

```python
AuthenticationForm(data=request.POST)
AuthenticationForm(request, request.POST)
AuthenticationForm(None, request.POST)
```

- Reference

> http://www.janosgyerik.com/django-authenticationform-gotchas/
> https://docs.djangoproject.com/en/1.8/_modules/django/contrib/auth/forms/
> https://stackoverflow.com/questions/8421200/using-authenticationform-in-django