---
layout: post
title: Django 10 - REST API
category: Django
tags: [Django]
comments: true
---





# REST API

> HTTP URI(Uniform Resource Identifier)를 통해 자원(Resource)을 명시하고, HTTP Method(POST, GET, PUT, DELETE)를 통해 해당 자원에 대한 CRUD Operation을 적용하는 것을 의미

### Reference

> 본 포스팅은 아래 링크의 내용을 참조하여 작성하였습니다.
>
> https://meetup.toast.com/posts/92
> <https://ko.wikipedia.org/wiki/REST>
> [http://blog.remotty.com/blog/2014/01/28/lets-study-rest/](https://blog.remotty.com/blog/2014/01/28/lets-study-rest/)
> <https://yangbongsoo.gitbooks.io/study/content/restc758_c774_d574_c640_c124_acc4.html>
> http://spoqa.github.io/2012/02/27/rest-introduction.html](https://spoqa.github.io/2012/02/27/rest-introduction.html)



### REST API의 탄생

REST는 Representational State Transfer라는 용어의 약자로서 2000년도에 로이 필딩 (Roy Fielding)의 박사학위 논문에서 최초로 소개되었습니다. 로이 필딩은 HTTP의 주요 저자 중 한 사람으로 그 당시 웹(HTTP) 설계의 우수성에 비해 제대로 사용되어지지 못하는 모습에 안타까워하며 웹의 장점을 최대한 활용할 수 있는 아키텍처로써 REST를 발표했다고 합니다.



### REST 구성

- **자원(RESOURCE)** - URI

  모든 자원에 고유한 ID가 존재하고, 이 자원은 Server에 존재한다.
  자원을 구별하는 ID는 ‘/groups/:group_id’와 같은 HTTP URI 다.
  Client는 URI를 이용해서 자원을 지정하고 해당 자원의 상태(정보)에 대한 조작을 Server에 요청한다.

- **행위(Verb)** - HTTP METHOD

  HTTP 프로토콜의 Method를 사용한다.
  HTTP 프로토콜은 GET, POST, PUT, DELETE 와 같은 메서드를 제공한다.

- **표현(Representations)**

  Client가 자원의 상태(정보)에 대한 조작을 요청하면 Server는 이에 적절한 응답(Representation)을 보낸다.
  REST에서 하나의 자원은 JSON, XML, TEXT, RSS 등 여러 형태의 Representation으로 나타내어 질 수 있다.
  JSON 혹은 XML를 통해 데이터를 주고 받는 것이 일반적이다.



### REST API 디자인 가이드

REST API 설계 시 가장 중요한 항목은 다음의 2가지로 요약할 수 있습니다.

**첫 번째,** URI는 정보의 자원을 표현해야 한다.
**두 번째,** 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE)로 표현한다.

다른 것은 다 잊어도 위 내용은 꼭 기억하시길 바랍니다.



### REST API 중심 규칙

**1) URI는 정보의 자원을 표현해야 한다. (리소스명은 동사보다는 명사를 사용)**

```
    GET /members/delete/1
```

위와 같은 방식은 REST를 제대로 적용하지 않은 URI입니다. URI는 자원을 표현하는데 중점을 두어야 합니다. delete와 같은 행위에 대한 표현이 들어가서는 안됩니다.



**2) 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE 등)로 표현**

위의 잘못 된 URI를 HTTP Method를 통해 수정해 보면

```
    DELETE /members/1
```

으로 수정할 수 있겠습니다.
회원정보를 가져올 때는 GET, 회원 추가 시의 행위를 표현하고자 할 때는 POST METHOD를 사용하여 표현합니다.

**회원정보를 가져오는 URI**

```
    GET /members/show/1     (x)
    GET /members/1          (o)
```

**회원을 추가할 때**

```
    GET /members/insert/2 (x)  - GET 메서드는 리소스 생성에 맞지 않습니다.
    POST /members/2       (o)
```

**예시**

```
	GET /movies/show/1 (x)
	GET /movies/1      (o)
```

