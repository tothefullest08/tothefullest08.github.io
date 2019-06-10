---
layout: post
title: Django 19 - 1:N 관계설정 (댓글)
category: Django
tags: [Django]
comments: true
---



# 1:N 관계 설정 (댓글)

> 1:N 관계 설정을 통해 각 게시글에 댓글을 쓰는 코드를 작성해보자. 유저와 댓글간의 1:N 및 게시글과 댓글 간의 1:N, 이중 1:N 관계를 띄도록 코드를 작성 할 수 있음.



## 기본설정하기

### Comment 모델 클래스 생성

- 어떤 유저가 쓴 댓글인지 & 어떠한 게시글에 달린 댓글인지에 대한 1:N 관계 설정
  - 클래스는 posts 어플리케이션 내에 작성
- `models.ForeignKey` 의 첫번째 인자로는 외래키가 연결되는 테이블을 입력함. 
  - `settings.AUTH_USER_MODEL` 에는 유저 테이블에 대한 정보가 담겨 있음.
- 두번째 인자에는 `ForeignKeyField` 가 바라보는 값이 삭제될 때 어떻게 처리할지를 옵션으로 정함
  - `CASCADE` : 부모가 삭제 되면, 자기 자신도 삭제
  - `PROTECT` : 자식이 존재하면, 부모 삭제 불가능 ( `ProtectedError `발생시킴)
  - `SET_NULL` : 부모가 삭제되면, 자식의 부모 정보를 NULL로 변경 

```python
from django.conf import settings

class Comment(models.Model):
    user = models.ForeignKey(setting.AUTH_USER_MODEL, on_delete=models.CASCADE)
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    content = models.TextField()
```

- 스키마 재설정(`makemigrations` & `migrate` )



### Comment 모델에 대한 모델폼 생성

```python
from django import forms
from .models import Comment

class CommentForm(forms.ModelForm):
    model = Comment
    fields = ['content',]
```



## 댓글 생성 및 읽기(Create & Read)

### 댓글 생성하기

- models.py & forms.py 내 새로 생성한 클래스인 `Comment` 와 `CommentForm`을 각각 import
- 앞 포스팅에서 views.py - create 함수를 통해 게시글을 생성할 때, 유저정보를 입력하였던 방법을 동일하게 호활용하여,  Comment - CommentForm 으로 wrapping 되어있는 데이터를 한번 벗겨 낸 후, comment의 인스턴스 객체의 멤버변수인 user 과 post에 각각 데이터를 집어넣음.
  - `comment`의 멤버변수인 `post` 는 `Post` 모델과 외래키로 연결되어있으며, 이때 자동적으로 연결된 `post`에 대한 id를 입력하는 column이 생성된다. `Comment` 모델 클래스를 보면 `post`  필드라고 명시 되어있지만 실제로 생성되는 필드는 `post`가 아니라 `post_id` 이다. (Default 값)
- `comment.post_id` 에서 `post_id` 는 함수의 2번째 인자인 `post_id` 와는 무관함을 유의하자. 
  - 함수의 2번째 인자로 들어오는 `post_id` 는 urls.py를 통해 들어오는 variable routing을 의미.
  - `comment.post_id` 는 `comment` 모델에서 외래키로 연결된 `post`의 id 값을 저장시키는 필드명을 나타냄.

```python
#views.py
from .models import Comment
from .forms import CommentForm

def comment_create(request, post_id):
    comment_form = CommentForm(request.POST)
    if comment_form.is_valid():
        comment = Comment_form.save(commit=False)
        comment.user = request.user
        comment.post_id = post_id
        comment.save()
    return redirect('posts:list')

#urls.py
urlpatterns = [
    path('<int:comment_id>/comments/create/', views.comment_create, name='comment_create'),
]
```



- 위의 코드는 아래와 같은 방법으로도 나타낼 수 있음. 그러나 DB를 접속하여 어떠한 게시글인지에 대한 정보를 한번 불러오는 등에 불필요한 과정이 포함되어있으므로 비효율적임.

```python
def comment_create(request, post_id):
    #DB에서부터 어떠한 게시글인지에대한 정보를 찾아서 post변수에 저장
    post = get_object_or_404(Post, id=post_id)
    comment_form = CommentForm(request.POST)
    if comment_form.is_valid():
        comment = comment_form.save(commit=False)
        comment.user = request.user
        #comment.post를 통해 comment에 대한 1:N 관계 설정
        comment.post = post
        comment.save()
        
    return redirect('posts:list')
```



