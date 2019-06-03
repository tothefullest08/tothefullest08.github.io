---
layout: post
title: Django 10 - 템플릿 확장 및 상속
category: Django
tags: [Django]
comments: true
---





# 템플릿 확장 및 상속

> templates 내 html 파일을 살펴보면, 많은 부분 (특히, head)의 코드가 중복되고 있음을 알 수 있음.
>
> 템플릿 확장(template extending)을 통해 웹사이트 안의 서로 다른 페이지에서 HTML의 일부를 동일하게 재사용 할 수 있게 됨. 이 방법을 사용하면 동일한 정보/레이아웃을 사용하고자 할 때, 모든 파일마다 같은 내용을 반복해서 입력 할 필요가 없게 됨.

### 기본 템플릿 생성

- 기본 템플릿은 웹사이트 내 모든 페이지에 확장되어 사용되는 가장 기본적인 템플릿임.
- 프로젝트 폴더(이하, crud) 안에  templates 폴더 생성  - base. html 생성
- 동적으로 바뀌는 부분을 아래의 양식 안에 정의
  - {% raw %}{% block container %}{% endraw %}
  - {% raw %}{% endblock %}{% endraw %}

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1>여기는 base.html!</h1>
    <!--동적으로 바뀔 부분을 아래에 정의-->
    {% raw %}{% block container %}{% endraw %}
    {% raw %}{% endblock %}{% endraw %}
</body>
</html>
```

- settings.py 내 기본 템플릿 경로 설정
- `'DIRS': [os.path.join(BASE_DIR, 'crud','templates')],`
  - `BASE_DIR` : 최상위 ROOT 디렉토리를 의미함. 
  - 2번째 이하 인자로는 ROOT 디렉토리 이하 기본 템플릿이 위치한 경로를 입력
  - 위의 의미는 다음과 같음; `기본주소\crud\templates` 에 기본 템플릿이 위치하고 있음.

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'crud','templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```



### 각 html 파일에 base.html 상속 받기

- base.html을 기본으로 깔고, {% raw %}`{% block container %}/{% endblock %} ` {% endraw %}`{사이에 내용을넣으면 됨.
- 기본 템플릿을 상속받기 위해 {% raw %}`{% extends 'base.html' %}` {% endraw %}을 우선 입력.

1. index.html

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}
    <h1>Post Index</h1>
    {% raw %}<a href="{% url 'posts:new' %}">New - 새로운 글쓰기</a>{% endraw %}
    <ul>
    {% raw %}{% for post in posts %}
        <li>
        {% raw %}<a href="{% url 'posts:detail' post.pk %}">{{ post.title }}
        {% endraw %}    
        </a></li>    
    {% endfor %}{% endraw %}
    </ul>
{% raw %}{% endblock %}{% endraw %}
```



2. detail.html

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

    <h1>Post Detail</h1>
    <h2>Title : {{post.title}}</h2>
    <p> Content : {{post.content}}</p>
	
    <a href="{% raw %}{% url 'posts:list' %}{% endraw %}">List</a>
    <a href="{% raw %}{% url 'posts:edit' post.pk %}{% endraw %}">Edit</a>
    <a href="{% raw %}{% url 'posts:delete' post.pk %}{% endraw %}">Delete</a>
	
    
    <hr>
	{% raw %}<form action="{% url 'posts:comments_create' post.pk %}" method="post">{% endraw %}
        {% raw %}{% csrf_token %}{% endraw %}
        댓글 : <input type="text" name="content"/>
			   <input type="submit" value="Submit"/>
    </form>
    <ul>
      {% raw %}{% for comment in post.comment_set.all %}
      <li> 
       {{ comment.content }} - 
        <a href="{% raw %}{% url 'posts:comments_delete' post.pk comment.pk %}{% endraw %}">Delete</a>
      </li>
      {% endfor %}{% endraw %}
    </ul>

{% raw %}{% endblock %}{% endraw %}
```



3. edit.html

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}
    <h1>Post Edit</h1>
    <form method="post">
        {% raw %}{% csrf_token %}{% endraw %}
        <input type="text" name="title" value="{{ post.title }}"/>
        <input type="text" name="content" value="{{ post.content }}"/>
        <input type="submit" value="Submit"/>
    </form>
{% raw %}{% endblock %}{% endraw %}
```



4. new.html

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}
    <form method="post" enctype="multipart/form-data">
        {% raw %}{% csrf_token %}{% endraw %}
        <input type="text" name="title"/>
        <input type="text" name="content"/>
        <input type="file" name="image" accept="image/*"/>
        <input type="submit" value="Submit"/>
    </form>
{% raw %}{% endblock %}{% endraw %}
```



