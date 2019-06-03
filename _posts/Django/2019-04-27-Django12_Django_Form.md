---
layout: post
title: Django 12 - Django Form
category: Django
tags: [Django]
comments: true
---



# Django Form

> Django 를 사용하기 위한 기본적인 설정하기
>
> 가상환경 설정 / 장고 설치 / 프로젝트 & 어플리케이션 생성 / 
> settings.py 설정 / urls.py 설정 / models.py 설정 / migrate 설정



- 모델 클래스 생성
  - 마이그레이션 적용

```python
class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```



### Create 

- views.py 내 `create` 함수 생성
  - `pass` 는 비어있는 코드로 아무런 역할을 하지 않음 (에러를 피하기 위해 사용)
  - 다른 언어(Java, C)의 경우,  조건문을 비워둬도 되나, python에서는 에러가 발생함

```python
def create(request):
    if request.method == 'POST':
        pass
    else:
        return render(request, 'create.html')
```



- url 경로 설정

```python
#urls.py
from . import views

app_name = 'articles'

urlpatterns = [
    path('new/', views.create, name="create"),
]
```



- `Form` 태그를 사용한 `create.html ` 코드 작성
  - title 과 content의 내용이 반드시 들어가야하는 조건을 적용하기 위해 `required`  속성을 입력

```html
<!-- create.html --> 

<h1>New Aricle </h1>
<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    <input type="text" name="title" required/>
    <input type="text" name="content" required/>
    <input type="submit" value="Submit"/>
</form>
```



- views.py 내 `create` 함수 재설정
  - `Article.objects.create(title=title, content=content)  ` :  데이터베이스에  input값 저장

```python
from django.shortcuts import render, redirect
from .models import Article

def create(request):
    if request.method == 'POST':
        title = request.POST.get('title')
        content = request.POST.get('content')
        article = Article.objects.create(title=title, content=content)  
        # 위의 코드는 아래의 코드와 동일함.
        # article = Article(title=title, content=content)
        # article.save()
        
        return redirect('articles:detail', article.pk)
    else:
        return render(request, 'create.html')
```



### Detail 

- `detail`  함수 작성

```python
def detail(request, article_id):
    article = Article.objects.get(pk=article_id)
    return render(request, 'detail.html' , {'article':article} )
```



- url 경로 설정

```python
path('<int:article_id>/', views.detail, name="detail")
```



- detail.html 생성

```html
<h1>article detail</h1>
<p> title: {{article.title}}</p>
<p> content: {{article.content}}</p>
```



<br>

# Form 작성하기

> `Form` 태그를 사용할 경우,  `input` 값(제목, 내용 등)을 입력하지 않고 submit 할 경우, Error 가 발생함.  또한, 이 필드가 반드시 들어가야하는 필드라면, submit 입력이 막혀있어야함. 
>
> - `required`를 넣으면, submit 입력을 막을 수 있음.
>
> 그러나,  개발자도구에서 HTML 코드를 직접 수정하여, 만약 `required`를 지워서 제출할 경우, 개발자가 설정한 방어책이 뚫릴 수 있게 된다.  따라서, 장고 서버 내부에서 검증을 해줘야므로,  `Form` 태그가 아닌 `Form`  class를 사용하여, 유효성 검증을 하거나, input을 자동으로 만드는 역할을 수행 할 수 있음.



### forms.py 작성

- 어플리케이션(articles) 내 생성
- `ArticleForm`  클래스 생성
  - Django에서 구현해놓은 `form`  라이브러리를 `import` 해서 `ArticleForm`에 상속 시킴.
  - 상속받은 자식 클래스 `ArticleForm` 가 `form` 라이브러리 내 메소드 및 설정들을 사용할 수 있게 됨.
    (models.py 에서 Django DB  내 models.Model을 상속해 와서, 기본설정만 하면 되는 것만 같은 맥락)
- `Form` 클래스 내에서는 `CharField` 의 경우 `max_length`가 필수 인자가 아님.

```python
from django import forms
class ArticleForm(forms.Form):
    title = forms.CharField(label="제목")
    content = forms.CharField(label="내용", widget=forms.Textarea(attrs={
        'rows':5,
        'cols':50,
        'placeholder':'내용을 입력하세요',
    }))
```