### HTTP methods에 대한 데코레이터 설정

이전 포스팅에서 `@login_required` 데코레이터를 사용하여 로그인이 되어야만이 함수가 실행되게끔 설정을하였던 것처럼, HTTP methods에 대한 데코레이터 설정이 가능함. 해당 데코레이터 사용을 위해 관련코드 `import`

- `from django.views.decorators.http import require_POST`
- `import` 이하에는 원하는 HTTP methods를 입력해주면 됨
  - `require_POST`:  POST 방식으로 주소를 접근할때만 함수를 실행하게 설정
  - `require_GET` : GET 방식만 가능
  - `require_safe` :GET 방식과 HEAD방식만 가능
  - `require_http_methods(['GET','POST'])` : GET, POST방식 사용 가능
- 위의 방법을 사용하면 `if request.method == 'POST'` 와 같은 조건문을 사용할 필요가 없음
- <https://docs.djangoproject.com/en/2.2/topics/http/decorators/>

```python
from django.views.decorators.http import require_POST

@require_POST
def comment_create(request, post_id):
    comment_form = CommentForm(request.POST)
    if comment_form.is_valid():
        comment = comment_form.save(commit=False)
        comment.user = request.user
        comment.post_id = post_id
        comment.save()
        
    return redirect('posts:list')
```



### 댓글 표시하기(views.py)

- 리스트 페이지에 댓글을 보여주기 위해 list 함수 수정
- forms.py 내 생성한 `CommentForm` 을 사용하기 위해 불러온 후, 템플릿 변수로 넘겨줌
- 추가로, 최근에 작성한 게시글이 상위에 나타내기 위해 `order_by('-id')` 입력

```python
def list(request):
    posts = Post.objects.order_by('-id').all()
    comment_form = CommentForm()
    return render(request, 'posts/list.html', {'posts':posts, 'comment_form': comment_form})
```



### 댓글 표시 & 생성(_post.html)

- 댓글 정보 및 생성기능을 실제로 브라우져에 나타내기 위한 코드를 `_post.html` 에 작성
- urls.py 에 작성된 url 경로에 따라 함수가 실행될 수 있도록 form 태그 및 action 속성 경로 설정
  - {% raw %}`{% if user.is_authenticated %}`{% endraw %} 을 사용하여 로그인이 된 경우만 댓글생성이 보이도록 함.
- 댓글 목록도 동시에 출력 (부트스트랩 레이아웃 적용)
  - {% raw %}'`{% for comment in post.comment_set.all %}`{% endraw %}  진자템플릿을 이용하여 반복문을 실행함.
  - 실제 코드는 `post.comment_set.all()` 으로 작성되어야 하나, 진자템플릿 문법에는 `()` 이 생략됨.
  - 반복문이 비어있을 경우, {% raw %} `{% empty %}`{% endraw %}  이하 코드 실행됨.



※ Post와 Comment는 models.py를 통해 1:N 관계가 설정되어있으며, 각각의 위치에서 관계된 다른반대쪽을 불러오는 방법은  아래와 같음
  - Post에서 Comment로 접근할 때(1 => N) : `post.comment.set.all()`
  - Comment에서 Post로 접근할 때(N => 1) : `comment.post`

```html
<!-- _post.html -->

<div class="card-body">
    {% raw %}{% for comment in post.comment_set.all %}{% endraw %}
      <div class="card-text">
        <strong> {% raw %}{{ comment.user.username }}{% endraw %} </strong> {% raw %}{{ comment.content }}{% endraw %}
      </div>
    {% raw %}{% empty %}{% endraw %}
      <div class="card-text">댓글이 없습니다.</div>
    {% raw %}{% endfor %}{% endraw %}
</div>

{% raw %}{% if user.is_authenticated %}{% endraw %}
<form action="{% raw %}{% url 'posts:comment_create' post.id %}{% endraw %}" method="POST">
   {% raw %}{% csrf_token %}{% endraw %}
   {% raw %}{{ comment_form }}{% endraw %}
   <input type="submit" value="Submit"/>
</form>
{% raw %}{% endif %}{% endraw %}
```



### 모델 필드속성 오버라이딩 설정