```
	GET /movies/create (x) - GET METHOD는 자원 생성에 부적합
	POST /movies   	   (o) 
```

```
	GET /movies/2/update (x) - GET 부적합
	PUT /movies/2		 (O) 
```



### HTTP METHOD의 알맞은 역할

POST, GET, PUT, DELETE 이 4가지의 Method를 가지고 CRUD를 구현 할 수 있습니다.

| METHOD | 역할                                                         |
| ------ | ------------------------------------------------------------ |
| POST   | POST를 통해 해당 URI를 요청하면 리소스를 생성합니다.         |
| GET    | GET를 통해 해당 리소스를 조회합니다. 리소스를 조회하고 해당 도큐먼트에 대한 자세한 정보를 가져온다. |
| PUT    | PUT를 통해 해당 리소스를 수정합니다.                         |
| DELETE | DELETE를 통해 리소스를 삭제합니다.                           |

다음과 같은 식으로 URI는 자원을 표현하는 데에 집중하고 행위에 대한 정의는 HTTP METHOD를 통해 하는 것이 REST한 API를 설계하는 중심 규칙입니다.

그러나, 장고에서는 PUT, DELETE  HTTP method는 지원하지 않음.  해당 메소드는  Rest Architecture을 표현하기 위해 근래에 생성된 메소드임. 장고에서는 해당 메소드들을 사용 할 수 없기 때문에 절충안으로 아래의 방법을 사용함. 

```
GET   /movies/2/edit		- 수정 페이지 보여줌
POST  /movies/2/edit		- 수정 작업을 수행함
```

행위에 따라서 서로 다른  방식을 표현함.
그 외, rest의 권장 방법으로는 뒤에 슬래쉬를 넣지 않으나, 장고에서는 슬래쉬를 넣어야함.



### urls.py 수정하기

- 리스트 페이지는 GET 방식으로 모든 포스트를 다가져오므로 그대로 유지
- 게시글 생성의 경우,  
  - `‘new/’ `는  `GET`을 통해 해당 리소스(URI)를 조회함. 해당 다큐멘트에 대한 자세한 정보(새글을 작성하는 폼)를 가져옴. (새로운 페이지를 가져옴. `GET`에 적합)
  - `‘create/’`의 경우, 새글을 작성(새로운 리소스 생성)하므로 `POST`방식에 맞음. 그런데, create라는 동사를 URL에 포함시키는것은 `REST Architecture`에 적합하지 않으므로 수정이 필요
  -  `'new/' `   `URI` 를 기반으로, `GET/POST` 방식에 따라, 다른 코드가 적용되게 views.py  내용을 수정
  - 어떠한 요청의 방식이  `POST`이면, `CREATE `안의 코드를 실행, 그렇지 않으면(`GET`일 경우), 새글 작성하는 폼이 있는 페이지를 보여주게끔, 구현 가능. 
  - `create/` 함수는 삭제 가능
- 동일한 방법으로 edit & update 또한 수정이 가능함
  - 어떠한 요청의 방식이  `POST`이면, `update`안의 코드를 실행, 그렇지 않으면(`GET`일 경우), 글을 수정하는 폼이 있는 페이지를 보여주게끔, 구현 가능. 

```python
urlpatterns = [
    #GET
    path('', views.index, name='list'), 
    
    #GET(new), POST(create)
    path('new/', views.new, name='new'), 
    #path('create/', views.create, name='create'),
    
    #GET
    path('<int:post_id>/', views.detail, name='detail'), 
    
    #GET(conifrm) POST(delete)
    path('<int:post_id>/delete/', views.delete, name='delete'), 
   
    #GET(edit), POST(update)
    path('<int:post_id>/edit/', views.edit, name='edit'), 
    #path('<int:post_id>/update/', views.update, name='update'),
]
```



### New/Create 수정

>  `'new/' `   `URI` (리소스)를 기반으로, `GET/POST` 방식에 따라, 다른 코드가 적용되게 views.py  내용을 수정.
>
> 어떠한  요청의 방식이  `POST`이면, `CREATE `안의 코드를 실행, 그렇지 않으면(`GET`일 경우), 새글 작성하는 폼이 있는 페이지를 보여주게 코드 수정.

