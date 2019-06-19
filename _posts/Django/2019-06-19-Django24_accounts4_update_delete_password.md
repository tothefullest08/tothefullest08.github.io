---
layout: post
title: Django 24 - 유저 정보 수정 & 삭제
category: Django
tags: [Django]
comments: true
---



# 유저 정보 수정 & 삭제

## 유저 정보 수정하기

### 기본 MTV 패턴 구현 

- User에 대한 정보를 수정하므로 accounts 앱 내에서 MTV 패턴을 구현
- Django 내장 모델폼인 `UserChangeForm` 사용을 위해 import 
- posts/views.py/update함수와 마찬가지로, 유저정보에 대한 수정이므로 instance 값에 `request.user` 을 입력하여, 유저 정보가 담긴 모델폼을 변수에 저장한 후 템플릿 변수로 뿌림.

```python
# views.py
from django.contrib.auth.forms import UserChangeForm

def update(request):
    user_change_form = UserChangeForm(instance = request.user)
    return render(request, 'accounts/update.html', {'user_change_form':user_change_form})

# urls.py
urlpatterns = [
    path('update/', views.update, name='update'),
]
```
- update 함수로 넘겨진  `user_change_form` 을 bootstrap 양식에 맞춰서 html에 보여줌

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% load bootstrap4 %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>User Edit</h1>

<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    
    {% raw %}{% bootstrap_form user_change_form %}{% endraw %}
    <input type="submit" value="Submit"/>
</form>

{% raw %}{% endblock %}{% endraw %}
```



### forms.py

장고 내장 모델폼인 `UserChangeForm`에는 사용자가 업데이트할 필요가 없는 정보들도 함께 많이 담겨 있음. 따라서 많은 데이터 중, Username, Firs tname, Last Name, Email Address에 대한 정보만 수정하게끔 코드를 다듬어보자.

- accounts/forms.py 파일 생성
  - `UserChangeForm`을 상속한 `CustomUserChangeForm`을 만들어 필요한 필드만 가져옴
  - `UserChangeForm` 사용을 위해 import
- 모델폼에서 연결해주는 모델의 경우 `User()` 이라고 설정해도 되나, global settiings 오버라이딩을 통해서 인증 User 모델을 다른 모델로 변경할 수도 있음. 따라서 settings.py에서 설정된 User 모델을 갖고오는 `get_user_model()` 을 이용하여 사용하자.
  - `get_user_model()` 사용을 위해 import

```python
from django.contrib.auth.forms import UserChangeForm
from django.contrib.auth import get_user_model

class CustomUserChangeForm(UserChangeForm):
    class Meta:
        model = get_user_model()
        fields = ['username', 'email', 'first_name','last_name']
```



### views.py 재 수정

- views.py 내 유저정보수정하는 모델폼을 forms.py에서 새로 정의한 `CustomChangeForm` 으로 변경
  - 사용을 위해 `CustomUserChangeForm` import 해올 것
- POST 방식인 경우, 수정한 내용으로 업데이트를 하고, GET 방식인 경우, 기존의 데이터를 instance에 담은 채, 폼을 html으로 넘김
- 로그인이 된 경우에만 해당 함수가 실행될 수 있도록 데코레이터 설정

```python
from .forms import CustomUserChangeForm
from django.contrib.auth.decorators import login_required

@login_required
def update(request):
    if request.method == 'POST':
        user_change_from = CustomUserChangeForm(request.POST, instance=request.user)
        if user_change_form.is_valid():
            user_change_form.save()
            return redirect('accounts:people', request.user.username)
    
    else:
	    user_change_form = CustomUserChangeForm(instance = request.user)
	return render(request, 'accounts/update.html', {
        						'user_change_form':user_change_form
    								})
```



### people.html

- 유저정보를 수정하는 링크 추가
- 현재 로그인된 `user`가 `people`과 동일한 경우에만 유저 정보 수정 링크가 보이도록 조건문 적용

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<div class="container">
    <h1> {% raw %}{{ people.username }}{% endraw %}</h1>
    {% raw %}{% if user == people %}{% endraw %}
    <div>
    <a href="{% raw %}{% url 'accounts:update' %}{% endraw %}">계정정보수정</a>
    </div>
    {% raw %}{% endif %}{% endraw %}
    <div class="row">
        {% raw %}{% for post in people.post_set.all %}{% endraw %}
        <div class="col-4">
            <img src= "{% raw %}{{ post.image_set.first.file.url }}{% endraw %}" class="img-fluid"/>
        </div>
        {% raw %}{% endfor %}{% endraw %}
    </div>
</div>

{% raw %}{% endblock %}{% endraw %}
```



## 유저 정보 삭제하기 (회원 탈퇴)

### 기본 MTV 패턴 구현

- POST 방식인 경우, DB에서 유저 정보를 삭제하고, GET 방식인 경우, 삭제 페이지로 넘김
- 로그인이 된 경우에만 해당 함수가 실행 될 수 있또록 데코레이터 설정

