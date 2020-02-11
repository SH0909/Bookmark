# Django로 Bookmark만들기

## 1.프로젝트 만들기
1. 가상환경 설정  
 python -m venv venv   
source venv/Scripts/activate
2. 장고 설치   
pip install django
3. 장고 프로젝트 생성   
django-admin startproject config .
4. 데이터 베이스 초기화   
python manage.py migrate
5. 관리자 계정 생성   
python manage.py createsuperuser

## 2.북마크 앱 생성하기
 python manage.py startapp bookmark

## 3.모델 만들기
모델이란? 데이터베이스 사용을 쉽게 하기 위해 사용하는 도구이다 데이터베이스에 저장해서 사용해야겠다는 데이터가 있다면 모델을 만들어야 한다
* bookmark/models.py
```
from django.db import models

class Bookmark(models.Model): //models.Model을 상속받는 클래스 생성
    site_name=models.CharField(max_length=100)
    url=models.URLField('Site URL')
```
데이터베이스에 사이트 이름과 url을 저장하기 위해 두 개의 필드를 생성함
이 정보가 기록되는 테이블 이름은 bookmark
* config/settings.py에 bookmark 앱 추가
```
INSTALLED_APPS=[
    'bookmark',
]
```
* 데이터베이스 설정   
1. python manage.py makemigrations bookmark   
마이그래이션 파일 :데이터베이스 변경사항이 있는지 확인하고 변경 내용이 있으면 파일 생성 
2. 마이그레이션 파일 내용 실제 데이터베이스에 적용   
python manage.py migrate bookmark

## 4.관리자 페이지에 모델 등록
* bookmark/admin.py
```
from django.contrib import admin
from .models import Bookmark //models.py에서 Bookmark라는 모델을 불러옴

admin.site.register(Bookmark) //Bookmark모델 등록
```
admin.py는 모델을 관리자 페이지에 등록해 관리할 수 있도록 하는 역할과 관리자 페이지에서 보이는 내용의 변경,기능 추가 등을 할 수 있도록 코드를 입력하는 파일
## 5.모델에 __str__메소드 추가
*언더바가 앞뒤로 두 개씩 붙어있는 함수들을 매직 메소드라고 부른다 (특별한 기능이 있음)   
```__str__메소드는 클래스의 오브젝트를 출력할 때 나타나는 내용을 결정```
* boomark/models.py 북마크 모델에 메소드 추가
```
def __str__(self):
        return "이름: "+self.site_name+" 주소: "+self.url
```
__str__메소드 반환값이 화면에 출력된다

## 6.목록 뷰 만들기
* bookmark/views.py
```
from django.views.generic.list import ListView
from .models import Bookmark

class BookmarkListView(ListView):
    model=Bookmark //Bookmark모델 설정
```

## 7.URL 연결하기
1. 어떤 주소를 입력했을때 해당 페이지를(뷰) 보여주고 싶은지 설정   
3. 앱에 관한 url연결은 앱 폴더에 있는 urls.py에 설정 (한번 만든 앱은 다른 프로젝트에서도 재사용할 수 있기 때문)   
3. 앱에 관한 urls.py 내용은 루트 파일에서 연결해줘야 한다
* config/urls.py
```
from django.contrib import admin
from django.urls import path ,include

urlpatterns = [
    path('',include('bookmark.urls')), //bookmark.urls을 연결하는 path추가
    path('admin/', admin.site.urls),
]
```
```.../bookmark/이하url/ 주소로 접속하면 bookmark 까지의 url을 잘라내고 나머지 부분을 bookmark.urls로 전달/나머지 부분을 가지고 어떤 뷰를 연결할지 bookmark앱에 있는 urls.py에 작성```
* bookmark/urls.py
```
from django.urls import path
from .views import BookmarkListView

urlpatterns=[
    path('',BookmarkListView.as_view(),name='list'), 
]
```
```클래스 형 뷰는 as_view() 꼭 붙여야 함 name 설정을 가지고 해당 url패턴을 찾을 수 있다```