- `Form`  & `input` 태그의 속성값을 적용하여 추가적인 설정을 했던 것을  `Form` 클래스에서 설정 할 수 있음.따라서,  input에 대한 추가적인 설정을 HTML 문서가 아니라 Forms.py에서 적용
  - `label` : 기본적으로 input의 이름(html 문서에 보여지는 제목) 을 커스터마이징
  - `widget` : 속성값을 줄 때 사용

- `content` 필드의 경우, 

  1) 모델 클래스 `Article` : `TextField`  사용

  2) 폼 클래스 `ArticleForm` : `CharField` 사용함 (폼에서는 `TextField`가 없음) => `widget` 사용

  - `widget` 의 속성을  `forms.Textarea` 문법을 사용하여  textarea로 변경
  - textarea의 속성은 괄호안에 사용 `attrs={}` . 일반적으로 textarea 태그 내 작성하는 속성을  `attrs` 라는 딕셔너리로 넘겨주면 속성을 적용할 수 있음.



### View & Tempate 수정하기

- views.py

```python
#ArticleForm 을 사용하기 위해 import 해옴
from .forms import ArticleForm 

def create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        
        if form.is_valid():
            title = form.cleaned_data.get('title')
            content = form.cleaned_data.get('content')
            
            article = Article.objects.create(title=title, content=content)  
            
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm()
    return render(request, 'create.html', {'form' : form})
```

- 조건문 else 수정 (GET  요청인 경우) => 기본 폼 생성
  - Forms.py 내 `ArticleForm`  인스턴스를 생성하여 `form`  변수에 저장
  - `form` 을 템플릿 변수로 create.html으로 넘겨줌. 
  - 이를 통해,  create.html에서는  `ArticleForm` 으로 형성된 input 양식을 사용할 수 있게 됨.

```python
else:
    form = ArticleForm()
    return render(request, 'create.html', {'form' : form})
```



- 조건문 if 수정 (POST 요청인 경우) => 폼 데이터를 처리
  - `request.POST` 에는 사용자가 입력하는 input의 정보가 저장되어 있음
  - `form` 인스턴스 `ArticleForm()` 의 인자로 `request.POST` 를 넘기고,  `form`  변수에 저장함.

> Form 클래스에서 정의된 구성에 따라서 입력된 내용을 `ArticleForm` 이라는 인스턴스의 인자로 넘기면, 인스턴스는 클래스의 양식에 맞춰서 내용을 분석한다.
>
> 인스턴스는 사용자가 입력한 정보를 전혀 갖고 있지 않으므로, 이걸 인자로 넘겨줘야 한다고 이해해보자!

```python
if request.method == 'POST':
    form = ArticleForm(request.POST)
```



- Form이 유효한지 체크 (유효성 검증) : 입력한 데이터가 유효할 경우, create에 대한 코드 적용.
- `form.cleaned_data.get` 은 검정된 데이터를 가져옴.
  - cf) `request.POST.get`은 검증되지 않은 값을 가져옴.

```python
if form.is_valid():
    title = form.cleaned_data.get('title')
    content = form.cleaned_data.get('content')
    article = Article.objects.create(title=title, content=content)  

    return redirect('articles:detail', article.pk)
```



- 데이터가 유효성 검증에 실패할 경우, `if form.is_valid():` 조건문에 들어가지 않으며, 조건문에서 최종적으로 벗어남. 기본폼 생성하는 `create.html` 으로 돌려주기 위해, `return`값을 `else` 문 아래가 아니라 내어쓰기로 조건문 밖으로 빼줌.

  (else에 적용되는 경우와 유효성 검증에 실패하는 경우를 만족시키기 위해)

```python
else:
    form = ArticleModelForm()
return render(request, 'form.html', {'form' : form})
```



- create.html 수정하기
  - `input` 태그로 작성한 내용을  `ArticleForm` 의 인스턴스가 저장된 `form`으로 대체 
  - 개발자도구로 보면, {{ form }}은 아래의 HTML 코드와 같음

```html
<h1>New Aricle </h1>
<form method="POST">
    {% raw %}{% csrf_token %}{% endraw %}
    <!--<input type="text" name="title" required/>-->
    <!--<input type="text" name="content" required/>-->
    {{ form }}
    <input type="submit" value="Submit"/>
</form>
```

```html
<label for="id_title">Title:</label>
<input type="text" name="title" required="" id="id_title">
<label for="id_content">Content:</label>
<input type="text" name="content" required="" id="id_content">
```