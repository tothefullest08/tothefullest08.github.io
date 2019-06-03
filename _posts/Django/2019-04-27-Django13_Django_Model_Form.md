---
layout: post
title: Django 13 - Django Model Form
category: Django
tags: [Django]
comments: true
---



# Model Form 

> Model 클래스를 이용하면, 직접 필드를 정의하고 위젯 설정을 할 필요 없이 Model과 연동되어 자동으로 폼 필드를 생성할 수 있음.

### forms.py 작성

- 메타 클래스:  우리가 만들 클래스에 대한 정보가 담긴 클래스 ( `ArticleModelForm` 클래스의 정보가 담김)
  (메타? 데이터를 설명하는 데이터)
  - `model` :  연동하고자 하는 모델 클래스를 저장
  - `fields` : 모델 클래스 내 연동하고자 하는 필드 명을 리스트의 형태로 저장
    - Model 클래스인 `Article`의  필드명 (title, content)를 바탕으로 input값을 입력함.
    - `__all__` : 모든 필드를 가져와서 사용

```python
#forms.py
from .models import Article

class ArticleModelForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']        
        #cf) fields = '__all__'  => 모든 필드를 가져오겠다는 의미. 
        
#models.py
class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
```


### View 수정하기

- 모델 폼 클래스 사용의 가장 큰 이점은 모델과 연동이 된다는 점임. 따라서 폼 클래스를 사용했던 것 처럼, 유효성 검증을 한 데이터를 일일이 입력할 필요 없음. 
- 사용자가 입력한 값이 `request.POST` 에 저장되어있으며,  `ArticleModelForm` 인스턴스의 인자로 넘겨져 `form` 변수에 저장함. 이때, 이미 모델클래스와 연동이 되어 있으므로, 해당 데이터는 자동으로 DB에 생성됨.

> 모델 폼은 모델 클래스의 필드의 속성과 django의 ModelForm 이라는 클래스를 통해서 유효성 검증을 해주며, 에러메시지를 자동으로 생성해줌.  
>
> 예) max_length가 100자로 제한된 경우에  글자를 입력할 경우, 
> "이 값이 최대 100 개의 글자인지 확인하세요(입력값 108 자)." 라는 에러메세지가 뜨게 해줌.

- 따라서, `form.save()` 를 통해 DB에 저장만 한 뒤, `article` 이라는 변수에 저장
- `article`이라는 변수는 detail 페이지로 redirect 하기 위한 `variable routing`으로 사용 

```python
from .forms import ArticleModelForm 

def create(request):
    if request.method == 'POST':
        form = ArticleModelForm(request.POST)
        
        if form.is_valid():
            article = form.save()
            
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleModelForm()
    return render(request, 'create.html', {'form' : form})
```



- 모델 클래스가 아니라 다른 필드명을 추가로 입력하여 오버라이드도 가능
  - 설정을 안할 경우, 모델 클래스의 필드를 그대로 가져와 사용함

```python
#아래의 예시의 경우, title의 이름은 title이 아니라 "제목" 으로 화면에 표시됨.
class ArticleModelForm(forms.ModelForm):
    title = forms.CharField(label="제목")
    class Meta:
        model = Article
        fields = ['title', 'content']
```



<br>

# Form vs Model Form

**모델 폼과 폼의 가장 핵심적인 차이점은 모델(모델 클래스)과의 연동 유무임.**

- 폼은 모델과 연동이 되어있지 않기 때문에, `request.POST` 로 들어오는 `input` 에 대한 정보를 선별하여 각 변수에 저장하고 DB에 생성 및 저장을 해야함.
- 그러나, 모델 폼의 경우, 모델과 연동이 되어있기 때문의 위의 과정을 거칠 필요 없이 DB에 저장만 하면됨.

```python
#Form 클래스
if request.method == 'POST':
    form = ArticleModelForm(request.POST)
	
    if form.is_valid():
        title = form.cleaned_data.get('title')
        content = form.cleaned_data.get('content')
        article = Article.objects.create(title=title, content=content)  
        return redirect('articles:detail', article.pk)
    ...

#ModelForm 클래스
if request.method == 'POST':
    form = ArticleModelForm(request.POST)
	
    if form.is_valid():
        article = form.save()
        return redirect('articles:detail', article.pk)
    ...
```



