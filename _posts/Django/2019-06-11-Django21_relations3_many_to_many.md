---
layout: post
title: Django 21 - M:N 관계설정 / 개인 페이지 만들기
category: Django
tags: [Django]
comments: true
---



# M:N 관계 설정 

> 1:N 관계는 하나의 모델(N)에 대하여 하나의 모델만이 연결이 될 수 있음. 그러나 일반적으로 "좋아요" 기능의 경우, 한명의 유저(user)가 여러 게시글(post)에 "좋아요"를 누를 수 있으며, 반대로 하나의 게시글(post)에도 여러명의 유저(user)가 좋아요를 누를 수 있어야함. 따라서 1:N 관계가 아니라 다대다(M:N) 관계 설정이 필요함.

```HTML
POST1 [P1, U1] USER1
POST2 [P2, U2] USER2
	  [P2, U3] USER3
POST3 [P3, U3] USER3
POST4 [P3, U4] USER4
```



### 모델 클래스 내 필드 추가

- M:N 관계를 설정하기 위해 `ManyToManyField`  사용
- 관계되는 모델을 첫번째 인자에 입력. Django 내 내장된 유저모델을 사용하기 위해 `settings.AUTH_USER_MODEL` 을 입력.
- 관계되는 변수명은 `related_name` 속성값으로 표시. 이는 첫번째 인자에서 접근할때 사용하는 변수명이며, 아래와 같이 쓸 수 있음.
  - `Post` => `post.like_users`
  - `User` => `user.like_posts`
- 이후, 마이그레이션 재설정

```python
class Post(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    content = models.TextField()
    image = ProcessedImageField(
                    upload_to= post_image_path, 
                    processors=[ResizeToFill(600,600)], 
                    format='JPEG', 
                    options= {'quality': 90 },
        )
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name='like_posts')
```



### views.py & urls.py 코드 작성(좋아요 함수 구현)

`Post` 모델 클래스에 새로 생성한 `like_users` 필드는 리스트의 형태를 띄고 있음. 이는 M:N 관계에 따라, `like_users` 에 하나의 값만 저장되는게 아니라 여러 값이 저장 될 수 있기 때문임.  이를 활용하여,  동일한 url과 동일한 함수를 사용하여, 조건문 분기를 통해 요청을 보내는 유저가 `like_users ` 리스트에 포함되지 않았을 경우,  리스트에 유저를 추가 한 후, 좋아요 기능이 활성화되고, 반대의 경우는 유저를 리스트에서 제거하여 좋아요 기능을 비활성화 시키면 됨.

- `post.like_users.all()` : 인스턴스 객체인 `post` 내 `like_users` 안의 모든 유저정보를 갖고 옴.
- 조건문을 이용하여 좋아요 기능 활성화/비활성화를 나눔.

```python
#views.py
def like(request, post_id):
    post = get_object_or_404(Post, id=user_id)
    
    if request.user in post.like_users.all():
        #좋아요 취소
        post.like_users.remove(request.user)
    else:
        post.like_users.add(request.user)
    
    return redirect('posts:list')

#urls.py
urlpatterns = [
    path('<int:post_id>/like/', views.like, name='like'),
]
```



### _post.html 수정

[font Awesome](<https://fontawesome.com/>) 은 벡터 기반의 웹폰트 아이콘 제공하는 사이트로, CDN을 활용하여 아이콘을 웹페이지에 삽입 시킬 수 있음.  원하는 아이콘을 찾아서 html 문서 안에 코드를 삽입하면 됨. 좋아요 기능을 나타내기위해 꽉찬 하트와 빈 하트를 각각 사용.
- 조건문을 통해 현재 `user` 가 `posts.like_users` 리스트 안에 있을 경우(좋아요가 이미 된 경우),  좋아요 이모티콘(꽉찬 하트) 표시. 그렇지 않은 경우(좋아요가 되지 않은 경우),  빈 하트 이모티콘 표시
- 추가적으로 `post.like_users.count` 를 통해 좋아요의 갯수를 카운팅 할 수 있음.
- fontAwesome을 사용하기 위한 CDN 코드를 base.html에 삽입하자.

```html
<!--좋아요 기능-->
<a href="{% raw %}{% url 'posts:like' post.id %}{% endraw %}">
  {% raw %}{% if user in post.like_users.all %}{% endraw %}
    <a href="{% raw %}{% url 'posts:like' post.id %}{% endraw %}"><i class="fas fa-heart"></i></a>
  {% raw %}{% else %}{% endraw %}
    <a href="{% raw %}{% url 'posts:like' post.id %}{% endraw %}"><i class="far fa-heart"></i></a>
  {% raw %}{% endif %}{% endraw %}
</a>
<p class="card-text">
    <!--좋아요 갯수 카운트-->
    {% raw %}{{ post.like_users.count }}{% endraw %} 명이 좋아합니다.
</p>
```



<br>

# 개인 페이지 만들기

인스타그램의 개인 계정페이지와 같은 개인 페이지를 만들어보도록 하자. 계정에 대한 페이지이므로, posts가 아닌 accounts 어플리케이션 내 views.py에 함수를 정의하는 것이 적합함.

### views.py 작성

- `get_object_or_404` 사용을 위해 import 
- `get_user_model` : settings 안의 auth의 user 모델을 불러옴. 사용을 하기 위해 import
  - `settings.AUTH_USER_MODEL`은 모델 클래스를 반환하는 것이 아니라 모델 명을 string 형태로 반환하므로, `get_user_model` 대신 사용할 경우,  `ValueError` 가 발생한다.
- 앞의 `username`은 `user`모델의 `username`을 말하며, 뒤의 `username`은 주소로 들어오는 `username`을 말함.

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth import get_user_model

def people(request, username):
    #get_user_model() => User 클래스를 호출함
    people = get_object_or_404(get_user_model(), username = username)
    return render(request, 'accounts/people.html', {'people':people})
```



### urls.py 작성

- urls.py 내  url 경로를 설정할 때, username을 variable routing으로 사용하므로 str 타입으로 작성

```python
urlpatterns = [
    path('people/<str:username>/', views.people, name='people'),
] 
```



### people.html 생성

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>{% raw %}{{ people.username }}{% endraw %}</h1>

{% raw %}{% endblock %}{% endraw %}
```



### _post.html 수정

_post.html 에 각 게시글마다 게시글에 대한 유저정보가 뜨도록 코드를 작성해놓았음 (Django18 - 게시글에 유저정보 표시하기 참조). 유저정보는 `post.user.username` 을 통해 표시되어있으며, 이를 활용하여 개인 페이지로 들어가는 링크와 연결 시킬 수 있음

- Boostrap card를 이용하여 양식을 다듬었음.

```html
<div class="card" style="width: 18rem;">
  <div class="card-header">
    <span> <a href="{% raw %}{% url 'accounts:people' post.user.username %}{% endraw %}">{% raw %}{{ post.user.username }}{% endraw %} </a></span>
  </div>
```

