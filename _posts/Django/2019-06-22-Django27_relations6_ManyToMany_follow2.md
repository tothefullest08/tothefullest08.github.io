---
layout: post
title: Django 27 - 팔로우 기능, 유저모델(AbstractUser) 상속 (2)
category: Django
tags: [Django]
comments: true
---



# 팔로우 기능, 유저모델(AbstractUser) 상속 (2)

### accounts/views.py - follow 함수 작성

urlpattern을 통해 people.html (개인페이지)에서 팔로우를 누를 경우, follow 함수가 실행되게 코드 작성. 이때, 팔로우를 누르는 사람은 로그인한 유저이며,  people은 follow를 하려는 유저의 정보가 담긴 User의 인스턴스 객체임.

- people.followers은 people을 팔로우 하고 있는 팔로워들이 저장되어있는 리스트임. 
- 조건문을 적용하여,  동일한 urlpattern / a태그에 대해 팔로우 & 언팔로우 가능을 구현할 수 있음. 만약 `request.user` (로그인한 유저)가
  -  리스트에 있는 경우 리스트에 제거 => 언팔로우 기능
  -  리스트에 없는 경우 리스트에 추가 => 팔로우 기능

```python
def follow(request, user_id):
    people = get_object_or_404(get_user_model(), id=user_id)
    if request.user in people.followers.all():
        # people을 unfollow 하기
        people.followers.remove(request.user)
    else:
        # 1. people을 follow 하기
        people.followers.add(request.user)
    
    return redirect('accounts:people', people.username)
```



### people.html

- 개인 페이지(people.html)에 팔로우/언팔로우 기능을 하는 follow 함수를 연결시키는 링크를 연결
  - 지금 로그인된 유저와 people 페이지의 people와 같지 않은 경우에만 해당 링크가 보이게 설정
  - 리스트의 길이를 출력하면 팔로워/팔로잉의 수를 뽑아낼 수 있음.

```html
{% extends 'base.html' %}
{% load bootstrap4 %}
{% block container %}

<div class="container">
    <div class="row">
        <div class="col-3">
            <h1> 
            {% if people.profile.image %}
            <img src="{{ people.profile.image.url }}" width= 70, 
                 alt="{{ people.profile.image}}">
            {% endif %}
            {{ people.username }}
            </h1>
        </div>
        <div class="col-9">
            <div>
                <strong>{{ people.profile.nickname }}</strong>
                {{ people.followers.all }}
                {% if user != people %}
                    {% if user in people.followers.all %}
                    <a href="{% url 'accounts:follow' people.id %}">UnFollow</a>
                    {% else %}
                    <a href="{% url 'accounts:follow' people.id %}">Follow</a>
                    {% endif %}
                {% endif %}
            </div>
            <div>
                <strong> Followers: </strong> {{ people.followers.count }}
                <strong> Followings: </strong> {{ people.followings.count }}
```



### 리스트 페이지에 팔로우한 사람에 대한 게시글만 보이게 하기

- `request.user` (로그인된 유저)가 팔로우한 사람(`request.user.followings`)들의 전체 리스트를 뽑아와 `followings` 에 저장
- `user` 필드를 기준으로 filter 조건을 적용하려하나, `user` 는 필드에는 단 하나의 유저정보(유저의 id값)가 담겨있으므로, 필드값에 `followings ` 을 넣는 것은 부적합하다 (followings에는 여러 유저 id값이 리스트의 형태로 담겨있음) 
- 이때, `user__in` 을 사용하여, 다수의 유저정보를 토대로 filter를 사용하면 됨.

```python
@login_required
def list(request):
    #posts = Post.objects.order_by('-id').all()
    
    # 1. 내가 follow하고 있는 사람들의 리스트
    followings = request.user.followings.all()
    # 2. follow하고 있는 사람들이 작성한 Posts만 뽑아옴.
    posts = Post.objects.filter(user__in=followings).order_by('id')
    comment_form = CommentForm()
    
    return render(request, 'posts/list.html', {'posts':posts, 'comment_form':comment_form})
```



