---
layout: post
title: JavaScript 05 - Django 내에서 JS 적용하기 (좋아요 기능 동적 구현)
category: Javascript
tags: [Javascript, Django]
comments: true
---




# Django 내에서 JS 적용하기 (좋아요 기능 동적 구현)

Django 카테고리 내 인스타그램 기반의 [프로젝트](<https://tothefullest08.github.io/django/2019/06/22/Django27_relations6_ManyToMany_follow2/>) 포스팅을 통해 좋아요 기능을 구현해보았다. 이 때, 구현한 기능은 MTV 패턴에 따라서, url 경로를 요청 보내면, views.py 내 함수에 따라 return 되는 값을 template으로 보여주는 방식으로 구현 되었는데, 페이지를 새로고침을 해야 동작이 되었음. 

이제부터, 페이지 redirect 없이 JS를 이용하여 좋아요 기능이 동적으로 동작하도록 코드를 수정해보자.

- 좋아요 기능 관련 [포스팅](<https://tothefullest08.github.io/django/2019/06/11/Django21_relations3_many_to_many/>)
- Django 인스타그램 기반 프로젝트 [소스코드](<https://github.com/tothefullest08/Django_Reivew_PJT>)



## 1. 좋아요 기능 구현

#### 1) 기존 코드

- _post.html 내 좋아요 기능이 아래와 같이 구현되어있음.
- base.html 내 head 태그 내 axios CDN 추가
- 좋아요 버튼을 자바스크립트 요청으로 변경해보도록 하자.

```html
    <a href="{% raw %}{% url 'posts:like' post.id %}{% endraw %}">
        {% raw %}{% if user in post.like_users.all %}{% endraw %}
        <!-- 좋아요 취소 -->
        <i class="fas fa-heart"></i>
        {% raw %}{% else %}{% endraw %}
        <!-- 좋아요 -->
        <i class="far fa-heart"></i>
        {% raw %}{% endif %}{% endraw %}
    </a>
```



#### 2) _post.html 수정

- a 태그 삭제 -> 자바스크립트로 요청을 보내므로 필요 없음
- 조건문을 i 태그  클래스 안에 삽입
- 어떠한 포스트에 대한 버튼인지 하트에 정보를 더 추가해야함 (고유 클래스 부여)
  - like-button 클래스 속성(자바스크립트에서 클래스를 찾기 위한 용도)
  - 자바스크립트는 `data-id`를 통해 개별적인 오브젝트에 접근 가능
- `data-id` 는 [HTMLElement.dataset](<https://developer.mozilla.org/ko/docs/Web/API/HTMLElement/dataset>) 속성을 활용한 것으로 HTML 이나 DOM 요소의 커스텀 데이터 속성(data-*)에 대한 읽기와 쓰기 접근을 허용시킴. 

```html
<i class="{% raw %}{% if user in post.like_users.all %}{% endraw %}
          fas 
          {% raw %}{% else %}{% endraw %}
          far
          {% raw %}{% endif}{% endraw %} fa-heart
          like-button" data-id="{% raw %}{{post.id}}{% endraw %}">
</i>

<!-- 좋아요 취소할 경우, fas fa-heart 클래스가 적용
	 좋아요 경우, far fa-heart 클래스가 적용 -->
```



#### 3) list.html 내 JS 코드 작성

_post.html은 아래와 같이 list.html의 for문안에 있으므로, 그 안에서 코드를 작성할 경우, 똑같은 스크립트가 계속 생김. 따라서 자바스크립트 코드는 list.html에서 작성함

```html
{% raw %}{% for post in posts %}{% endraw %}
    {% raw %}{% include 'posts/_post.htm' %}{% endraw %}
{% raw %}{% endfor %}{% endraw %}
```

- 버튼이 한개만 있는 것이 아니므로 (각 post 별로 좋아요 버튼이 있음), 모든 버튼을 다 가지고 와야함. `document.querySelectorAll` 을 이용하여 모든 데이터를 가져오며, 반복문을 통해 개별적인 버튼에 대한 `addEventListentListener` 구현. 
- 이때 querySelectorAll의 인자로 i 태그 클래스 내에서 정의한 클래스명인 `like-button`를 사용
-  `event` 오브젝트 객체 안, 정확히는 `event.target.dataset.id`의 value에는 data-id의 값인 "{% raw %}{{post.id}}{% endraw %}이 저장되어있으며, 이를  좋아요 기능에 대한 url 주소를 불러올 variable routing값으로 쓰임.
- urlpattern에 따라 좋아요의 url 주소는  `<int:post_id>/like/` 와 같이 정의 되어 있으며, 이 주소를 axios.get의 인자로 넘김.
- 요청에 대한 응답(`.then` 메서드)으로, response 객체의 반환 값으로 views.py를 통해 redirect된 list.html 전체가 반환됨. 우리는 "좋아요" 기능만 구현하면 되므로, HTML 문서 전체를 반환할 필요가 없음. 이는 불필요한 응답이므로 views.py 수정이 필요함

```html
<script>
    const likebuttons = document.querySelectorAll('.like-button')
    likebuttons.forEach(function(button){
        button.addEventListener('click', function(event){
            // console.log(event)
            const postId = event.target.dataset.id
            axios.get(`{% raw %}/posts/${postId}/like{% endraw %}/`)
            		.then(function(response){
                		console.log(response)
            		})
        })
    })
</script>
```



#### 4) views.py 수정

- 기존의 return 값이었던 `redirect('posts:list')` 를  Json의 형태로 반환해줌.  사용을 위해 `JsonResponse`를 import 해줌
- 좋아요를 취소하는 경우, liked 변수는 False로,  반대의 경우(좋아요를 누르는 경우)는 liked 변수에 True 값을 적용

```python
from django.http import JsonResponse

@login_required
def like(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    
    if request.user in post.like_users.all():
        post.like_users.remove(request.user)
        liked = False
    else:
        post.like_users.add(request.user)
        liked = True
        
    #return redirect('posts:list')
    return JsonResponse({'liked':liked})
```



#### 5) list.html 내 JS 코드 추가 작성

- `response.data` 에는 views.py 내 like 함수를 통해 반환되는 Json 오브젝트가 저장되어 있으며, `response.data.liked` 를 통해 오브젝트의 key에 접근이 가능함.
- `event.target.classList` 에는 해당 클래스 내 데이터가 리스트와 같은 형태로 모두 저장되어 있음. remove 와 add를 통해 클래스 속성을 조작 할 수 있음.

> classList: DOMTokenList(3)
> 	0: "fa-heart"
> 	1: "like-button"
> 	2: "fas"
> 	length: 3
> 	value: "fa-heart like-button fas"

- `if (response.data.liked)`, 즉 liked 값이 True 라면, (좋아요 기능), 빈 하트를 의미하는 i 태그의 class 값인  `fas fa-heart` 에서 `fas` 를 제거한 후 `far` 를 추가 => 빈 하트에서 꽉찬 하트로 변경
- 좋아요를 취소하는 경우는 반대로 생각하면 됨.

```html
<script>
    const likebuttons = document.querySelectorAll('.likebutton')
    likebuttons.forEach(function(button){
        button.addEventListener('click', function(event){
            // console.log(event)
            const postId = event.target.dataset.id
            axios.get(`{% raw %}/posts/${postId}/like{% endraw %}`)
            		.then(function(response){
						// response.data => {'liked':True} 혹은 {'liked':False}
                		if (response.data.liked) {
                            event.target.classList.remove('far')
                            event.target.classList.add('fas')
                        } else {
                            event.target.classList.remove('fas')
                            event.target.classList.add('far`)
                        }
            		})
        })
    })
