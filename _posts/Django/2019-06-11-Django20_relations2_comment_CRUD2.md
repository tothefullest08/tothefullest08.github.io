---
layout: post
title: Django 20 - 댓글 CRUD (2)
category: Django
tags: [Django]
comments: true
---



# 댓글 CRUD (2)

### 댓글 삭제하기 (Delete)

urls.py의 댓글 삭제 url에 `post_id` 와 `comment_id` 2개를 받아 넘기므로 이에 따라 2개의 variable routing이 필요하다. 따라서, url 경로에 명기된 variable routing 순서에 맞춰서 views.py 함수의 인자로 넘겨 줘야한다.
- `require_http_method` 을 `import` 하여 GET 또는 POST 방식의 경우 코드가 실행되게 데코레이터 설정. 
  (기본적으로는 POST 방식이 적합하나, 코드 작성의 편의성을 위해  GET 방식으로 코드구현)
- `Comment` 모델의 멤버 변수인 `commnet.user` 이 현재 요청을 보내는 `user` 와 일치하지 않는 경우, 댓글이 삭제되지 않고 다시 리스트 페이지가 뜨도록 redirect 설정
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
        <a href="{% raw %}{% url 'posts:comment_delete' post.id comment.id %}{% endraw %}">댓글 삭제</a>
        {% raw %}{% endif %}{% endraw %}
    </div>
    {% raw %}{% empty %}{% endraw %}
    <div class="card-text">댓글이 없습니다.</div>
    {% raw %}{% endfor %}{% endraw %}
</div>
```



### 댓글 수정하기(Update)

views.py 내 댓글을 수정하는 코드는 게시글을 수정하는 코드를 작성했던 방식과 매우 유사함. 
- `get_object_or_404`  & `Comment` 모델을 이용하여 원하는 댓글에 대한 정보를 갖고 옴. 
- HTTP Method 가 GET 인 경우;\
  - `commentForm` 의 instance에 현재 인스턴스 객체의 값을 넣어준 후, `comment_form` 변수에 저장시키고, comment_update.html에 템플릿 변수로 넘김. 브라우저를 통해 html 문서에 접근하면 기존에 저장되있는 정보를 확인 할 수 있음.
- HTTP Method가 POST인 경우
  - comment_html 에서 내용을 수정한 후 제출할 경우, form 태그 내 method 속성값에 따라, POST 방식으로 데이터를 요청보내게 되고, `request.method == 'POST'` 이하 코드가 실행된다.
  - CommentForm의 첫번째 인자 `request.POST `에는 수정된 데이터의 정보가 들어가 있으며, 2번째 인자인 `instance=comment` 에는 어떤 댓글인지에 대한 정보가 담겨있음. 
  - 유효성 검증을 통과한 경우, DB에 수정된 내용을 저장시키고, 리스트 페이지로 `redirect` 시킴.

```python
#views.py
def comment_update(request, post_id, comment_id):
    comment = get_object_or_404(Comment, id=comment_id)

    if request.user != comment.user:
        return redirect('posts:list')

    if request.method == 'POST':
        comment_form = CommentForm(request.POST, instance=comment)
        if comment_form.is_valid():
            comment_form.save()
        return redirect('posts:list')
    
    else:
        comment_form = CommentForm(instance=comment)
        return render(request, 'posts/comment_update.htm', {'comment_form':comment_form})
 
#urls.py
urlpatterns = [
    path('<int:post_id>/comment/<int:comment_id>/update/', views.comment_update, name="comment_update"),
]
```

- 댓글 수정 코드와 동일한 방식으로 `comment.user` 이 `user`와 일치한 경우에만 수정 버튼이 보이도록 `_post.html`  코드 수정
  - 댓글을 보여주는 반복문 안에서 삭제 링크를 작성

```html
<div class="card-body">
    {% raw %}{% for comment in post.comment_set.all %}{% endraw %}
    <div class="card-text">
        <strong> {% raw %}{{ comment.user.username }}{% endraw %} </strong> {% raw %}{{ comment.content }}{% endraw %}
        {% raw %}{% if comment.user == user %}{% endraw %}
        <a href="{% raw %}{% url 'posts:comment_delete' post.id comment.id %}{% endraw %}">댓글 삭제</a>
        <a href="{% raw %}{% url 'posts:comment_update' post.id comment.id %}{% endraw %}">댓글 수정</a>     
        {% raw %}{% endif %}{% endraw %}
    </div>
```

