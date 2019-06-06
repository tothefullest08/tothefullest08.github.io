---
layout: post
title: Django 16 - 사용자 권한 관리
category: Django
tags: [Django]
comments: true
---



# 사용자 권한 관리

### 로그인 정보 및 사용자 인증 관련 링크 연결

로그인 정보 및 사용자 인증 관련 링크(회원가입/로그인/로그아웃)은 어느 페이지를 가던 표시가 되어야하므로, 모든 페이지의 기본이 되는 기본 템플릿(base.html; insta\templates)에 코드를 작성하는 것이 가장 적합하다.

- Django는 기본적으로 `user` 모델을 갖고 있으며,  템플릿 변수로 사용하는 것이 가능하다. 이를 통해 해당 객체의 username을 갖고 올 수 있음.
- 사용자 인증 링크는 a태그로 작성 가능함
- 조건문을 사용하여, 상황에 따라(로그인 유무) 다른 링크가 보여주도록 분기 
  - `user.is_authenticated` : user가 로그인되어있는지 유무를 확인
  - 마찬가지로, 로그인 된 경우에만 새글 쓰기 링크가 보이도록 조건 설정

```html
{% raw %}{% if user.is_authenticated %}{% endraw %}
<h3> {% raw %}{{ user.username }}{% endraw %}님 환영합니다.</h3>
{% raw %}<a href="{% url 'posts:create' %}"><h3>새글 쓰러 가기</h3></a>{% endraw %}
{% raw %}<a href="{% url 'accounts:logout' %}">로그아웃</a>{% endraw %}
{% raw %}{% else %}{% endraw %}
{% raw %}<a href="{% url 'accounts:signup' %}">회원가입</a>{% endraw %}
{% raw %}<a href="{% url 'accounts:login' %}">로그인</a>{% endraw %}
{% raw %}{% endif %}{% endraw %}
```



### 회원 가입 후 자동 로그인 구현

- `auth_login(request, user)`  코드를 추가로 삽입하여 세션에 로그인 정보를 생성 & 저장 시킬 수 있음.

```python
def signup(request):
    if request.method == 'POST':
        signup_form = UserCreationForm(request.POST)
        if signup_form.is_valid():
            #회원가입 폼의 데이터를 DB에 저장하는 코드를 user 라는 변수에 저장
            user = signup_form.save()
            #회원가입 후 자동 로그인 코드
            auth_login(request, user)
            return redirect('posts:list')
    #중략
```



### 로그인 후 회원가입 및 로그인 페이지 접근 막기

기본 템플릿에 `user.is_authenticated`에 따라 선별된 링크가 보이게 분기시켜주었으나, 여전히 주소창에 회원가입 및 로그인 페이지로 이동하는 URL 주소를 입력하여 접근하는 것이 가능하다. 따라서 이러한 접근을 막기위한  추가적인 코드를 작성할 필요가 있음.

- views.py 내 login 및 singup 함수 바로 아래에 추가적인 조건문을 작성
- `request.user.is_authenticated` : 요청된 user가 이미 유효하다면(로그인되어있다면), list 페이지로 바로 redirect 시킴

```python
def login(request):
    if request.user.is_authenticated:
        return redirect('posts:list')
    #중략
    
def signup(request):
    if request.user.is_authenticated:
        return redirect('posts:list')
    #중략
```



### 새글 생성 페이지(new.html) 의 접근(URL 주소) 막기

위의 방식과 마찬가지로,  로그인이 되어있는 경우에만 새글이 보이도록 base.html에서 조건문을 적용하였으나, 주소창을 통한 접근이 가능하다. 회원가입 및 로그인 페이지 접근을 막았던 방법을 동일하게 적용하여, login 페이지로 redirect 시키는 아래의 코드를 사용할 수도 있으나,  Django에서는 `@login_required` 라는 파이썬 데코레이터를 또한 사용할 수 도 있다.

```python
#로그인 후 회원가입 페이지 접근을 막았던 방법
def create(request):
    if not request.user.is_authenticated:
        return redirect('accounts:login')
```

`@login_required` 는 파이썬썬 문법으로 정의된 파이썬 데코레이터이며, 일련의 함수/메서드이다. 이 함수의 파라미터로 아래 함수를 받는 개념이다.

- Django 내부에 함수로 이미 정의 되어있으며, 적용하고자 하는 함수 위에 데코레이터로 사용 가능
- 데코레이터 import 요  `from django.contrib.auth.decorators import login_required`
- 로그인이 되어있지 않은 경우 로그인 페이지로 redirect 시킴
- Django 에 정의된 데코레이터 함수의 기본값은 아래와 같음. 지금까지 우리는 어플리케이션 명과 함수이름을 아래 URL에 맞게 정의해줬기 때문에, 인자값을 따로 넣어주지 않아도 로그인 페이지로 redirect 시킬 수 있음.

```python
login_required(login_url='/accounts/login') 
```

- setttings.py 를 통해 redirect  시키고자 하는 경로를 임의로 지정하여 기본값을 변경 할 수 있음 
  - 예) `LOGIN_URL = '/accounts/login'`

```python
from django.contrib.auth.decorators import login_required

@login_required
def create(request):
    if request.method == 'POST':
        post_form = PostForm(request.POST)
        
        if post_form.is_valid():
            post_form.save()
            return redirect('posts:list')
    
    else:
        post_form = PostForm()
    return render(request, 'posts/form.html', {'post_form': post_form})
```



### 새글 생성 페이지(new.html) 의 접근(URL 주소) 막기 (2)

로그인 하지 않은 상태에서 새글 생성 페이지를 주소창을 통해 접근 할 경우, 로그인 화면으로 redirect 되도록 `@login_required` 데코레이터를 사용해보았다. 이후, 로그인을 하면, 다시 리스트 페이지로 돌아가게되어, 다시 새글쓰기 링크를 선택해야하는 번거로움이 생기게 된다. 따라서 이 경우, 로그인을 하면 새글을 쓰는 창으로 바로 넘어가게 설정을 할 수 있음.

- 위의 case에서 새글 생성 페이지를 주소창을 통해 접근하면 아래와 같이 URL이 변경됨
  - 기본주소/accounts/login/?next=/posts/new
  - 해당 주소로도 새글 생성 페이지로 들어갈 수 있도록 login 함수의 redirect 경로 설정

```python
def login(request):
    if request.user.is_authenticated:
        return redirect('posts:list')
        
    if request.method == 'POST':
        login_form = AuthenticationForm(request, request.POST)
        if login_form.is_valid():
            auth_login(request, login_form.get_user())
            return redirect(request.GET.get('next') or 'posts:list')
        
    else:
        login_form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'login_form' : login_form})
```


