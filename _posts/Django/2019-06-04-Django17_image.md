---
layout: post
title: Django 17 - 미디어 파일 관리 및 업로드 구현
category: Django
tags: [Django]
comments: true
---



# 사용자 권한 관리





# 미디어 파일 관리 및 업로드 구현

### Models.py 수정

- 이미지 업로드를 위한 필드 추가.  
- 빈값을 넣어도 되는 옵션인 `blank=True` 추가

```python
class Post(models.Model):
    content = models.TextField()
    image = models.ImageField(blank=True)
```



### 이미지 재 가공

기존의 ImageField를 그대로 사용할 경우,  이미지 크기를 수정하거나,  저장 확장자 설정 등 이미지 가공에 많은 제약이 있음. 이에, 외부 라이브러리를 사용하여 이미지를 적절하게 재가공할 수 있음

- `pip install pillow` , `pip install pilkit` & `pip install django-imagekit` 설치
- settings.py 내 INSTALLED_APPS 내에 앱 추가

```python
INSTALLED_APPS = [
    'imagekit',]
```



### Models.py 재 수정

- imagekit 사용을 위해 import
  - `from imagekit.models import ProcessedImageField`
  - `from imagekit.processors import ResizeToFill`
- `ProcessedImageField` 의 세부항목
  - `upload_to` : 미디어파일의 저장 위치 설정
  - `processors`  : 업로드시 어떠한 처리과정을 거칠건지에 대한 설정. 처리할 작업 목록을 리스트 형태로 작성 . 안의 내용 수정 시, 추가적으로 `migrate`는 필요 없음
  - `ResizeToFill` : 설정한 사이즈에 딱 맞춰서 이미지 크기 수정. 가로가 긴 경우, 좌,우 영역을 삭제함
  - `ResizeToFit` : 비율 유지한 채로 이미지 크기 수정. 가로가 긴 경우, 사이즈에 맞추고 위 아래 여백이 생김.
  - `options` : 저장 포맷 관련 옵션 (JPEG 압축률 설정)

```python
from imagekit.models import ProcessedImageField
from imagekit.processors import ResizeToFill

class Post(models.Model):
    content = models.TextField()
    # image=models.ImageField(blank=True) 기존코드; ImageField 사용
    image = ProcessedImageField(
               upload_to='posts/images', # 저장 위치
               processors=[ResizeToFill(600,600)], # 처리할 작업 목록
               format='JPEG', # 저장 포맷(확장자)
               options= {'quality': 90 }, # 저장 포맷 관련 옵션 (JPEG 압축률 설정)
        )
```





### 저장 위치 임의 설정하기

- create 페이지에서 모델 폼에 따라 내용이 작성되면, instance가 생성된다. instance에는 데이터에 대한 고유 값과 정보가 담겨 있음. 이를 활용하여 파일 경로를 개별적으로 설정 해 줄 수 있음.
- `filename`은 확장자를 함께 갖고 있음
- `instance.pk` 는 사용이 불가능하다. 이유는 호출되는 시점이 DB에 저장이 되지 않은 시점이기 때문에, 경로를 결정하는 id값을 갖고 있지 않기 때문이다. 따라서 None 폴더에 저장되어 `instance.pk` 는 추천되지 않는 방법임.

```python
def post_image_path(instance, filename): 
    {% raw %}# return f'posts/{instance.content}/{filename}'{% endraw %}
    {% raw %}return f'posts/{instance.content}/{instance.content}.jpg'{% endraw %}

class Post(models.Model):
    content = models.TextField()
    image = ProcessedImageField(
               upload_to = post_image_path, 
               processors = [ResizeToFill(600,600)], 
               format = 'JPEG', 
               options = {'quality': 90 }, 
        )

```

- [reference][https://docs.djangoproject.com/ko/2.2/ref/models/fields/#filefield]

> `upload_to` may also be a callable, such as a function. This will be called to obtain the upload path, including the filename. This callable must accept two arguments and return a Unix-style path (with forward slashes) to be passed along to the storage system. The two arguments are:
>
> | Argument   | Description                                                  |
> | ---------- | ------------------------------------------------------------ |
> | `instance` | An instance of the model where the `FileField` is defined. More specifically, this is the particular instance where the current file is being attached.In most cases, this object will not have been saved to the database yet, so if it uses the default `AutoField`, *it might not yet have a value for its primary key field*. |
> | `filename` | The filename that was originally given to the file. This may or may not be taken into account when determining the final destination path. |
>
> For example:
>
> ```python
> def user_directory_path(instance, filename):
>     # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
>     return 'user_{0}/{1}'.format(instance.user.id, filename)
> 
> class MyModel(models.Model):
>     upload = models.FileField(upload_to=user_directory_path)
> ```



### forms.py 수정

- image 필드 추가

```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['content', 'image']
```



### create.html 수정

- 이미지 파일은 바이너리 형태이므로 `enctype` 속성을 form 태그 내 추가로 입력해줘야함. form 태그는 기본적으로 텍스트 데이터를 전송함
- `enctype` : 데이터를 보낼 때 사용하는 인코딩 유형을 정의할 때 사용하는 속성
  - `application/x-www-form-urlencoded` : form 태그의 기본값으로 텍스트 데이터만 전송 가능
  - `multipart/form-data`: binary 데이터 전송 가능

```html
{% raw %}{% extends 'base.html' %}{% endraw %}
{% raw %}{% load bootstrap4 %}{% endraw %}
{% raw %}{% block container %}{% endraw %}
<h1>New Post</h1>

<form method="post" enctype="multipart/form-data">
    {% raw %}{% csrf_token %}{% endraw %}
    {% raw %}{% bootstrap_form post_form %}{% endraw %}
    <input type="submit" value="Submit"/>
</form>
{% raw %}{% endblock %}{% endraw %}
```



### settings.py 설정

- 이미지를 업로드 할 때 저장되는 기본 경로 설정
- 경로 설정은 기본 템플릿 설정 시 진행했던 방법과 매우 유사함.
- 아래의 코드의 경우, Root Directory 바로 밑 media 폴더 내에 이미지 파일이 저장됨.

```python
#Media
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.joion(BASE_DIR, 'media')
```



### 프로젝트 내 urls.py 수정

- `from django.conf.urls.static import static` : 정적 파일을 제공하기 위한 내장함수
- `from django.conf import settings` : settings.py 내 설정한 미디어 경로를 사용할 수 있게 함
- 미디어 파일을 주소로 설정하는 코드 작성

```python
from django.conf.urls.static import static
from django.conf import settings

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