```python
# views.py
from django.contrib.auth.decorators import login_reqiured

@login_required
def delete(request):
    if request.method == 'POST':
        request.user.delete()
        return redirect('posts:list')
    return render(request, 'accounts/delete.html')


# urls.py
urlpatterns = [
    path('delete', views.delete, name='delete'),
]
```



- delete.html

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>User delete</h1>
<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    <p>정말로 탈퇴하시겠습니까?</p>
    <input type="submit" value="탈퇴"/>
</form>
{% raw %}{% endblock %}{% endraw %}
```



- update.html
  - 유저정보 삭제 링크를 update.html에 표기

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% load bootstrap4 %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>User Edit</h1>

<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    
    {% raw %}{% bootstrap_form user_change_form %}{% endraw %}
    <input type="submit" value="Submit"/>
</form>

<a href="{% raw %}{% url 'accounts:delete' %}{% endraw %}" class="btn btn-danger">회원탈퇴</a>

{% raw %}{% endblock %}{% endraw %}
```



## 비밀번호 변경하기

accounts/update.html에서 사용하는 `UserChangeForm` 으로는  비밀번호 수정이 불가능함.  비밀번호는 암호화가 되어 DB에 저장되므로, 단순히 User edit form에서 알아볼 수 있는 문자로는 수정이 불가능하며 암호화 과정이 필요함. 따라서 비밀번호 변경을 위한 별도의 폼을 사용해야한다.



### views.py & urls.py

- 비밀번호 수정을 위한 Django 내장 폼인 `PasswordChangeForm` 을 사용 & import
- 앞서 반복했던 방식과  마찬가지로 POST 방식인 경우, 수정된 비밀번호로 변경시키고, GET 방식인 경우에는 비밀번호 변경 폼을 password.html에 뿌려줌
  - 해당 폼의 경우 들어오는 인자의 순서가 조금 다름. 
  - 첫번째 인자: request.user (유저에 대한 정보)
  - 두번째 인자: request.POST (비밀번호에 대한 정보)
  - 혹은 키워드 인자명을 함께 써줘도 됨.

```python
# views.py
from django.contrib.auth.forms import PasswordChangeForm

@login_required
def password(request):
    if request.method == 'POST':
        password_change_form = PasswordChangeForm(request.user, request.POST)
        
        # 키워드인자명을 함께 써줘도 가능
        # password_change_form = PasswordChangeForm(user=request.user, data=request.POST)
        if password_change_form.is_valid():
            password_cchange_form.save()
            return redirect('accounts:people', request.user.username)
    
    else:
        password_change_form = PasswordChangeForm(request.user)
    return render(request, 'accounts/password.html',{
        						'password_change_form':password_change_form
    										})'


# urls.py
urlpatterns = [
    path('password/', views.password, name='password'),]           
```



### password.html & update.html

- password.html

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>User password change</h1>
<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    {% raw %}{{ password_change_form }}{% endraw %}
    <input type="submit" value="submit"/>
</form>
{% raw %}{% endblock %}{% endraw %}
```



- update.html
  - 비밀번호 변경 링크를 update.html에 표기

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% load bootstrap4 %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>User Edit</h1>

<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    
    {% raw %}{% bootstrap_form user_change_form %}{% endraw %}
    <input type="submit" value="Submit"/>
</form>

<a href="{% raw %}{% url 'accounts:delete' %}{% endraw %}" class="btn btn-danger">회원탈퇴</a>
<a href="{% raw %}{% url 'accounts:password' %}{% endraw %}">비밀밀호 변경</a>

{% raw %}{% endblock %}{% endraw %}
```



### views.py (2)

세션에는 비밀번호를 포함하여 유저의 정보가 담겨있음. 비밀번호가 바뀔경우, 기존의 세션에 담긴 유저의 비밀번호와 일치하지 않게 되고, 세션이 만료되어 로그인이 풀리게 됨. 따라서 비밀번호를 변경하고도 로그인을 유지하게 하기 위해서는 추가적인 코드 작성이 필요함.

- `update_session_auth_hash(request, user)` 
  - 세션에 있는 로그인 정보 해쉬를 업데이트할 때 사용하는 메서드
  - 새로 설정된 비밀번호의 정보는 `user` 에 담겨있음. 
  - `reqeuest` 인자를 통해 session에 정보를 업데이트 
  - 사용을 위해 import 해올 것
  - [공식문서](<https://docs.djangoproject.com/en/2.2/topics/auth/default/#django.contrib.auth.update_session_auth_hash>) 참조 

```python
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib.auth import update_session_auth_hash

@login_required
def password(request):
    if request.method == 'POST':
        password_change_form = PasswordChangeForm(request.user, request.POST)

        if password_change_form.is_valid():
            # 추가된 부분
            user = password_cchange_form.save()
            update_session_auth_hash(request, user)
            # 끝 
            return redirect('accounts:people', request.user.username)
    
    else:
        password_change_form = PasswordChangeForm(request.user)
    return render(request, 'accounts/password.html',{
        						'password_change_form':password_change_form
    										})'
```