### 리스트 페이지에 본인글 포함하기 (1)

이 경우, followings  변수(본인이 팔로우하고 있는 유저정보가 담긴 리스트)에 나 자신도 넣어야함. 이 리스트는 쿼리셋이라는 데이터 타입인데, 여기에서는 데이터 수정을 하는 것이 어려움. 따라서 데이터 타입을 수정할 필요가 있음

- `values_list` : 쿼리셋을 튜플의 형태로 변경시킴. 인자로 넣는 필드명 순서에 맞게 튜플에 저장이 됨

  - 하나의 필드만 넣을 경우, `flat` 인자를 입력할 수 있음. `flat=True` 인 경우,  결과값을 리스트의 형태처럼 single values로 반환함

  ```python
  >>> Entry.objects.values_list('id', 'headline')
  <QuerySet [(1, 'First entry'), ...]>
  >>> Entry.objects.values_list('id').order_by('id')
  <QuerySet[(1,), (2,), (3,), ...]>
  >>> Entry.objects.values_list('id', flat=True).order_by('id')
  <QuerySet [1, 2, 3, ...]>
  ```

- [공식문서](<https://docs.djangoproject.com/ko/2.1/ref/models/querysets/#values-list>) 참조

- `request.user` (로그인된 유저)가 팔로우 하는 유저의 정보들을 `values_list`를 이용. 이때, `id` 하나를 단일 필드로 입력하고, `flat=True` 값을 입력하여 로그인된 유저가 팔로우하는 유저의 id값을 리스트의 형태로 나타낸 후, `followings` 라는 변수에 저장

- 이후,  변수에 본인의 유저 id 값 또한 append를 사용하여 저장.

```python
@login_required
def list(request):
    
    followings = request.user.followings.values_list('id', flat=True)
    followings.append(request.user.id)
    
    posts = Post.objects.filter(user__in=followings).order_by('id')
    comment_form = CommentForm()
    
    return render(request, 'posts/list.html',
                  {'posts':posts,'comment_form':comment_form})
```

 

> 위의 방법이 권장되나, 게시글 목록을 보여주는 함수의 이름을 list로 정의하는 바람에 values_list를 사용하지 못하는 상황이 발생함. (values_list의 list를 함수명의 list로 인식해버림). 따라서, 대안으로 아래의 방법을 사용할 수 있음.



### 리스트 페이지에 본인글 포함하기 (2)

- `itertools.chain()` 을 사용하여 여러개의 쿼리셋을 하나의 리스트로 합칠 수 있음.
  - chain의 인자에는 iterable가능한 데이터 타입이 들어가야함. request.user은 iterable 하지않으므로 리스트의 형태로 타입을 변경하기 위해 []로 묶어줌.
- 사용을 위해 import

```python
from itertools import chain

@login_required
def list(request):
    
    followings = request.user.followings.all()
    followings = chain(followings, [request.user])
    
    posts = Post.objects.filter(user__in=followings).order_by('-id')
    comment_form = CommentForm()
    
    return render(request, 'posts/list.html', 
                  {'posts':posts, 'comment_form': comment_form})
```



### 전체 리스트 페이지 구현

views.py 내 list 함수를 통해 본인이 팔로우한 사람들의 게시글 + 자신의 게시글 목록을 보여주는 페이지를 구현하였음. 팔로우를 하지 않은 사람의 경우, id값을 알지 못하는 이상 팔로우를 할 수 없으므로, 팔로우 유무와 관계 없이 모든 글이 뜨는 또다른 리스트 페이지를 작성해보자.

- 기본 MTV 패턴 구현
- explore 함수가 실행되는 링크를 base.html 삽입
- html은 list.html을 그대로 활용

```python
#views.py
def explore(request):
    posts = Post.objects.order_by('id').all()
    comment_form = CommentForm()
    return render(request, 'posts/list.html', 
                  {'posts':posts, 'comment_form':comment_form})

#urls.py
urlpatterns = [
    path('explore/', views.explore, name='explore'),
]
```

- base.html

```html
<a href="{% url 'posts:explore' %}">Explore</a>
```
