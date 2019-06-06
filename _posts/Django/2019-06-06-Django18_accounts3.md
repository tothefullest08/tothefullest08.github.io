---
layout: post
title: Django 18 - 관계 설정(1) / 유저정보 추가 및 권한 설정
category: Django
tags: [Django]
comments: true
---







# 관계설정 (1) - 유저 정보 추가

### Models.py 수정

- 유저정보를 저장시킬 새로운 필드인 user를 posts/models.py 내 추가
- `settings.AUTH_USER_MODEL` 에는 Django에서 기본적으로 내장되어있는 User 모델에 대한 정보가 담겨있음. 해당 모델을 사용하기 위해 `from django.conf import settings` 입력
- `models.ForeignKey()`는 외래키를 설정하는 함수임
  - 1번째 인자에는 외래키가 연결되는 모델(테이블)을 입력함
  - 2번째 인자에는 외래키가 바라보는 값이 삭제될 때,  외래키를 갖고오는 필드를 어떻게 처리할건지에 대한 옵션을 입력함
    - `CASCADE` : 부모가 삭제 되면, 자기 자신도 삭제
    - `PROTECT` : 자식이 존재하면, 부모 삭제 불가능 ( `ProtectedError `발생시킴)
    - `SET_NULL` : 부모가 삭제되면, 자식의 부모 정보를 NULL로 변경 
- 스키마 재설정
  - `python manage.py makemigrations`
  - `python manage.py migrate`

```python
from django.conf import settings

class Post(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    content = models.TextField()
    ...(중략)
```



### 게시글에 유저정보 표시하기

`post_form` 은 모델폼인 `PostForm`의 양식에 따라 저장된 정보를 담고 있는 인스턴스 객체임. 모델 폼의 인스턴스 객체를 `post` 라는 변수에 다시 저장시킴. 이때, `commit=False` 속성을 적용하여 DB에 바로 저장시키지 않음. 

여기서 잠깐 ,  `post_form` 과 `post` 는 아래와 같이 정리할 수 있음

- `post_form` : 모델폼 `PostForm`의 인스턴스 객체
- `post` : 모델 `Post`의 인스턴스 객체 

모델폼은 모델로부터 일종의 wrapping되어 있는 개념인데,  모델에 user 정보를 넣기 위해서는 wrapping 되어있는 모델폼을 벗겨줄 필요가 있음. 이러한 작업을 하기 위해서 `PostForm` 의 인스턴스 변수를 `Post`의 인스턴스 변수인 `post`에 다시 저장시키는 것임. 

`post` 의 멤버변수인 `user`에 현재 들어오는 request의 user를 저장시키고, 최종적으로 DB에 저장을 시킴. 

이후, 유저 정보는 `_post.html`   내 원하는 영역에 {% raw %}`{{ post.user.username}}`  {% endraw %} 와 같은 방식으로 코드를 작성하여 표시 시킬 수 있음.

```python
@login_required
def create(request):
    if request.method == 'POST':
        post_form = PostForm(request.POST, request.FILES)
        if post_form.is_valid():
            post = post_form.save(commit=False)
            post.user = request.user
            post.save()
        return redirect('posts:list')
    
    else:
        post_form = PostForm()
    return render(request, 'posts/form.html', {'post_form': post_form})
```



### 사용자 권한 설정

Django에서는 내장되어있는 User 모델을 추가적인 작업없이 바로 불러올 수 있음. 이를 활용하여, 본인이 작성한 글에 대해서만 수정 또는 삭제를 할 수 있도록 설정을 할 수 있음.

- `_post.html` 에서 게시글의 유저정보와 로그인된 유저정보가 같은 경우에만 수정/삭제 링크가 뜨도록 조건문을 적용시킴.

```html
<!--게시글의 유저와 지금 로그인된 정보가 유저가 같다면-->
{% raw %}{% if post.user == user %}{% endraw %}
<a href="{% raw %}{% url 'posts:update' post.id %}{% endraw %}" class="btn btn-info">Update</a>
<a href="{% raw %}{% url 'posts:delete' post.id %}{% endraw %}" class="btn btn-primary">Delete</a>
{% raw %}{% endif %}{% endraw %}
```



- 그러나 게시글의 id값을 알고 있으면 주소창에 url 주소를 입력하여 여전히 수정 또는 삭제를 할 수 있음. 따라서 추가적인 코드 작성을 통해 url 주소를 통한 접근을 막을 필요가 있음.

```python
def update(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    
    #post의 유저와 현재 로그인된 유저가 일치하지 않다면, 리스트 페이지를 보여주게 함
    if post.user != reqeust.user:
        return redirect('posts:list')
    
    if request.method == 'POST':
        post_form = PostForm(request.POST, request.FILES, instance=post)

        if post_form.is_valid():
            post_form.save()
            return redirect('posts:list')
            
    else:
        post_form = PostForm(instance=post)
    
    return render(request, 'posts/form.html', {'post_form': post_form})

def delete(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    
    #post의 유저와 현재 로그인된 유저가 일치하지 않다면, 리스트 페이지를 보여주게 함
    if post.user != reqeust.user:
        return redirect('posts:list')

    post.delete()
    return redirect('posts:list')
```