</script>
```



## 2. 좋아요 갯수 카운팅 기능 구현

좋아요 갯수 또한 자바스크립트를 통해 동적으로 구현이 가능함. 현재 _post.html에는 다음과 같이 코드가 구현되어있음. 자바스크립트는 해당 코드만을 정학히 캐치해서 변경하는 것이 불가능하므로, 이 코드만을 특정 태그로 묶은 후, 코드를 식별하기 위한 추가 정보를 삽입해야 함 (어떠한 게시글에 대한 숫자인지 등..)

```html
<!-- 기존 코드 -->
<p>{{ post.like_users.count }} 명이 좋아합니다. </p>
```



#### 1) _post.html 코드 수정

- 변경되는 부분을 `span` 태그로 묶은 후, 태그의 id 값을 {% raw %}`{{ post.id }}` {% endraw %}을 통해 동적으로 할당함. 이 id를 가지고 자바스크립트는 각 게시글 내 좋아요 갯수에 대한 고유 코드에 접근이 가능하게 됨.

```html
<!-- 변경한 코드 -->
<span id="like-count-{% raw %}{{ post.id }}{% endraw %}"> {{ post.like_users.count }}</span> 명이 좋아합니다.
```



#### 2) views.py 수정

- like 함수의 return 값으로 좋아요 갯수를 나타내기 위한 `post.like_users.count()` 를 JSON 객체로 넘김.

```python
@login_required
def like(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    
    if request.user in post.like_users.all():
        post.like_users.remove(request.user)
        liked = False
    else:
        post.like_users.add(request.user) 
        liked = True
    
    return JsonResponse({'liked': liked, 'count':post.like_users.count()})
```



#### 3) list.html 수정

- `document.querySelector()` 을 통해 _post.html 내 좋아요 갯수 코드에 접근 가능함. 인자로 우리가 설정한 span 태그의 id 값인 `like-count-{% raw %}{{ post.id }}{% endraw %}` 사용
- `postId` 는 `event.target.dataset.id` 가 저장된 변수로, 이는 좋아요 기능 내 `data-id="{% raw %}{{post.id}}{% endraw %}"` 을 통해 가져온 post의 id 값임.
- 해당 태그의 내용을 불러오는 `.innerText` 에 like 함수의 반환 값인 Json 객체 내 count의 value 값을 저장시킴.

```html
<script>
    const likebuttons = document.querySelectorAll('.likebutton')
    likebuttons.forEach(function(button){
        button.addEventListener('click', function(event){
            // console.log(event)
            const postId = event.target.dataset.id
            axios.get(`{% raw %}/posts/${postId}/like{% endraw %}`)
            		.then(function(response){
                
                		// 추가한 내용
                		document.querySelector(
                        	`{% raw %}#like-count-${postId}{% endraw %}`)
                			.innerText = response.data.count
                		// 끝
                
                		if (response.data.liked) {
                            event.target.classList.remove('far')
                            event.target.classList.add('fas')
                        } else {
                            event.target.classList.remove('fas')
                            event.target.classList.add('far`)
                        }
            		})
        })
    })
</script>
```
