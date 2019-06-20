---
layout: post
title: Django 25 - 1:1 관계 설정 / 프로필 모델 구현 
category: Django
tags: [Django]
comments: true
---



# 1:1관계 설정 (프로필 모델 구현)

유저모델과 1:1로 대응하는 프로필 모델을 구현해보도록 하자.  하나의 유저는 하나의 프로필만을 가지므로 1:1의 관계를 띄는 것이 당연할 것임. 프로필 모델은 유저 모델을 기반으로 프로필 정보를 입력하는 모델로, 인스타그램의 프로필정보와 같은 역할을 하게끔 코드를 작성할 예정임.

### accounts/models.py

- user모델과 1:1관계를 형성해야하므로 `OneToOneField` 사용
- 그외 별명 & 자기소개 & 프로필 사진에 대한 필드를 추가
  - 이미지 필드를 사용하기 위해 `ProcessedImageField` & `ResizeToFill` import
- 프로필 필드의 경우, 값이 반드시 있어야 하는게 아니라 비어있어 괜찮다는 옵션 적용:  `blank = True`
- 작성 후 마이그레이션 재 설정

```python
from imagekit.models import ProcesssedImageField
from imagekit.processors import ResizeToFill
form django.conf import settings

class Profile(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    nickname = models.CharField(max_length=40, blank =True)
    introduction = models.TextField(blank=True)
    image = ProcessedImageField(
    		blank = True,
        	upload_to = 'profile/images',
        	processors = [ResizeToFill(300, 300)],
        	format = 'JPEG',
        	options = {'quality':90},
    		)

```



### views.py (프로필 생성)

유저와 프로필은 1:1관계이므로 유저가 존재할 경우 프로필도 반드시 존재해야함. 따라서 유저가 회원가입하면 프로필도 바로 생성되게 하자.

- signup (회원가입) 함수에 유저가 생성되는 코드 바로 아래에 프로필을 생성하는 코드 작성
- `profile` 모델 import
- 기존에 회원가입이 이미 완료된 유저들의 경우 프로필을 생성 할 수 없으므로 DB를 밀어서 초기화 진행
  - DB 초기화 경우, db.sqlite3 파일과 migrations폴더 내 0001, 00002 등으로 시작되는 파일을 삭제하면 됨.

```python
from .models import Profile

def signup(request):
    if request.user.is_authenticated:
        return redirect('posts:list')
        
    if request.method == 'POST':
        signup_form = UserCreationForm(request.POST)
        if signup_form.is_valid():
            user = signup_form.save()
            Profile.objects.create(user=user) #프로필 생성
            auth_login(request, user)
            return redirect('posts:list')
    
    else:
        signup_form = UserCreationForm()
    
    return render(request, 'accounts/signup.html', {'signup_form': signup_form})
```



### 프로필 수정

#### 기본 MTV 패턴 구현

유저가 존재하는 이상 프로필은 삭제할 필요가 없으므로 delete 로직은 불필요함. signup 함수를 통해서 create로직은 이미 구현되어있으므로 update 로직에 대한 코드만 작성하면 됨.

```python
#views.py
def profile_update(request):
    return render(request, 'accounts/profile_update.html')


#urls.py
urlpatterns = [
    path('profile/', views.profile_update, name='profile_update'),
]
```

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>Profile Update</h1>

{% raw %}{% endblock %}{% endraw %}
```



#### forms.py  

- 프로필 모델 폼 생성 (프로필 모델 import)

```python
from .models import Profile

class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['nickname', 'introduction', 'image',]
```



#### views.py 

- forms.py 에서 새로 생성한 프로필 모델폼 import
- 요청을 보내는(현재 로그인된) 유저의 프로필(`request.user.profile`) 을 `profile` 이라는 변수에 저장
- HTTP methods(GET/POST)에 따라, 프로필 폼을 넘기거나, 사용자가 html 문서에서 새로 입력한 프로필 정보를 받아와 내용을 업데이트 & DB에 저장하게 코드를 작성
  - POST방식의 경우, 이미지 파일이 있으므로 `request.FILES` 를 인자로 넘겨줄 것.

```python
from .forms import ProfileForm

def profile_update(request):
    profile = request.user.profile
    if request.method == 'POST':
        profile_form = ProfileForm(request.POST, request.FILES, instance=profile)
        if profile_form.is_valid():
            profile_form.save()
        return redirect('accounts:people', request.user.username)
    else:
	    profile_form = ProfileForm(instance=profile)
    return render(request, 'accounts/profile_update.html', {
        						'profile_form':profile_form
    							})
```



#### profile_update.html

- 프로필 폼을 부트스트랩 양식으로 나타냄
- 이미지 파일이 있으므로, `enctype` 속성값 적용

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% load bootstrap4 %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<h1>Profile Update</h1>

<form method='POST' enctype="multipart/form-data">
    {% raw %}{% csrf_token %}{% endraw %}
    {% raw %}{% bootstrap_form profile_form %}{% endraw %}
    <input type="submit" value="Submit"/>
</form>

{% raw %}{% endblock %}{% endraw %}
```



### people.html

- 프로필에 대한 정보를 people.html에 표시
  - 조건문을 사용하여 프로필 이미지가 있는 경우, 프로필 이미지를 보여줌
  - bootstrap - gridsystem을 사용하여 적절히 영역 분배
- 프로필 수정 링크를 a태그로 연결

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% load bootstrap 4 %}{% endraw %}
{% raw %}{% block container %}{% endraw %}

<div class="container">
    <div class="row">
        <div class="col-3">
            <h1> 
            {% raw %}{% if people.profile.image %}{% endraw %}
            <img src="{% raw %}{{ people.profile.image.url }}{% endraw %}" width= 70, 
                 alt="{% raw %}{{ people.profile.image}}{% endraw %}">
            {% raw %}{% endif %}{% endraw %}
            {% raw %}{{ people.username }}{% endraw %}
            </h1>
        </div>
        <div class="col-9">
            <div>
                <strong>{% raw %}{{ people.profile.nickname }}{% endraw %}</strong>
            </div>
            <div>
                {% raw %}{{ people.profile.introduction }}{% endraw %}
            </div>
        </div>
    </div>
    
    {% raw %}{% if user == people %}{% endraw %}
    <div>
        <a href="{% raw %}{% url 'accounts:profile_update' %}{% endraw %}">프로필 수정</a>
```



### _post.html

- 리스트 페이지의 각 게시글에 작성자의 프로필 이미지가 보이도록 코드 작성

```html
<div class="card" style="width: 18rem;">
  
  <div class="card-header">
    {% raw %}{% if post.user.profile.image %}{% endraw %}
    <img src="{% raw %}{{ post.user.profile.image.url }}{% endraw %}" width="25" alt="">
    {% raw %}{% endif %}{% endraw %}
```

