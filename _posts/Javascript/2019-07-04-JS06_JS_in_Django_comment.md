---
layout: post
title: JavaScript 06 - Django 내에서 JS 적용하기 (2) (댓글 생성 기능 동적 구현)
category: Javascript
tags: [Javascript, Django]
comments: true
---




# Django 내에서 JS 적용하기 (2) (댓글 생성 기능 동적 구현)

Django 인스타 프로젝트에서 좋아요 기능을 동적으로 구현했던 것 처럼, 댓글 기능 또한 동적으로 구현해 보도록 하자.

- 댓글 기능 관련 [포스팅](<https://tothefullest08.github.io/django/2019/06/10/Django19_relations1_comment_CRUD1/>)
- Django 인스타그램 기반 프로젝트 [소스코드](<https://github.com/tothefullest08/Django_Reivew_PJT>)



#### 1) 기존 코드

- _post.html 내 form 태그로 댓글 생성 코드가 작성되어있음.

```html
{% raw %}{% if user.is_authenticated %}
<form action="{% raw %}{% url 'posts:comment_create' post.id %}" method="POST">
    {% raw %}{% csrf_token %}
    {% raw %}{{ comment_form }}
    <input type="submit", value="submit">
</form>
{% raw %}{% endif %}{% endraw %}
```



#### 2) _post.html 수정

- 댓글 작성 폼 태그 수정
  - 자바스크립트에서 이 태그를 불러 올 용도로 클래스(`class="comment-form"`) 추가

```html
{% raw %}{% if user.is_authenticated %}
<form action="{% raw %}{% url 'posts:comment_create' post.id %}" method="POST" class="comment-form">
    {% raw %}{% csrf_token %}
    {% raw %}{{ comment_form }}
    <input type="submit", value="submit">
</form>
{% raw %}{% endif %}{% endraw %}
```



#### 3) list.html

- _post.html 내 폼태그를 불러옴
- `document.querySelectorAll("comment-form")` 을 통해 모든 포스트의 댓글들을 갖고옴.
- `addEventListener` 의 첫번째 인자로 `submit` 입력. `submit` 은 폼태그에서 submit을 누를 때 이벤트가 발생하게 설계되어있는 트리거이며, 이 트리거는 폼태그만 들고 있음.
- 아래의 코드를 통해 댓글 작성 후 submit을 누를 경우, 2가지의 이벤트가 작동함.
  - 기존의 이벤트(폼태그가 하는 행동. post 요청을 통해 action으로 url주소를 넘김)
  - `addEventListener` 통해 `console.log` 이벤트 작동
- 우리는 페이지를 새로고치지 않고 동적으로 변경되게 하고 싶으므로 기존의 이벤트를 막아야함 => `preventDefault` 사용 (기존의 이벤트는 막히게 됨)
- `preventDefault` 로 막지 않을 경우, 댓글 생성 버튼을 눌렀을때, MTV 패턴에 따라 함수가 실행되고 페이지가 새로고침되어 console창에 출력되는 event 객체를 확인할 수가 없음!

```javascript
const commentForms = document.querySelectorAll('.comment-form')
commentForms.forEach(function(form){
    form.addEventListener('submit', function(event){
        event.preventDefault()
        console.log(event)
    })
})
```



#### 4) list.html (2)

- 댓글의 주소 뿐만 아니라 텍스트(생성된 댓글)도 갖고와야함.
- `event.target` 에는 이벤트를 전달한 객체(이벤트가 시작된 DOM 요소)가 담겨 있음 (form 태그 안의 모든 정보가 담겨있음)
- `FormData` 검증하기
  - `FormData` 인터페이스는 form필드와 그 값을 나타내는 일련의 key/value 쌍을 쉽게 생성할 수 있는 방법을 제공함 (폼에 글을 작성하고, html에서 submit 버튼을 눌렀을 때, 보내게 되는 요청을 객체에 실어서 던지게 되는데, 그게 담겨있는 바구니의 개념)
  - `FormData` 는 일종의 클래스로, 클래스로부터 인스턴스를 만들 때는 `new` 키워드를 붙여 줘야함.
  - `FormData.entries()` : 객체가 담긴 모든 key/value 쌍을 순회할 수 있는 `iterator` 을 반환함.
- `event.target.action` 에는 댓글 생성에 대한 url 정보가 담겨있음.
- `axios.post`의 두번째 인자로, Formdata를 통해갈무리된 객체인 `data`를 넘겨줌.
- `response` 에는 html 문서 전체가 담겨있음. 댓글 관련 정보만 갖고오면 되므로, Django가 하는 응답을 수정해야함.

```javascript
const commentForms = document.querySelectorAll('.comment-form')
commentForms.forEach(function(form){
    form.addEventListener('submit', function(event){
        event.preventDefault()
        
        const data = new FormData(event.target)
        
        //Inspect FormData
        //for (const item of data.entries()){
        //		console.log(item)
        //}
        
        axios.post(event.target.action, data)
        	.then(function(response){
            	console.log(response)
        })
    })
})
```



#### 4-1) `FormData.entries()`

- 위에서 FormData.entries()에는 form 태그를 통해 전송되는 값을 나타내는 key/value 쌍을 순회할 수 있는 iterator을 반환한다는 것을 알 수있음. 
- 위의 코드 박스 내 "Inspect FormData" 내 반복문 코드를 입력한 후, 댓글을 생성하고 console창에 찍힌 로그를 보면 아래와 같이 출력됨을 알수 있음
- Form 태그를 통해 POST방식으로 요청을 보낼 때는 아래와 같이 2가지의 정보가 전송됨 을 알 수 있음.
  - CSRF_TOKEN & 댓글 내용(content)

```javascript
(2) ["csrfmiddlewaretoken", "AuGAXE7Jio1BxQePvUfC85eDDcYU0OtSD2hF2EhvCVhrLsPxzNi6xfnMUz3K17Ll"]
	0: "csrfmiddlewaretoken"
	1: "AuGAXE7Jio1BxQePvUfC85eDDcYU0OtSD2hF2EhvCVhrLsPxzNi6xfnMUz3K17Ll"
	length: 2
	__proto__: Array(0)

(2) ["content", "댓글입니다"]
    0: "content"
    1: "댓글입니다"
    length: 2
    __proto__: Array(0)
```



#### 5) views.py

- 좋아요 기능 동적 구현때와 마찬가지로 `from django.http import JsonResponse`  입력 후 코드 수정
- 댓글 생성 함수의 return 값을 `JsonResponse`로 변경. `JsonResponse ` 에는 다음과 같은 정보를 저장하여 넘길 필요가 있음.
  - `comment.id` : 댓글의 id
  - `post_id` : 게시글의 id (어떠한 글에서 댓글이 생성되었는지 알아야 함)
  - `comment.user.username` : 댓글 작성자
  - `comment.content` : 댓글 내용

```python
from django.http import JsonResponse

def comment_create(request, post_id):
    comment_form = CommentForm(request.POST)
    if comment_form.is_valid():
        comment = comment_form.save(commit=False)
        comment.user = request.user
        comment.post_id = post_id
        comment.save()
        
    # return redirect('posts:list')
    return JsonResponse({
                            'id': comment.id,
                            'postId':post_id,
                            'username':comment.user.username,
                            'content':comment.content,
                        })
```



#### 6) _post.html (2)

아직까지는 어떤 포스트에 대한 댓글인지 자바스크립트는 알 수 없음. `<ul>` 는 댓글들이 존재하는, 댓글들을 보여주는 태그이며 이곳에 id값을 부여 해줘야함.

- `<ul id="comment-list-{% raw %}{{ post.id }}{% endraw %}">` 를 기반으로 댓글이 달릴 위치를 갖고오고, 새로운 댓글을 붙여주면 됨.

```html
<ul id="comment-list-{% raw %}{{ post.id }}{% endraw %}">
{% raw %}{% for comment in post.comment_set.all %}{% endraw %}
    <li>
    <strong>{% raw %}{{ comment.user }}{% endraw %}</strong> | {% raw %}{{ comment.content }}{% endraw %}
    {% raw %}{% if comment.user == user %}{% endraw %}
        <a href="{% raw %}{% url 'posts:comment_delete' post.id comment.id %}{% endraw %}">댓글 삭제</a>
        <a href="{% raw %}{% url 'posts:comment_update' post.id comment.id %}{% endraw %}">댓글 수정</a>
    {% raw %}{% endif %}{% endraw %}
    </li>
{% raw %}{% endfor %}{% endraw %}
</ul>
```



#### 7) list.html (3)

- `response.data` 에는 댓글 생성 함수(`comment_create`)를 통해 return 되는 `JsonResponse` 오브젝트가 담겨있음. 이를 `comment` 변수에 저장
- `document.querySelector('#comment-list-${comment.postId}')`을 통해 어떠한 게시글에 댓글이 달린건지를 확인 후, 그 게시글에 담긴 댓글 전체 리스트를 갖고 옴. 후, `commentList` 변수에 저장. `comment.postId` 를 통해 post_id에 접근 가능 (`JsonResponse` 값 참조)
- 새 댓글을 표시하는 부분을 코드로 작성하여 `newComment` 변수에 저장시킴. 이때 변수의 값으로는 `_post.html` 에서 작성하였던 `<li>`  태그 이하 전체 코드를 갖고옴.
- 자바스크립트에서는 장고 템플릿을 사용할 수 없으므로 수정 요
  - {% raw %}`{{ comment.content }}`{% endraw %}와 같이 사용했던 장고 템플릿은 `comment` 변수에 저장된 `JsonResponse` 오브젝트를 사용하여 변경
  - {% raw %}`{{ comment.user }}` {% endraw %}=> `${comment.username}`
  - {% raw %}`{{ comment.content }}` {% endraw %}=> `${comment.content}`
  - url 주소는 하드하게 수정
  - {% raw %}`{% if comment.user == user %}`{% endraw %}  & {% raw %}`{% endif %}` {% endraw %}조건문은 필요 없음 (댓글을 쓴사람이 당연히 나이므로, 삭제도 당연히 가능)
- 진자 템플릿을 붙일 때는 `insertAdjacentHTML` 사용 (2개의 인자를 가짐 - 위치, 내용)
- 댓글 작성 후, 작성한 댓글 내용이 input 태그 양식 안에 남아있으므로, 생성 버튼 후 내용이 사라지게 하기 위해서는 폼 태그 그 자체를 의미하는 `event.target` 을 초기화 해야함. => `event.target.reset()`

```javascript
const commentForms = document.querySelectorAll('.comment-form')
commentForms.forEach(function(form){
    form.addEventListener('submit', function(event){
        event.preventDefault()
        
        const data = new FormData(event.target)

        axios.post(event.target.action, data)
        	.then(function(response){
            	const comment = response.data
                const commentList = document.querySelector(
                    `#comment-list-${comment.postId}`)
                const newComment = `<li>
				<strong>${ comment.username }</strong> | ${ comment.content}
				<a href="/posts/${comment.postId}/comments/${comment.id}/delete/">
삭제</a>
				<a href="/posts/${comment.postId}/comments/${comment.id}/update/">
수정</a>
    </li>`
                commentList.insertAdjacentHTML('beforeEnd', newComment)
            	event.target.reset()
        })
    })
})
```



#### 7-1) insertAdjacentHTML 구문 

```html
element.insertAdjacentHTML(position, text);
```

position은 아래 있는 단어만 사용 가능하다.

- `'beforebegin'` : element 앞에 
- `'afterbegin'` : element 안에 가장 첫번째 child
- `'beforeend'` : element 안에 가장 마지막 child
- `'afterend'` : element 뒤에

```
text(인자)는 HTML 또는 XML로 해석될 수 있는 문자열이고(html code), (DOM) tree에 삽입할 수 있다.
```

※비동기식으로 동적으로 만들 때는, 하나의 댓글을 뒤에 갖다붙이기보다는 전체 댓글을 갖고와서 뒤에 쓰는 것을 선호함.