## 8.bookmark_list.html
1. 템플릿이란? 프론트엔드 소스코드가 저장되는 파일들이면서 장고에서 데이터를 껴 넣는 양식 파일
2. 템플릿 위치   
앱 폴더 내부에 templates폴더 생성->앱 이름으로 폴더 한번 더 생성->템플릿 파일 생성 ```bookmark/templates/bookmark/~.html```
* bookmark/bookmark_list.html
```
<div class="btn-group">
        <a href="#" class="btn btn-info">Add Bookmark</a> //북마크 추가 링크
    </div>
    <table class="table"> //북마크 목록 출력
        <thead>
            <tr>
                <th scope="col">#</th>
                <th scope="col">Site</th>
                <th scope="col">URL</th>
                <th scope="col">Modify</th>
                <th scope="col">Delete</th>
            </tr>
        </thead>
        <tbody>
            {% for bookmark in object_list %} //제너릭 뷰에서는 모델의 오브젝트가 여러개일 경우 object_list변수로 전달
            <tr>
                <td>{{forloop.counter}}</td>
                <td><a href="{% url 'detail' pk=bookmark.id %}">{{bookmark.site_name}}</a></td>
                <td><a href="{{bookmark.url}}" target="_blank">{{bookmark.url}}</a></td> //url누르면 새창에서 열림
                <td><a href="{% url 'update' pk=bookmark.id %}" class="btn btn-success btn-sm">Modify</a></td>
                <td><a href="{% url 'delete' pk=bookmark.id %}" class="btn btn-danger btn-sm">Delete</a></td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
```
* bootstrap
1. class 값으로 btn btn-xxx를 추가하면 버튼이 꾸며진다 ex)info(하늘색),success(초록색),danger(빨간색)..
2. btn-outline-info 하면 테두리에 색 생김
3. btn-sm :작은 버튼

## 9.북마크 추가 기능 구현
* bookmark/views.py
```
from django.views.generic.edit import CreateView
from django.urls import reverse_lazy

class BookmarkCreateView(CreateView):
    model=Bookmark
    fields=['site_name','url']
    success_url=reverse_lazy('list') 
    template_name_suffix='_create' 
```
1. fields 변수 :어떤 필드들을 입력받을 것인지 설정하는 부분
2. success_url :글쓰기를 완료하고 이동할 페이지
3. template_name_suffix :사용할 템플릿의 접미사만 변경 (bookmark_create라는 템플릿 사용)
```CreateView와 UpdateView는 form이 접미사임```
* bookmark/urls.py (북마크 추가 뷰와 url연결)
```
from .views import BookmarkListView,BookmarkCreaeView
urlpatterns=[
    path('',BookmarkListView.as_view(),name='list'),
    path('add/',BookmarkCreateView.as_view(),name='add'),
]
```
* bookmark/bookmark_create.html
```
<form action="" method="post"> 
        {%csrf_token%}
        {{form.as_p}} 
        <input type="submit" value="Add" class="btn btn-info btn-sm">
    </form>
```
1. action="" :현재 페이지로 자료 전달
2. {%csrf_token%} :CSRF 공격 막기 위해
3. form.as_p :클래스 형 뷰의 옵션 값으로 설정한 필드를 출력하는데 각 필드 폼 태그들을 p태그로 감싸 출력 (사이트 이름,사이트 url)

* bookmark/bookmark_list.html (Add bookmark 링크 작동)
```
<a href="{% url 'add %}" class="btn btn-info">Add Bookmark</a>
```
url 템플릿 태그 사용/add라는 이름을 가진 url패턴을 찾아 url 출력해라

## 10.북마크 확인 기능 구현
* bookmark/views.py
```
from django.views.generic.detail import DetailView

class BookmarkDetailView(DetailView):
    model=Bookmark
```
* bookmark/urls.py (뷰를 url과 연결)
```
from .views import *

urlpatterns=[
    path('',BookmarkListView.as_view(),name='list'),
    path('add/',BookmarkCreateView.as_view(),name='add'),
    path('detail/<int:pk>/',BookmarkDetailView.as_view(),name='detail'),
]
```
1. int :컨버터(타입)
2. pk :컨버터를 통해 반환받은 값/패턴에 일치하는 값의 변수명
* bookmark/bookmark_detail.html
```
{{object.site_name}}<br> 
{{object.url}}
```
```ListView는 object_list를 쓰고 DetailView는 object를 쓰는군...```
* bookmark_list.html에 디테일 연결
```
<a href="{% url 'detail' pk=bookmark.id%}">{{bookmark.site_name}}</a>
```
```pk에 id값 주는거 잊으면 안됨```

## 11.북마크 수정 기능 구현
* bookmark/views.py
```
from django.views.generic.edit import UpdateVIew

class BookmarkUpdateView(UpdateView):
    model=Bookmark
    fields=['site_name','url']
    template_name_suffix='_update'
```
1. 입력받을 필드 설정
2. bookmark_update.html 템플릿 설정

