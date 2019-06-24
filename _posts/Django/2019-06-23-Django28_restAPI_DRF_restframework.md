---
layout: post
title: Django 28 - Django-RestFramework(DRF)를 이용한 API 구현
category: Django
tags: [Django]
comments: true
---



# Django-RestFramework를 이용한 RESTful API 구현

DRF(Django RestFramework)는 Django 안에서 RESTful API 서버를 쉽게 구축할 수 있도록 도와주는 오픈소스 라이브러리이다.  

REST API에 대한 [이전 포스팅](<https://tothefullest08.github.io/django/2019/04/27/Django11_REST_API/>)에서 REST API에 대해 정리를 한 적이 있는데, REST란 간단히 URI는 자원을 표현하는데 집중하고, HTTP METHOD(POST / GET /PUT /DELETE)를 통해 행위(해당 자원을 제어하는) 에 대해 정의하는 방식의 아키텍쳐를 의미한다.  이와 같은 방식으로 적용된 API를 RESTful API 라고 한다.

따라서, DRF를 이용하여 RESTful API 백엔드 서버를 구축하는 작업을 해보도록 하자.



## 기본 설정

#### 1. 기본 설정

- `python -m venv api_venv` : api_venv 라는 가상환경 생성
- `source activate` : api_venv/Scripts/ 폴더로 이동 후 가상환경 실행 
- `pip install django==2.1.8` :  2.1.8 버젼의 장고 설치

- `django-amdin startproject api .`  : api 라는 신규 프로젝트 생성
- `python manage.py startapp musics` : musics 이라는 어플리케이션 생성
- `pip install djangorestframework` : DRF 라이브러리 설치
- 어플리케이션 생성 후, settings.py 에 앱 추가

```python
INSTALLED_APPS = [
    'rest_framework',
    'musics',
]
```



#### 2. models.py

- 가수(artist)에 대한 모델 생성
- 곡(music)에 대한 모델을 생성하고, artist 모델과 1:N 관계 설정 
- 댓글(comment)에 대한 모델 생성 후, music 모델과 1:N 관계 설정
- `def __str__(self)` 는 클래스가 생성되었을 때, 어떻게 보여줄건지를 오버라이딩하여 정의할 때 쓰는 함수
- 마이그레이션 설정

```python
from django.db import models

class Artist(models.Model):
    name = models.TextField()
    
    def __str__(self):
        return self.name

class Music(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE)
    title = models.TextField()
    
    def __str__(self):
        return self.title
    
class Comment(models.Model):
    music = models.ForeignKey(Music, on_delete=models.CASCADE)
    content = models.TextField()
```



#### 3. admin.py

- `python manage.py createsuperuser` 을 통해 admin페이지에 접속할 superuser 생성
- 이 프로젝트에서는 따로 create에 대한 로직을 생성하지 않을 예정이므로, admin 페이지에 접속하여 각 모델에 대한 정보를 DB에 저장하도록 하자

```python
from django.contrib import admin
from .models import Artist, Music, Comment

admin.site.register(Artist)
admin.site.register(Music)
admin.site.register(Comment)
```



#### 4. urls.py

- 프로젝트(api) / 어플리케이션(musics) 별로 urls.py 를 구분하고, 주소에서 API에 대한 것인것을 알 수 있게 네이밍 설정. v1은 version1을 의미
- admin 페이지를 사용하기 위해 import

```python
# 프로젝트 (api) urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('api/v1/', include('musics.urls')),
    path('admin/', admin.site.urls),
]

# 앱 (musics) urls.py
from django.urls import path
from . import views

urlpatterns = [
]

```



## Music 리스트 API 구현

#### 1. urls.py

```python
urlpatterns = [
    path('musics/', views.music_list),
]
```



#### 2. views.py 

- `@api_view()`: 어떠한 HTTP Methods를 허옹할건지에 대한 설정을 하는 데코레이터. 아래의 예시의 경우, GET 방식으로 music 리스트 함수를 호출하므로 `@api_view(['GET'])` 사용
- serializer에 대한 코드 작성 필요.

```python
from django.shortcuts import render
from .models import Music
from rest_framework.decorators import api_view

@api_view(['GET'])
def music_list(request):
    musics = Music.objects.all()
    
    #serializer
```



#### 3. serializers.py

장고 MTV 패턴에 따라 HTML 문서를 넘기면 브라우저는 HTML 문서 내 코드를 해석하여 사용자에게 화면을 표시하였음. API는 브라우저를 통해 응답하는 것이 아니라, 요청을 주소에 담아 던지면 HTML 파일이 아니라 단순하게 텍스트 데이터의 형태로 응답을 보냄. 좀 더 밀도있는 문자열의 형태로 만들어주는 친구가 serializer임. serializer은 쿼리셋과 모델 인스턴스와 같은 복잡한 데이터를 JSON, XML 또는 다른 컨텐츠의 유형으로 쉽게 변환 할 수 있도록 해줌.

> `DJango`에서 `Client`으로 복잡한 데이터(모델 인스턴스 등)를 보내려면 ‘string’으로 변환해야합니다. 이 변환을 **serializer** 라고 함.

- forms.py 내 모델폼을 작성했던 것과 거의 동일한 형태임. ModelSerializer를 통해 우리가 얻고자 하는 정보(id, title, artist)를 받아옴.
- serializer은 DRF에 내장된 기능이므로 import 할 것
- Music 모델 import
- 외래키인 artist의 id가 들어가는 필드명은 artist_id 이나 artist  중 어떤걸 사용해도 상관없음

```python
from rest_framework import serializers
from .models import Music

class MusicSerializer(serializers.ModelSerializer):
    class Meta:
        model = Music
        fields = ['id', 'title', 'artist']
```



#### 4. views.py 

- serializers.py 내 정의한 `MusicSerializer` import 

- views.py  함수에서는 최종적으로 `response`라는 객체를 반환해야함. (`render` 이나 `redirect` 도 `response` 클래스를 상속받은 메소드임). imoprt 해오자.
- 함수는 항상 응답으로 return 해야함 (Web은 항상 요청과 응답으로 이루어져있음을 기억하자)
- serializer의 첫번째 인자에는 문자열의 형태를 변환시킬 데이터를 넣음.
  - `musics` 에는 Music 모델에 저장된 모든 데이터가 저장된 변수로, 여러개의 객체가 저장되어있으므로 `many=True` 인자를 넘겨줄 것.

```python
from django.shortcuts import render
from .models import Music
form rest_framework.decorators import api_view
# 추가
from .serializers import MusicSerializer
from rest_framework.response import Response


@api_view(['GET'])
def music_list(request):
    musics = Music.objects.all()
    serializer = MusicSerializer(musics, many=True)
    return Response(serializer.data)
```



## Music 디테일 API 구현

#### 1. urls.py

```python
urlpatterns = [
    path('musics/<int:music_id>/', views.music_detail),
]
```



#### 2. views.py

- 동일한 방법으로 함수 구현
- `get_object_or_404` 메서드 사용을 위해 import
- `id=music_id` 을 만족하는 Music 모델의 인스턴스 객체만을 갖고오므로 하나의 객체만 저장되어있음. 따라서 serializer에 `many=True` 인자는 필요 없음.

```python
from django.shortcuts import get_object_or_404

@api_view(['GET'])
def music_detail(request, music_id);
	music = get_object_or_404(Music, id=music_id)
    serializer = MusicSerializer(music)
    return response(serializer.data)
```



## API 자동 문서화 라이브러리 설치

#### 1. settings.py

- `pip install django-rest-swagger` 설치
- INSTALLED_APPS에 추가

```python
INSTALLED_APPS = [
    'rest_framework_swagger',
]
```



#### 2. urls.py

- swagger가 제공하는 view를 갖고와서 import
- `get_swagger_view` 에는  키워드 인자 `title` 사용 시,  해당 페이지의 제목을 설정 할 수 있음. 

```python
from django.urls import path
from . import views
from rest_framework_swagger.views import get_swagger_view

urlpatterns = [
    path('docs/', get_swagger_view(title='API Docs')),
]
```



## Artist 리스트 API 구현

#### 1. urls.py

```python
urlpatterns = [
    path('artists/', views.artist_list),
]
```



#### 2. serializers.py

- `Artist` 모델 import 

```python
from .models import Music, Artist

class ArtistSerializer(serializers.ModelSerializer):
    class Meta:
        model = Artist
        fields = ['id', 'name',]
```



#### 3. views.py

- `ArtistSerializer`  & `Artist` import
- `music_list` 함수와 동일한 방식으로 코드 작성

```python
from .serializers import MusicSerializer, ArtistSerializer #ArtistSerializer import
from .models import Music, Artist #Artist 모델 import

@api_view(['GET'])
def artist_list(request):
    artists = Artist.objects.all()
    serializers = ArtistSerializer(artists, many=True)
    return response(serializer.data)
```



## Artist 디테일 API 구현 & 관계설정

models.py에 구현한 모델에 따르면 `Artist` 모델은 아래와 같이 name 필드만 갖고 있음. 하지만 `Artist`와 `Music` 이 1:N 관계가 형성되어있기 때문에, Artist 디테일 API에서 관계가 형성되어 있는 music 정보도 함께 보여줄 수 있음. 이를 위한 코드를 작성해보자.

```python
class Artist(models.Model):
    name = models.TextField()
	#...중략

class Music(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE)
    title = models.TextField()
	#...중략

```



#### 1. serializers.py 

- `ArtistDetailSerializer`  클래스 정의. 
- `MusicSerializer`를 통해 갖고오는 music 정보는 `music_set`이라는 변수에 저장. 
  - `artist.music_set.all()` 와 같이 1:N 관계에서 1로부터 N으로 접근할 경우 `_set`을 사용함. 이 방법을 사용하여 자식모델에 접근하기 위해 변수명에 `_set`을 넣어서 정의
  -  MusicSerializer의 인자에 변환할 기존의 데이터를 첫번째 인자에 넣지 않는 이유는  views.py 내에서 Artist 모델의 인스턴스 객체인 artist를 ArtistDetailSerializer의 인자로 넘기는데, 해당 객체는 artist_id를 갖고 있으므로, `ArtistDetailSerializer` 내 `MusicSerializer` 의 인자로 변수를 따로 넘길 필요가 없으며, 관계설정에 따라 해당 가수의 음악 목록을 다 갖고 올 수 있음.

```python
class ArtistDetailSerializer(serializers.ModelSerializer):
    music_set = MusicSerializer(many=True)
    class Meta:
        model = Artist
        fields = ['id', 'name','music_set',]
```



#### 2. views.py

```python
@api_view(['GET'])
def artist_detail(Request, artist_id):
    artist = get_object_or_404(Artist, id=artist_id)
    serializer = ArtistDetailSerializer(artist)
    return Response(serializer.data)
```



## Comment API 설정

#### 1. urls.py 

```python
path('musics/<int:music_id>/comments/', views.comment_create),
```



#### 2. serializers.py

- Comment 모델의 외래키인 `music_id` 는 `CommentSerializer` 내 구현할 필요가 없음. `serializer` 는 오롯이 데이터를 가공하는 역할을 할 뿐임.

```python
from .models import Music, Artist, Comment

class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ['id', 'content',]
```



#### 3. views.py

- 댓글 생성은 POST 방식이 적합
- 유저가 입력하는 댓글의 정보는 `request.data`에 저장되어있으므로 serializer의 키워드인자 data에 저장
- 잘못된 요청이 들어왔을 때, 사용자에게 적절한 응답을 보내기 위해 `.is_valid()` 의 인자에 `raise_exception=True` 입력
- 어떠한 music에 대한 대한 댓글인지를 알려줘야함. serializer의 기능으로 save 메서드를 사용가능하며, 이 메서드의 인자로, Comment 모델의 music_id 필드의 값으로 variable routing으로 들어오는 music_id를 입력해주자. 이를 통해 들어온 데이터를 DB에 바로 저장할 수 있음.
- POST 방식으로 요청을 보낼 경우, form 태그를 사용해야하나, template은 따로 구현하지 않았으므로 [POSTMAN](<https://www.getpostman.com/>) 을 활용하자.

```python
from .models import Comment
from .serializers import CommentSerializer

@api_view(['POST'])
def comment_create(request, music_id):
    serializer = CommentSerializer(data=request.data)
    if serializer.is_valid(raise_exception=True):
        serializer.save(music_id=music_id)
        return Response(serializer.data)
```



## Comment 수정/삭제 API 구현

#### 1. urls.py

```python
path('musics/<int:music_id>/comments/<int:comment_id>/', views.comment_update_and_delete),
```



#### 2. views.py

- HTTP Methods에 따라 분기문을 이용해서 수정 & 삭제 구분하여 구현
  - `PUT` : 데이터 수정할 때 사용
  - `DELETE` : 데이터 삭제 할 때 사용
- 어떠한 댓글인지에 대한 정보를 `get_object_or_404` 를 통해 갖고 왔기 때문에, `serializer.save()` 의 인자에 따로 추가적인 값을 넣을 필요는 없음.

``` python
from .models import Comment
form .serializers import CommentSerializer

@api_view(['PUT','DELETE'])
def comment_update_and_delete(request, music_id, comment_id):
    comment = get_object_or_404(Comment, id=comment_id)
    if request.method == 'PUT':
        serializer = CommentSerializer(data=request.data, instance=comment)
        if serializer.is_valid(raise_exception=True):
            serializer.save()
            return Response({'message':'Comment has been updated!'})
    else:
        comment.delete()
    	return Response({'message':'Comment has been deleted!'})
```
