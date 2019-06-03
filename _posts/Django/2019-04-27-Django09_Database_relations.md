---
layout: post
title: Django 09 - 1:N 관계 설정 / 댓글 기능 구현
category: Django
tags: [Django]
comments: true
---





# 1:N 관계 설정 (Database relation)

> 앞서, 게시글을 작성하는 코드(CRUD)를 작성해보았음.  더 나아가, 각 게시글에 댓글을 쓰는 코드를 작성해 보도록 하자. 이때 한 게시글에는 여러개의 댓글이 작성 될 수 있으므로, 게시글과 댓글은 1:N 구조를 띔.

### models.py 내 comment 클래스 정의

- `post` 는 `models.ForeignKey()`  객체가  저장되어 있는 변수임.하기 예시의 경우, 댓글(Comment)이 새로 생성될때, 어떠한 게시글(Post)인지를 나타내는 대한 변수 명임.
- `models.ForeignKey()` 는 외래키를 설정하는 함수임. 
  - 첫번째 인자: 외래키가 연결되는 테이블을 입력함. 여기서는 `Post` 모델 클래스를 의미함
  - 두번째 인자: ForeignKeyField가 바라보는 값이 삭제 될 때, 어떻게 처리할건지를 옵션으로 정함
    - `CASCADE` : 부모가 삭제 되면, 자기 자신도 삭제
      예) 게시글에 대한 모델 클래스인 `Post` 에 따라 생성된 게시글이 삭제되면, 댓글에 대한 모델 클래스인 `Comment`에 따라 생성된 댓글도 삭제)
    - `PROTECT` : 자식이 존재하면, 부모 삭제 불가능 ( `ProtectedError `발생시킴)
    - `SET_NULL` : 부모가 삭제되면, 자식의 부모 정보를 NULL로 변경 

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    content = models.TextField()
```

- 클래스 작성 후 마이그레이션 재실행
  - `python manage.py makemigrations`
  - `python manage.py migrate`

### Python Shell 을 통해 데이터 입력

- `python manage.py shell` : 파이썬 쉘 실행
- `from posts.models import Post, Comment` : models.py 내 Post 및 Comment 클래스 import
- `post` = `Post.objects.last() `  : 데이터 베이스에 마지막으로 저장된 데이터를 post라는 변수에 저장.

```python
>>> post = Post.objects.last()
>>> post
<Post: 제목입니다>
```



- 댓글에 대한 모델 클래스인 `Comment` 를 호출하여, `c`라는 클래스의 인스턴스를 생성한다. 이때, post에는 위에서 정의한 인스턴스`post`를 값으로 저장시킨다. 이를 통해, 해당 댓글이 어느 글에 대한 것인지를 알 수 있게 됨. `content`에는 내용을 작성
  - 동일하게 `Comment` 모델 클래스를 통해 DB에 저장된 모든 레코드를 불러올 수 있음.
  - 또한, `c.post`  & `c.content`와 같이 `Comment` 클래스의 멤버변수를 호출 할 수도 있음.

```python
>>> c = Comment(post=post, content='댓글입니다!')
>>> c.save()
>>> Comment.objects.all()
<QuerySet [<Comment: Comment object (1)>]>