* bookmark/urls.py (업데이트 뷰와 url연결)
```
path('update/<int:pk>',BookmarkUpdateView.as_view(),name='update') //추가
```
* bookmark/bookmark_update.html
```
<form action="" method="POST">
        {%csrf_token%}
        {{form.as_p}}
        <input type="submit" value="Update" class="btn btn-info btn-sm">
    </form>
```
create와 거의 유사
* bookmark_list.html에 업데이트 연결
```
<a href="{% url 'update' pk:bookmark.id %}" class="btn btn-success btn-sm">Modify</a>
```
* 작업을 완료 한 후 이동할 페이지는 뷰에 success_url이 설정되어 있거나 모델에 get_absolute_url이라는 메소드를 통해 결정
* bookmark/models.py (get_absolute_url사용해보기)
```
from django.urls import reverse
class Bookmark(models.Model):
    def get_absolute_url(self):
        return reverse('detail',args=[str(self.id)]) 
```
```reverse :url패턴 이름과 추가 인자를 전달받아 url 생성, 객체 상세화면 주소 반환``` 수정하면 디테일 화면으로 이동 bookmark/detail/id/주소

## 12.북마크 삭제 기능 구현
* bookmark/views.py
```
from django.views.generic.edit import DeleteView

class BookmarkDeleteView(DeleteView):
    model=Bookmark
    success_url=reverse_lazy('list')
```
```삭제 후 목록 페이지로 이동```
* bookmark/urls.py (삭제 뷰와 url을 연결)
```
path('delete/<intLpk>',BookmarkDeleteView.as_view(),name='delete'),
```
* bookmark_list.html에 삭제 버튼 연결
```
<a href="{% url 'delete' pk=bookmark.id %}" class="btn btn-danger btn-sm">Delete</a>
```
* bookmark_confirm_delete.html   
```템플릿 이름이 왜 저래? 삭제 할 때는 확인이 필요하기때문에 confirm_delete라는 이름을 사용한다```
```
<form action="" method="POST">
        {% csrf_token %}
        <div class="alert alert-danger">Do you want to delete Mookmark"{{object}}"?</div>
        <input type="submit" value="Delete" class="btn btn-danger">
    </form>
```
alert클래스는 메세지 칸을 만들어줌 alert alert-danger:빨간색 success,info,warning등이 있음..

# 디자인 입히기
## 1.템플릿 확장하기
기준이 되는 레이아웃 부분을 담은 템플릿을 별도로 만들어두고 상속받아 사용하는것처럼 재사용하는것 /공통적인 코드를 계속 작성하면 비효율적이기 때문
1. 프로젝트 루트에 templates 폴더 추가하고 base.html 파일 만들어줌
2. settings.py를 변경해 우리가 만든 폴더에 저장된 템플릿 파일 사용할 수 있도록(base.html)변경   
```TEMPlATES변수에 DIRS키에 os.path.join(BASE_DIR,"templates")추가```
* base.html
```
<head>
    <title>{% block title %}</title>
</head>
<body>
    {% block content %}
    {% endblock %}
</body>
```
1. 템플릿 확장은 BLOCK을 기준으로 동작한다
2. base.html에는 다른 템플릿에서 끼워넣을 공간을 block태그를 사용해 만들어두고 하위 템플릿에서는 이 블록에 끼워넣을 내용을 채움   

```하위 템플릿에 block title과 block content를 넣어 수정 맨윗줄에 반드시 {% extends 'base.html' %} 있어야 함```

## 2.부트스트랩 적용하기 (부트스트랩은 나중에 더 공부하기)
1. 부트스트랩이란? css 프레임워크의 한 종류
부트스트랩을 사용하면 html태그에 class 속성을 추가하는 것만으로도 페이지를 꾸밀 수 있음
2. 사이트에서 css,js 파일 불러오기   
->base.html의 head태그 안쪽에 링크 작성
* 메뉴바 만들기 base.html
```
<div class="container">
        <nav class="navbar navbar-expand-lg navbar-light bg-light">
            <a class="navbar-brand" href="#">Django Bookmark</a>
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarSupportedContent">
                <ul class="navbar-nav mr-auto">
                    <li class="nav-item active">
                        <a class="nav-link" href="#">Home<span class="sr-only">(current)</span></a>
                    </li>
                </ul>
            </div>
        </nav>
        <div class="row">
            <div class="col">
                {% block content %}
                {% endblock %}
                {% block pagination %}
                {% endblock %}
            </div>
        </div>
    </div>
```
1. 네비게이션 바는 navbar 클래스 추가해야 함
2. 보여질 화면 크기에따라    navbar-expand-xl,lg,md,sm
3. 네비게이션 바 배경색 바꾸기   
bg-dark,light,success,danger,info등..
4. navbar-light :검정 글씨 dark :하얀 글씨
5. navbar-brand :특별히 강조하고 싶은 메뉴   