Model은 실제 DB에 저장 시키기 위해 사용(스키마의 형태)하는 역할을 함. `Comment` 모델의 `content` 필드의 경우 `TextField`를 사용하였으나, 실제 댓글은 길이의 제한이 없는 `TextField` 보다는 길이제한을 둘 수 있는 `CharField`가 좀 더 적합하다. 따라서 `content`의 필드 속성을 `TextField ` 에서 `CharField` 로 변경하는 작업을 forms.py에서 할 수 있음.(<https://docs.djangoproject.com/en/2.2/topics/forms/modelforms/#overriding-the-default-fields>)

- `label` :  기본적인 input의 이름. 빈 값을 넣어서 라벨을 제거.
- `widget` : 속성값을 줄 때 사용. `widget`의 속성을 `forms.TextInput` 이나 `forms.Textarea`   등 형태를 설정할 수 있음.
  - `TextInput` 의 속성을 괄호안에 사용. `attrs={}` 와 같이 딕셔너리의 형태로 입력 가능. 

```python
#forms.py

class CommentForm(forms.ModelForm):
    content = forms.CharField(label='', widget=forms.TextInput({attrs={'class':'form-control', 'placeholder': '댓글을 작성하세요.'}}))
    class Meta:
        model = Comment
        fields = ['content',]
```



- 아래와 같은 방법으로도 작성 가능함

```python
class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
        widgets = {
            'content': forms.TextInput(attrs={'placeholder':'댓글을 입력하세요'})
        }
```





### 댓글 생성 폼에 bootstrap 입히기 (_post.html)

```html
{% raw %}{% if user.is_authenticated %}{% endraw %} 
<form action="{% url 'posts:comment_create' post.id %}" method="POST">
    {% raw %}{% csrf_token %}{% endraw %} 
    <div class="input-group">
        {% raw %}{{ comment_form }}{% endraw %} 
        <div class="input-group-append">
            <input type="submit" value="Submit" class="btn btn-primary"/>
        </div>
    </div>
</form>
{% raw %}{% endif %}{% endraw %} 
```



## 댓글 삭제하기

댓글의 경우, 게시글 및 댓글에 대한 고유 id 값이 각각 필요하므로 2개의 variable routing이 필요하다. 아울러 url 경로에 명기된 variable routing 순서에 맞춰서 views.py 함수의 인자로 넘겨줘야한다.

- `require_http_method` 을 `import` 하여 GET 또는 POST 방식의 경우 코드가 실행되게 데코레이터 설정. 
  (기본적으로는 POST 방식이 적합하나, 코드작성의 편의성을 위해  GET 방식으로 코드구현)
- `Comment` 모델의 멤버 변수인 `commnet.user` 이 현재 요청을 보내는 `user` 와 일치하지 않는 경우, 댓글이 삭제되지 않고 다시 리스트 펭지가 뜨도록 redirect 설정
- 반대의 경우(user 정보가 일치할 경우), DB에서 해당 댓글을 삭제시킴

```python
from django.views.decorators.http import require_POST, require_http_methods

@require_http_methods(['GET', 'POST'])
def comment_delete(requests, post_id, comment_id):
    comment = get_object_or_404(Comment, id=comment_id)
    if comment_user != request.user:
        return redirect('posts:list')
   	comment.delete()
    return redirect('posts:list')

#urls.py
urlpatterns = [
    path('<int:post_id>/comments/<int:comment_id>/delete/', views.comment_delete, name="comment_delete")
]
```



- 동일한 방식으로 `comment.user` 이 `user`와 일치한 경우에만 삭제 버튼이 보이도록 `_post.html`  코드 수정
  - 댓글을 보여주는 반복문 안에서 삭제 링크를 작성

```html
<div class="card-body">
    {% raw %}{% for comment in post.comment_set.all %}{% endraw %}
    <div class="card-text">
        <strong> {% raw %}{{ comment.user.username }}{% endraw %} </strong> {% raw %}{{ comment.content }}{% endraw %}
        {% raw %}{% if comment.user == user %}{% endraw %}
        <a href="{% raw %}{% url 'posts:comment_delete' post.id comment.id %}{% endraw %}">삭제</a>
        {% raw %}{% endif %}{% endraw %}
    </div>
    {% raw %}{% empty %}{% endraw %}
    <div class="card-text">댓글이 없습니다.</div>
    {% raw %}{% endfor %}{% endraw %}
</div>
```