### 모델 폼을 활용하여 Update 구현하기

- 모델 폼을 사용하면 edit.html 을 따로 생성할 필요 없이 create.html 을 그대로 사용 하면 됨.
- 함수의 반환 값으로 실행되는 html 문서는 글 생성과 수정을 모두 담고 있으므로,  파일명을 form.html으로 변경.

```python
def update(request, article_id):
    article = Article.objects.get(pk=article_id)
    if request.method == 'POST':
        form = ArticleModelForm(request.POST, instance=article)
        
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleModelForm(instance=article)
        
    return render(request, 'form.html', {'form' : form})
```



- 기본키를 바탕으로 데이터 가져와 `article` 이라는 변수에 저장함.

```python
def update(request, article_id):
    article = Article.objects.get(pk=article_id)
```



- 조건문 else  => 기본 폼(수정을 위한 폼) 생성
  - 모델폼의 인스턴스인 `ArticleModelForm` 의 키워드인자 `instance` 의 값으로  `article` 을 넘겨주며,
    `form` 이라는 변수에 저장함.
  - 키워드인자 `instance` 에 `article` 을 넘기게 되면, 기존의 값이 html 파일에 나타남.
    (article에 대한 정보를 모델 폼에 넘겨줌. 폼태그의 value와 같은 역할)

```python
else:
    form = ArticleModelForm(instance=article) 
return render(request, 'form.html', {'form' : form})
```



- 조건문 if => 폼데이터를 수정
  - `request.POST` 에는 사용자가 입력하는 input의 정보가 저장되어 있음
  - 키워드 인자 `instance`  에는 수정하고자 하는 데이터의 기본 정보가 담겨있음. 이를 통해 어떠한 객체에 대한 정보를 수정할껀지 그 주소를 알 수 있음.
  - 폼 인스턴스 `ArticleModelForm()` 의 인자로 `request.POST`  &  `article` 을  넘겨  `form`  변수에 저장함  => 모델 폼에서 정의된 구성에 따라 넘겨받은 정보를 분석하여 정리함.
  - 변수 `form `을 DB 저장 한 후, 디테일 페이지로 `redirect` 시킴.

```python
if request.method == 'POST':
    form = ArticleModelForm(request.POST, instance=article)

    if form.is_valid():
        article = form.save()
        return redirect('articles:detail', article.pk)
```



### 템플릿을 통해  HTML 폼 최종 수정

- `form` 은 모델폼의 인스턴스로, 용도에 따라 아래와 같이 view에 정의되어있음
  - `ArticleModelForm()`  : Create 실행 시
  - `ArticleModelForm(instance=article)`  : update 실행 시
- `{% raw %}{{ form }}{% endraw %}`  으로 접근할 경우. 모든 input 값이 뭉쳐져있어서,  세부적인 커스터마이징에 어려움이 있음.
- 뭉쳐진 form을 세분화 시키는 방법을 폼 인스턴스에서 제공함
  - `form.non_field_errors` : 필드에 국한되지 않은 포괄적인 전체 에러를 보여줄 때 설정
  - `form.title.errors` : title 필드의 error 메세지 표시
  - `form.title.lable_tag` : title 필드의 label(이름) 표시
  - `form.title` : title에 대한 input 상자 표시
- `form.as_p` : 각각의 폼필드와 라벨을 `p`  태그로 감싸서 출력
- `form.as_table` :  각각의 폼 필드와 라벨을 `tr` 태그로 감싸서 출력
- `form.as_ul` : `ul & li ` 태그로 감싸서 출력 

```html
{% raw %}{{ form.non_field_errors }}

<form method="POST">
    {% csrf_token %}
    <!--<input type="text" name="title" required/>-->
    <!--<input type="text" name="content" required/>-->

    <!--1. title -->
    <div>
        {{ form.title.errors }}  <!-- error message (ul, li tag) -->
        {{ form.title.label_tag }}  <!-- label tag -->
        {{ form.title }}  <!-- input tag -->
    </div>

    <!--2. content -->
    <div>
        {{ form.content.errors }}
        {{ form.content.label_tag }}
        {{ form.content }}
    </div>
    <input type="submit" value="Submit"/>
</form>{% endraw %}
```

​                                                                                                      