```화면 너비가 넓으면 navbar-collaps가 표시되고 toggle은 숨겨짐 화면 너비 좁아지면 collapse가 숨겨지고 toggle이 보임```   
6. toggler (메뉴 숨겨놓는 창)   
```
<button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
    <span class="navbar-toggler-icon"></span>
</button>
```
7. toggle의 data-target과 collapse의 id 이름은 같아야 한다
8. navbar-nav :네비게이션 메뉴의 요소 /mr-auto :margin right auto
9. 수평 메뉴 만들기 :nav 클래스를 ul 요소에 추가하고 각 li에 대해 nav-item 추가하고 링크에 nav-link 클래스 추가 active는 활성화/disabled는 비활성화로 안 눌러짐
10. span class=sr-only :글씨가 화면에 보이지 않음

## 3.페이징 기능 만들기
1. views.py에 BookmarkListView에 paginate_by=6이라는 코드 추가   
->한 페이지에 몇 개씩 출력할지 결정
* bookmark_list.html 제일 아래쪽에 추가
```
{% block pagination %}
    {% if is_paginated %}
        <ul class="pagination justify-content-center pagination-sm">
            {% if page_obj.has_previous %}
            <li class="page-item">
                <a class="page-link" href="{% url 'list' %}?page={{page_obj.previous_page_number}}" tabindex="-1">Previous</a>
            </li>
            {% else %}
            <li class="page-item disbaled">
                <a class="page-link" href="#" tabindex=-1>Previous</a>
            </li>
            {% endif %}

            {% for object in page_obj.paginator.page_range %}
            <li class="page-item {% if page_obj.number == forloop.counter %}disbaled{% endif %}">
                <a class="page-link" href="{{request.path}}?page={{forloop.counter}}">{{forloop.counter}}</a>
            </li>
            {% endfor %}

            {% if page_obj.has_next %}
            <li class="page-item">
                <a class="page-link" href="{%url 'list'%}?page={{page_obj.next_page_number}}">Next</a>
            </li>
            {% else %}
            <li class="page-item disbaled">
                <a class="page-link" href="#">Next</a>
            </li>
            {% endif %}
        </ul>
    {% endif %}
{% endblock %}
```
```클래스형 뷰에서 paginate_by값을 사용하면 자동으로 Page 객체를 생성해 이전,이후,현재 페이지등을 알 수 있다```
1. justify-content-center :중앙 정렬 
2. pagination-sm :크기 작게
3. page_obj.has_previous :이전 페이지 있는지 확인
4. 
```
href="{% url 'list' %}?page={{page_obj.previous_page_number}}" /?page=1 이런식으로 주소 생성
```
tabindex=-1 :초점을 맞추지 않는다 (없어도 될듯..)

## 4.정적(static)파일 사용하기
1. 정적 파일이란? 로컬 서버에 있는 여러가지 파일이다 css,js,img등..
2. 앱 폴더 밑에 static 폴더 사용하기 or
루트에 static 폴더 만들고 settings.py 설정 변경
* config/settings.py   
```STATICFILES_DIRS=[os.path.join(BASE_DIR,'static')]```
* base.html에서 style.css 파일 불러오기
base.html head태그 안에
```
{% load static %}
<link rel="stylesheet" href="{% static 'style.css' %}">
```

# 배포하기-Pythonanywhere
1. 깃허브 가입
2. 깃허브에 소스코드 업로드
3. 파이썬 애니웨어 가입
4. 콘솔창에서(파이썬 애니웨어) 소스코드 다운로드    
git clone 주소
5. 다운받은 폴더로 이동
6. 가상환경 만들기   
virtualenv venv --python=python3.7
7. 가상환경 활성화   
source venv/bin/activate
8. 장고 설치   
pip install django
9. 데이터베이스 초기화   
python manage.py migrate
10. 관리자 계정 생성
11. web으로 들어가 add a new web app클릭
12. 가상환경 연결   
Virtualenv에 경로 입력 ```home/sh0909/bookmark/venv (본인 계정 쓰기)```
13. 정적파일 관리   
STATIC_ROOT라는 변수로 설정해둔 경로에 모아두고 웹 서버에서 이 파일을 찾아보도록 설정하는게 일반적임   
settings.py에```STATIC_ROOT=os.path.join(BASE_DIR,'static_files')추가```
14. 콘솔에서(파.웨) 정적파일 한곳으로 모으기   
python manage.py collectstatic
->STATIC_ROOT 경로에 정적 파일들이 모아짐
15. 웹앱페이지 Static files부분 url과 directory 입력   
```url=/static/  directory=/home/sh0909/bookmark/static_files/```
16. wsgi파일 수정
```
import os
import sys
path="/home/sh0909/bookmark"
if path not in sys.path:
    sys.path.append(path)
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGs_MODUlE","config.settings")
application=get_wsgi_application()
```
* 배포할때는 Debug=False ALLOWED_HOSTS=['*']