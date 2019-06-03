---
layout: post
title: Django 08 - Template 에서 하드코딩된 URL 을 제거하기
category: Django
tags: [Django]
comments: true
---





# Template 에서 하드코딩된 URL 을 제거하기

> 지금 까지 우리는 url.py - urlpatterns에 각 url 주소가 요청되면, 그에 따른 함수가 실행되게 설정을 하였으며, return되는 `render` 혹은 `redirect` 함수의 인자로 url 주소를 일일이 하드코딩하였음. 
>
> ```PYTHON
> <li><a href="/posts/{{post.pk}}/">{{ post.title }}</a></li>    
> ```
>
> 위와 같이 강력하게 결합되고 하드코딩된 접근방식의 문제점은 하드코딩을 인한 에러 발생율이 높으며(예, `/` 미입력), 수 많은 템플릿을 가진 프로젝트들의 URL을 바꾸는게 어렵다는 것임.  
>
> 그러나 urls.py 내 `path()` 함수에서 인수의 이름을 정의하면 템플릿을 포함한 Django 어디에서나 명확하게 url을 참조 할 수 있음. 이 강력한 기능을 이용하여, 단 하나의 파일만 수정해도 project 내의 모든 URL 패턴을 바꿀 수 있으며, 직관적으로 코드를 읽을 수 있음.



- urls.py
  - `path` 함수의 두번째 인자로 `name` 속성 적용. 
  - `name` 은 특정한 URL 매핑을 위한 고유 식별 ID의 개념임. 우리는 이 이름을 매퍼에 반전시킬 수 있음.
  - 즉, 매퍼가 처리하도록 설계된 리소스를 향하는 URL을 동적으로 생성하기 위해 설정

```python
app_name = 'posts' 

urlpatterns = [
    path('', views.index, name='list'),
    path('new/', views.new, name='new'),    
    path('create/', views.create, name='create'),
    path('<int:post_id>/', views.detail, name='detail'),
    path('<int:post_id>/delete/', views.delete, name='delete'),
    path('<int:post_id>/edit/', views.edit, name='edit'),
    path('<int:post_id>/update/', views.update, name='update'),
]
```



### 네임 스페이스 설정

- 위의 예시에서 우리는  `url.py`의 주인이 `posts` 라는 어플리케이션 이라는 것을 설정 해줘야함.
- url의 name 값을 사용하다 보면, 한 프로젝트 내에 많은 어플리케이션을 생성할 경우, 이름이 중복되는 문제가 발생할 수 있음. 따라서 중복을 방지 하기 위해 `app_name` 이라는 url 네임 스페이스를 사용 해야 함.

```python
#네임 스페이스 예시
app_name = 'posts'
```



### views.py 내 url 매핑 설정

- delete 함수 - `posts:list` 
  - `posts` : urls.py 에서 설정한 네임 스페이스를 의미.
  - `list` : urls.py에서 설정한 list 페이지로 연결하는 `urlpattern`에 대한 `name` 값.
    `path('', views.index, name='list')`

```python
def delete(request, post_id):
    post = Post.objects.get(pk=post_id)
    post.delete()
    
	#변경 전
	return redirect('/posts/')
	#변경 후
    return redirect('posts:list')
```



- create 함수
  - `create` 함수의 경우, 디테일 페이지로 `redirect`  지정해놓았으며, 디테일 페이지는 id값을 `variable routing`으로 설정이 되어있었음. 따라서, 이 작업을 추가로 적용해야함.
  - `variable routing` , 동적으로 설정되는 변수를 두번째 파라미터로 적용하면 됨. 
  - 만약 변수가 여러개 일 경우, 콤마로 구분하여 그 다음 파라미터로 삽입.

```python
def create(request):
    title = request.POST.get('title')
    content = request.POST.get('content')
    #DB Insert
    post = Post(title=title, content=content)
    post.save()
    
    #변경 전
    return redirect(f'/posts/{post.pk}')
    
    #변경 후
    return redirect('posts:detail', post.pk)
```



### html 문서 내 url 매핑 설정

> index.html 문서와 같이,  a태그에 연결되는 url주소를 입력한 경우도 있음.  url 매핑을 설정하는 기본적인 구조는 다음과 같음.
> ```python
> {% raw %}{% url '[app_name]:[설정된 name 값]' [동적 변수 명] %}{% endraw %}
> ```
> html문서 내에서는 동적으로 들어가는 변수에 대해서는 콤마가 아니라 띄워쓰기로 구분을 해줘야함.



- list.html
```html
<-- index.html -->
    
<body>
    <h1>Post Index</h1>
    <a href="/posts/new/">New - 새로운 글쓰기</a>
    <ul>
    {% raw %}{% for post in posts %}{% endraw %}
        <li><a href="/posts/{{post.pk}}/">{{ post.title }}</a></li>    
    {% raw %}{% endfor %}{% endraw %}
    </ul>
</body>
```

- 변경 전
```html
<a href="/posts/new/">New - 새로운 글쓰기</a>
<li><a href="/posts/{{post.pk}}/">{{ post.title }}</a></li>    
```

- 변경 후
```html
{% raw %}<a href="{% url 'posts:new' %}">New - 새로운 글쓰기</a>
<li><a href="{% url 'posts:detail' post.pk %}/">{{ post.title }}</a></li>{% endraw %} 
```



- new.html

```html
<form action="/posts/create/", method="post">
    {% raw %}{% csrf_token %}{% endraw %}
    <input type="text" name="title"/>
    <input type="text" name="content"/>
    <input type="submit" value="Submit"/>
</form>
```

```html
{% raw %}<form action="{% url 'posts:create' %}", method="post">{% endraw %}
```



- detail.html

```html
<h1>Post Detail</h1>
<h2>Title : {{post.title}}</h2>
<p> Content : {{post.content}}</p>
<a href="/posts/">List</a>
<a href="/posts/{{post.pk}}/edit/">Edit</a>
<a href="/posts/{{post.pk}}/delete/">Delete</a>
```

```html
{% raw %}<a href="{% url 'posts:list' %}">List</a>
<a href="{% url 'posts:edit' post.pk %}">Edit</a>
<a href="{% url 'posts:delete' post.pk %}">Delete</a>{% endraw %}
```



- edit.html

```html
<form action="/posts/{{ post.pk }}/update/", method="post">
    {% raw %}{% csrf_token %}{% endraw %}
    <input type="text" name="title" value="{{ post.title }}"/>
    <input type="text" name="content" value="{{ post.content }}"/>
    <input type="submit" value="Submit"/>
</form>
```

```html
{% raw %}<form action="{% url 'posts:update' post.pk %}", method="post">{% endraw %}
```

