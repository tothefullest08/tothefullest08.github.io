---
layout: post
title: Django 26 - 팔로우 기능, 유저모델(AbstractUser) 상속 (1)
category: Django
tags: [Django]
comments: true
---



# 팔로우 기능, 유저모델(AbstractUser) 상속 (1)

> [Django 21 M:N 관계설정](<https://tothefullest08.github.io/django/2019/06/11/Django21_relations3_many_to_many/>) 포스팅에서 좋아요 기능을 통해 M:N 관계를 설정하는 코드를 작성해보았음. M:N 관계설정을 이용하여  인스타그램의 팔로우 기능을 구현하고 Django 내장 유저모델을 상속받아보도록 하자.



### Django user 모델 확장의 4가지 방법

1. Proxy Model: 기존의 User 모델을 그대로 사용하되, 일부 동작을 변경하는데만 사용
2. User Profile: 하나의 새로운 모델을 정의한 후, User 모델과 1:1 관계설정(프로필 모델 참조)
3. AbstractBaseUser: 완전한 새로운  User 모델을 만들때 사용
4. AbstractUser: 기존의 User 모델을 사용하되, 추가적인 정보를 더 넣고 싶을 때 사용. 2번 방법은 추가로 클래스를 생성하지만, 이 방법의 경우 추가로 클래스를 생성하지는 않음.

[관련 내용 참조](<https://simpleisbetterthancomplex.com/tutorial/2016/07/22/how-to-extend-django-user-model.html#proxy>)



### accounts/models.py 

- Django 내장 유저 모델을 상속받아 사용
  - 기존의 User 모델을 사용하면서 추가적인 정보만 넣으므로 4번 방법(`AbstractUser`) 을 이용
  - 사용을 위해 import

```python
from django.conf import settings
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    followers = models.ManyToMnayField(settings.AUTH_USER_MODEL, related_name='followings')
```



### settings.py

- accounts/models.py 내 AbstractUser를 상속받은 User 모델을 새로 만들었으므로, 이 모델을 유저모델로 사용하기 위해 settings.py 내 추가 설정
- 유저모델을 변경하였으므로, DB & migration 파일을 삭제 하여 갈아엎은 후,  마이그레이션 재설정

```python
AUTH_USER_MODEL = 'accounts.User'
```



현재 상태에서 회원가입을 진행할 시에는 아래와 같이 에러가 발생한다.

> AttributeError at /accounts/signup/
> Manager isn't available; 'auth.User' has been swapped for 'accounts.User')



views.py/signup 함수에서 사용하는 `UserceationForm` 이 문제인데, 이 폼은 장고에서 기본적으로 제공하는 폼으로, 장고 기본 유저모델을 기반으로 만들어져있음. 그러나 settings.py 에서 커스텀 유저모델을 기본 모델로 설정을 변경하였기 때문에 에러가 발생. 따라서, 우리가 만든 유저모델을 기반으로 사용하기위해서는 해당 폼을 커스터마이징할 필요가 있음.



### UserCreationForm 커스터마이징

#### forms.py

- `UserCreationForm` 을 상속받아 `CustomUserCreationForm` 이라는 커스터마이징된 폼 생성
  - `get_user_model()` 은 settings.py 에서 설정한 유저모델을 가져옴.
  - fields에는 `UserCreationForm.Meta.fields` 를 입력


```python
from django.contrib.auth.forms import UserCreationForm

class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        models = get_user_model()
        fields = UserCreationForm.Meta.fields
```



#### views.py

- signup 함수 내에서 사용하는 `UserCreationForm`을 `CustomUserCreationForm` 으로 변경
  - 사용을 위해 import

```python
from .forms import CustomUserCreationForm

def signup(request):
    if request.user.is_authenticated:
        return redirect('posts:list')
        
    if request.method == 'POST':
        signup_form = CustomUserCreationForm(request.POST)
        if signup_form.is_valid():
            user = signup_form.save()
            Profile.objects.create(user=user) 
            auth_login(request, user)
            return redirect('posts:list')
    
    else:
        signup_form = CustomUserCreationForm()
    
    return render(request, 'accounts/signup.html', {'signup_form':signup_form})
```



### urls.py

- 팔로우 기능에 대한 urlpattern 설정

```python
urlpatterns = [
    path('<int:user_id>/follow/', views.follow, name='follow'),
]
```


