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

- delete 함수
  -  `posts:list` 
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

    => `variable routing` , 동적으로 설정되는 변수를 두번째 파라미터로 적용하면 됨. 
    => 만약 변수가 여러개 일 경우, 콤마로 구분하여 그 다음 파라미터로 삽입.

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
>
> ```python
> {% raw %}
> {% url '[app_name]:[설정된 name 값]' [동적 변수 명] %}
> {% endraw %}
> ```
>
> html문서 내에서는 동적으로 들어가는 변수에 대해서는 콤마가 아니라 띄워쓰기로 구분을 해줘야함.



- list.html

```html
<-- index.html -->
    
<body>
    <h1>Post Index</h1>
    <a href="/posts/new/">New - 새로운 글쓰기</a>
    <ul>
    {% for post in posts %}
        <li><a href="/posts/{{post.pk}}/">{{ post.title }}</a></li>    
    {% endfor %}
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
{% raw %}
<a href="{% url 'posts:new' %}">New - 새로운 글쓰기</a>
<li><a href="{% url 'posts:detail' post.pk %}/">{{ post.title }}</a></li> 
{% endraw %}
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
{% raw %}
<form action="{% url 'posts:create' %}", method="post">
{% endraw %}
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
{% raw %}
<a href="{% url 'posts:list' %}">List</a>
<a href="{% url 'posts:edit' post.pk %}">Edit</a>
<a href="{% url 'posts:delete' post.pk %}">Delete</a>
{% endraw %}
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
{% raw %}
<form action="{% url 'posts:update' post.pk %}", method="post">
{% endraw %}
```



### Delete (Delete 페이지 구현)

- views.py & urls.py
  - detail 함수를 작성했던 방식과 매우 유사함. 
  - urls 주소의 `post_id` 를 variable routing으로 사용하여, 함수 내 특정 레코드를 갖고오는 id 값으로 사용.
  - `post.delete()`  이용하여 레코드를 삭제 한 후,  list 페이지인 `‘/posts/’` 로 redirect 시킴.

```python
#views.py
def delete(request, post_id):
    post = Post.objects.get(pk=post_id)
    post.delete()
    return redirect('/posts/')

#urls.py
urlpatterns = [
    path('<int:post_id>/delete/', views.delete),
]
```

- detail.html
  - 게시글을 삭제하는데는 추가적인 웹페이지를 작성할 필요가 없음. 
  - 따라서,  레코드의 세부정보를 보여주는 `details.html`에 내용을 구현할 수 있음.
  - `<a href="/posts/{{post.pk}}/delete/">Delete</a>`
  -  id값을 불러오기 위해 템플릿 변수를 사용하여 삭제 기능을 동적으로 구현.

```html
<-- details. html-->
<body>
    <h1>Post Detail</h1>
    <h2>Title : {{post.title}}</h2>
    <p> Content : {{post.content}}</p>
    <a href="/posts/">List</a>
    <a href="/posts/{{post.pk}}/delete/">Delete</a>
</body>    
```



### Update(	Edit & Update 페이지 구현)

> 게시판 글을 수정하기 위해서는 어떠한 레코드를 수정할건지에 대한 정보가 필요하므로 다시 id값을 받아야함.
>
> new & create 페이지에서 우리는 아래와 같이 페이지의 역할을 구분했었음.
>
> - `new.html`: 내용 입력 Form을 작성. 작성된 내용을 `/posts/create/`로 전달하는요청을 보냄
> - `create` 함수: `/posts/create/` 로 url이 접근되면, 함수가 실행되어, 데이터베이스에 내용을 저장시킴.
>
> 마찬가지로, 게시판 글을 수정하는 페이지도 역할을 아래와 같이 2가지로 구분하여 작성해야함.
>
> - `edit.html` : 내용을 수정하는 Form을 작성하는 페이지, 수정된 내용을 `posts/[id값(템플릿변수)]/update`으로 전달하는 요청을 보냄. 
> - `update` 함수 : `posts/[id값(템플릿변수)]/update` url을 사용자가 입력하면 update 함수가 실행됨.



### Edit 페이지 구현

- views.py & urls.py
  -  `기본주소/[id값]/edit/` url으로 서버에 요청을 보내면, urlpattern에 따라 `edit` 함수가 실행됨. 
  - `post_id` 는 variable routing으로 `edit 함수의 두번째 파라미터로 받아옴.
  - 이 값을 가지고 DB의 특정 레코드에 접근. 
  - 레코드 값이 저장된 `post` 변수를 html 파일에서 사용하기 위해 템플릿 변수로 지정.

```python
#views.py
def edit(request, post_id):
    post = Post.objects.get(pk=post_id)
    return render(request, 'edit.html', {'post':post})

#urls.py
urlpatterns = [
    path('<int:post_id>/edit/', views.edit),
]
```



- edit.html
  - input 태그의 내용에는 변경사항을 실어서 `update`  주소로 보냄.
  - 기존에 입력된 내용을 보기 위해, 템플릿 변수로 연결된 post라는 인스턴스의 멤버변수인 `post.title` 와 `post.content` 를  value 속성의 값으로 저장.
  - 이를 통해 edit.html이 실행될 때, input 태그의 value에 기존의 데이터를 볼 수 있게됨.
  - 어떠한 레코드를 수정하는지에 대한 정보가 들어가야하므로, `post.pk`  변수는 `variable routing`으로 사용하여, update주소로 요청을 보냄.
  - 요청방식은 `POST`가 사용되었기 때문에, 추가로 `{% raw %}{% csrf_token %}{% endraw %}` 을 입력.

```html
<body>
    <h1>Post Edit</h1>
    <form action="/posts/{{ post.pk }}/update/", method="POST">
        {% raw %}{% csrf_token %}{% endraw %}
        <input type="text" name="title" value="{{ post.title }}"/>
        <input type="text" name="content" value="{{ post.content }}"/>
        <input type="submit" value="Submit"/>
    </form>
</body>
```



### update 페이지 구현

- views.py & urls.py
  - `variable routing` 인`post_id`를 통해 우리는 수정하고자 하는 레코드를 반환.
  - `Post ` 클래스의 인스턴스 객체인 `post` 의 멤버변수를 edit.html  - form 태그에 작성된 내용으로 업데이트 시킴.
  - 이후 `.save()` 을 통해 DB에 내용을 저장시킴.

```python
#views.py
def update(request, post_id):
    #수정하는 코드
    post = Post.objects.get(pk=post_id)
    post.title =request.POST.get('title')
    post.content =request.POST.get('content')
    post.save()
    return redirect(f'/posts/{post_id}/')

#urls.py
urlpatterns = [
    path('<int:post_id>/update/', views.update),
]
```



> 게시글을 업데이트 하고싶을 때 마다  `/posts/{{post.pk}}/edit/` url주소를 직접 입력하는 방법은 매우 비효율적일 것이다. 따라서, 레코드의 세부정보를 보여주는 detail.html에  게시글을 업데이트하는 내용을 구현하도록 하자.

- detail.html
  - delete 때와 마찬가지로, a 태그와 id값을 템플릿 변수로 적절히 활용하여 edit 함수를 실행시키는 링크를 작성.

```html
<body>
    <h1>Post Detail</h1>
    <h2>Title : {{post.title}}</h2>
    <p> Content : {{post.content}}</p>
    <a href="/posts/">List</a>
    <a href="/posts/{{post.pk}}/edit/">Edit</a>
    <a href="/posts/{{post.pk}}/delete/">Delete</a>
</body>    
```