- views.py
  - 조건문을 통해  코드를 분기하여 작성 가능.`request.method` 는 http Method를 의미함. 
  - create 함수는 삭제 가능

```python
def new(request):
    if request.method == 'POST':
        #create
        title = request.POST.get('title')
        content = request.POST.get('content')
        image = request.FILES.get('image')
        
        post = Post(title=title, content=content, image=image)
        post.save()
    
        return redirect('posts:detail', post.pk)
    
    else:
        #new
        return render(request, 'new.html')
```



- new.html 

  - 동일한 URI를 사용하므로 create 함수를 실행시키는 URL를 작성하였던 action 속성을 삭제
    - cf) `action="{% url 'posts:create' %}"`
  - `form` 태그에 `action` 속성값이 존재 하지 않을 경우, 자기자신에게 요청을 보냄.
  - 이에 따라, views.py - new 함수 내 조건문에 따라 POST 방식에 대한 코드가 실행됨.

```html
{% raw %}
{% extends 'base.html' %}
{% block container %}
    <form method="POST">
        {% csrf_token %}
        <input type="text" name="title"/>
        <input type="text" name="content"/>
        <input type="submit" value="Submit"/>
    </form>
{% endblock %}
{% endraw %}
```



- 기본 프로세스
  - `기본주소/posts/new` 의 리소스(URI)가 요청이 들어오면, `GET` 방식을 통해 `new.html` 페이지가 열림. 
    (`new` 함수 조건문에서  `else`가 실행됨)
  - 내용을 작성한 후, `submit` 을 할 경우, 자기자신에 `POST` 방식으로 요청을 보내게 되며, create 함수의 코드에 따라, 데이터베이스에 내용이 저장되고, `detail.html`  으로 `redirect ` 됨.
    (`new` 함수 조건문에서 `if` 부분이 실행됨)



### Edit & Update 수정

> 같은 방법으로,  `edit` & `update.html`에 대한 `urlpattern` & `views.py` 내용도 수정 가능.

- views.py

```python
#views.py
def edit(request, post_id):
    post = Post.objects.get(pk=post_id)
    
    #update
    if request.method == 'POST':
        post.title = request.POST.get('title')
        post.content = request.POST.get('content')
        post.save()    
        return redirect('posts:detail', post.pk)
    
    else:
        #edit
        return render(request, 'edit.html', {'post':post})
```



- edit.html

```html
{% raw %}
<-- edit.html -->
{% extends 'base.html' %}
{% block container %}
    <h1>Post Edit</h1>
    <form method="post">
        {% csrf_token %}
        <input type="text" name="title" value="{{ post.title }}"/>
        <input type="text" name="content" value="{{ post.content }}"/>
        <input type="submit" value="Submit"/>
    </form>
{% endblock %}
{% endraw %}
```



### Delete.html 생성 

> 장고에는 HTTP Method 중 하나인 `delete`를 지원하지 않기 때문에, `GET`과``POST`  메서드를 통해 대안으로 구현할 수 있음.
>
> - GET: 삭제에 대한 Confirm  유무를 보여주는 URI(리소스)를 조회함
> - POST:  Delete 실행



- urls.py

```python
#GET(conifrm) POST(delete)
path('<int:post_id>/delete/', views.delete, name='delete'), 
```



- views.py

```python
def delete(request, post_id):
    if request.method == 'POST':
        post = Post.objects.get(pk=post_id)
        post.delete()
        return redirect('posts:list')
    
    else:
        return render(request, 'delete.html')
```



- delete.html

```html
{% raw %}
<-- delete.html -->
{% extends 'base.html' %}
{% block container %}

<h1>정말로 삭제하겠읍니까?</h1>
<form method="post">
    {% csrf_token %}
    <input type="submit" value="삭제"/>
</form>

{% endblock %}
{% endraw %}
```