>>> c.post
<Post: 제목입니다>
>>> c.content
'댓글입니다'
```

- `post` 변수(`Post` 모델 클래스의 인스턴스)에 저장된 모든 댓글(관계로 연결된 `Comment` 클래스의 객체들)을 모두 가져올 수도 있음
- 이때 SQL 문법에 따라 모델 클래스 Comment는 소문자로 입력해줘야함.

```python
>>> post.comment_set.all()
<QuerySet [<Comment: Comment object (1)>, <Comment: Comment object (12)>]>
```

- `post.comment_set.first()` 는 post 변수 내`Comment` 모델 클래스를 통해 DB에 저장된 첫번째 레코드를 가져온다. 이 레코드는 객체를 갖고 있으며,  객체가 들고 있는 멤버 변수 `content` 를 반환할 수 있음.

```python
>>> post.comment_set.first()
<Comment: Comment object (1)>
>>> post.comment_set.first().content
'댓글입니다!'
```

- `c.post` 를 하면, `c` (`Comment` 모델 클래스의 인스턴스)의  멤버변수인 `post` 에 저장된 값을 반환하는데, 그 값은 `Post` 모델 클래스의 객체이다. 예) `Post.objects.last()` 
- 따라서, `post` 객체가 들고있는 변수도 당연히 반환 할 수 있으므로 아래의 방법으로 접근이 가능함.

```python
>>> c = Comment.objects.get(pk=1)
>>> c
<Comment: Comment object (1)>
>>> c.post
<Post: 제목입니다>
>>> post.title
'제목입니다'
>>> post.content
'내용입니다'
>>> c.post.title
'제목입니다'
>>> c.post.content
'내용입니다'
```



### Django 내 댓글 생성 기능 구현

```python
#views.py

#모델 클래스 Comment를 import해옴
from .models import Post, Comment

def comments_create(request, post_id):
    #댓글을 달 게시물에 대한 정보 가져오기
    post = Post.objects.get(pk=post_id)
    #form 태그에서 넘어온 댓글 내용 가져오기
    content = request.POST.get('content')
 
    #댓글 생성 및 저장 
    comment = Comment(post=post, comment=comment)
    comment.save()
    
    #댓글 생성후, 디테일 페이지로 redirect시킴
    return redirect('posts:detail', post.pk)


#urls.py
path('<int:post_id>/comments/create/', views.comments_create, name='comments_create'),
```

- detail.html
  - 댓글 생성을 위한 추가적인 html 페이지는 생성할 필요 없으며 디테일 페이지 내에서 작성 가능
  - form & input 태그등을 사용하여, `comment_create` 함수를 호출하기 위한 url 주소를 지정
  - 저장된 댓글(전체 댓글) html에서 보여주기 위해, 진자 템플릿을 이용하여 반복문을 실시
  - 이때, 댓글에 대한 템플릿 변수는 추가로 설정할 필요는 없으며, `post`를 통해 댓글에 접근이 가능함.
    - `post` 와 연결된`Comment` 모델 클래스의 객체을 불러오는 `post.comment_set.all` 을 사용
    - **html문서 내에서는 `all()` 을 쓰지 않음을 유의하자.** 

```html
	{% raw %}<form action="{% url 'posts:comments_create' post.pk %}" method="post">{% endraw %}
        {% raw %}{% csrf_token %}{% endraw %}
        댓글 : <input type="text" name="content"/>
        	  <input type="submit" value="Submit"/>
    	</form>

    <ul>
        {% raw %}{% for comment in post.comment_set.all %}
        <li> {{ comment.content }} </li>
        {% endfor %}{% endraw %}
    </ul>
```



### Django 내 댓글 삭제 기능 구현

- 게시글의 정보와 댓글의 정보가 둘다 필요하므로, variable routing으로  `post_id` 와 `comment_id` 를 둘다 넘겨 줘야함. 
```python
#views.py
def comments_delete(request, post_id, comment_id):
    comment = Comment.objects.get(pk=comment_id)
    comment.delete()
    
    return redirect('posts:detail', post_id)

#urls.py
path('<int:post_id>/comments/<int:comment_id>/delete/', views.comments_delete, name='comments_delete'),
```



- form 태그와 input 태그등을 적절히 사용하여, 댓글을 삭제하는 내용이 정의된  `comment_delete` 함수를 호출하기 위한 url 주소를 지정한다.
```html
<-- detail.html -->    
<ul>
  {% raw %}{% for comment in post.comment_set.all %}
   <li> {{ comment.content }} - 
    {% raw %}<a href="{% url 'posts:comments_delete' post.pk comment.pk %}">Delete</a>{% endraw %}
    </li>
   {% endfor %}{% endraw %}
</ul>
```

