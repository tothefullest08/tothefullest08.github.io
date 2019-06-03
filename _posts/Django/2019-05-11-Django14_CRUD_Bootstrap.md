---
layout: post
title: Django 14 - CRUD 구현 및 Django에 Bootstrap 입히기 
category: Django
tags: [Django]
comments: true
---



# 1. 기본 설정

### 가상환경 및 기본설정

- `pyenv virtualenv 3.6.7 06_venv`
- `pyenv local 06_venv`
- `pip install django==2.1.8` : django 버전 설정
- `django-admin startproject insta`
- `python manage.py startapp posts`
- `settings.py`  내 `INSTALLED_APPS`  & `ALLOWED_HOSTS` 설정

```PYTHON
INSTALLED_APPS = [
    'posts',
    #'posts.apps.PostsConfig',]
ALLOWED_HOSTS = [] #기본값 입력
```



###  Model 설정

```python
class Post(models.Model):
    content = models.TextField()
```

- `python manage.py makemigrations` : 마이그레이션 초안 생성
- `python manage.py migrate` : 설정된 스키마를 DB에 실제 적용



### URL 경로 설정

- 프로젝트 내 urls.py 설정
  - 어플리케이션 urls.py와 연결시키기 위해 `include`  import 해옴

```python
from django.urls import path, include

urlpatterns = [
    path('posts/', include('posts.urls')),]
```

- 어플리케이션 내 urls.py 설정
  - `views.py` 내 함수 호출을 위해 import 
  - 네임스페이스 `posts` 설정

```python
from . import views

app_name = 'posts'

urlpatterns = [
    path('create/', views.create, name='create'),
]
```



### 기본 템플릿 설정

- 프로젝트 폴더 /  `templates` 폴더 / `base.html` 
- CDN 설정하기 (Bootstrap - CSS & JSS,  fontawesome)

```html
<body>
    {% raw %}{% block container %}{% endraw %}
    {% raw %}{% endblock %}{% endraw %}
</body>
</html>
```

- Settings.py 내 기본 템플릿 경로 설정

```PYTHON
'DIRS': [os.path.join(BASE_DIR, 'insta', 'templates')],
```



### 모델 폼 작성

- 어플리케이션 폴더 내 forms.py 파일 생성
- 장고 내 forms 및 모델 클래스 `Post`를 import 해옴
- `model`에는 연동하고자 하는 모델 클래스를 저장

```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['content']
```



### MTV에 따라 Create 기능 구현

- urls.py 내 url 경로 설정
- views. py 내 가장 기본적인 템플릿으로 create 함수 생성
- 앱 폴더 내 templates 폴더 생성 후, create.html 생성. 
  - 기본적인 form 태그 양식에 따라 코드 작성



# 2. Django에 Bootstrap 입히기

### django-bootstrap4 설치

- `pip install django-bootstrap4` : 외부 라이브러리 설치
- settings.py 내 INSTALLED_APPS 내에 앱 추가

```python
INSTALLED_APPS = [
    'bootstrap4',
]
```



### create.html 수정

- {% raw %}`{% load bootstrap4 %}` {% endraw %} 을 템플릿 상속을 위한 코드 사이에 넣어줘야 함.
- {% raw %}`{% bootstrap_form 템플릿 변수 %}` {% endraw %}: 부트스트랩을 적용한 템플릿 변수 양식
  - cf) `{{ 템플릿 변수 }}` : 기본 방식

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% load bootstrap4 %}{% endraw %}
{% raw %}{% block container %}{% endraw %}
<h1>New Post</h1>

<form method="post">
    {% raw %}{% csrf_token %}{% endraw %}
     <!-- 기본방식 {{ post_form }}-->
     <!-- 부트스트랩 클래스 이용 -->
    {% raw %}{% bootstrap_form post_form %}{% endraw %}
    <input type="submit" value="Submit"/>
</form>
{% raw %}{% endblock %}{% endraw %}
```



- 버튼 모양 수정

```html
{% raw %}{% buttons %}{% endraw %}
<button type="submit" class="btn btn-primary">Submit</button>
{% raw %}{% endbuttons %}{% endraw %}
```



# 3. CRUD & Model Form 적용

### 1) Create 페이지 수정

- views.py

```python
from django.shortcuts import render, redirect
from .forms import PostForm
from .models import Post

def create(request):
    #POST 방식일 경우
    if request.method == 'POST':
        post_form = PostForm(request.POST)
        
        #모델 폼이 유효 할 경우
        if post_form.is_valid():
            post_form.save()
            return redirect('posts:list')
    
    #Get 방식일 경우
    else:
        post_form = PostForm()
    return render(request, 'posts/create.html', {'post_form': post_form})
```



### 2) list페이지 작성

- views.py

```python
from .models import Post

def list(request):
    posts = Post.objects.all()
    return render(request, 'posts/list.html'. {'posts':posts})
```



- list.html
  -  세부 내용은 _post.html에 작성 
  - {% raw %}`{% include 'posts/_post.html' %}`{% endraw %}

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1> Post List </h1>

<!-- 진자 템플릿 사용 -->
{% raw %}{% for post in posts %}{% endraw %}
<!-- 세부 내용은 _post.html에 작성 -->
{% raw %}{% include 'posts/_post.html' %}{% endraw %}

{% raw %}{% endfor %}{% endraw %}
{% raw %}{% endblock %}{% endraw %}
```



- _post.html
  - 재사용성을 위해 파일을 분리시킴
  - 파일의 제목은 반드시 `_` 로 시작할 필요는 없으나, `_`를 사용하는 것이 통상적인 룰.
  - 어플리케이션 폴더 - 템플릿 폴더 내 `_post.html` 파일 생성 후 코드 작성

```html
<div class="card" style="width: 18rem;">
  <img src="..." class="card-img-top" alt="...">
  <div class="card-body">
    <p class="card-text">{{ post.content }}</p>
  </div>
</div>
```



### 3) Delete

- urls.py

```python
from django.urls import path
from . import views

app_name = 'posts'

urlpatterns = [
    path('<int:post_id>/delete/', views.delete, name='delete'),
]
```



- views.py

```python
def delete(request, post_id):
    #기본키를 뜻하는 pk=post_id를 써도 되고, 아니면 id를 써도됨. 
    post = Post.objects.get(id=post_id)
    post.delete()
    
    return redirect('posts:list')
```



- _post.html

```html
<div class="card" style="width: 18rem;">
    <!-- 장고 템플릿 문법을 사용하여 삭제하는 링크 설정 -->
    {% raw %}<a href="{% url 'posts:delete' post.id %}" class="btn btn-primary">Delete</a>{% endraw %}
  </div>
</div>
```



- get_object_or_404
  - url 주소로 이미 삭제 된 id를 이용하려는 접근할 경우 에러가 발생함.
  - 존재 하지 않는 id 값 기반의 url 주소로 접속할 경우, 에러가 발생하므로 `Movie.objects.get` 대신 `get_object_or_404` 를 사용하여 404 에러 메세지(Page not found)를 표시하게 코드 작성(사용자 친화적)
  - 사용을 위해 import 해올 것. `from django.shortcuts import get_object_or_404`

```python
from django.shortcuts import render, redirect, get_object_or_404

def delete(request, post_id):
    # post = Post.objects.get(id=post_id)
    post = get_object_or_404(Post, id=post_id)
    post.delete()
    return redirect('posts:list')
```



 