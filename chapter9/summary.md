# 9장. 함수 기반 뷰의 모범적인 이용

## 9.1 함수 기반 뷰의 장점

함수 기반 뷰의 단순함은 코드 재사용을 희생하여 나온 결과

> **가이드라인** 
>
> 뷰 코드는 작을수록 좋다
>
> 뷰에서 절대 코드를 반복해서 사용하지 말자
>
> 비지니스 로직은 가능한 한 모델 로직에 적용시키고, 만약 해야 한다면 폼안에 내재시켜야 한다
>
> 뷰를 가능한 한 단순하게 유지하자
>
> 403, 404, 500을 처리하는 커스텀 코드를 쓰는 데 이용하라
>
> 복잡하게 중첩된 if 블록 구문을 피하자

## 9.2 HttpRequest 객체 전달하기

~~~python
# sprinkles/utils.py

from django.core.exceptions import PermissionDenied

def check_sprinkle_rights(request):
    if request.user.can_sprinkle or request.user.is_staff:
        # 탬플릿을 좀 더 일반적으로 사용할 수 있음
        # {% if request.user.can_sprinkle or or request.user.is_staff %} 
        # 대신 {% if request.can_sprinkle %}
        request.can_sprinkle = True
        return request
        
    # HTTP 403을 사용자에게 반환
    raise PermissionDenied
~~~

~~~python
# sprinkle/views.py

from django.shortcuts import get_object_or_404
from djagno.shortcuts import render

from .util import check_sprinkles
from .models import Sprinkle

def sprinkle_list(request):
    """표준 리스트 뷰"""
    
    request = check_sprinkles(request)
    
    return render(request, 
        "sprinkles/sprinkle_list.html", 
        {"sprinkles": Sprinkle.objects.all()})
        
def sprinkle_detail(request, pk):
    """표준 상세 뷰"""
    
    request = check_sprinkles(request)
    
    sprinkle = get_object_or_404(Sprinkle, pk=pk)
    
    return render(request, 
        "sprinkles/sprinkle_detail.html", 
        {"sprinkle": sprinkle})
    
def sprinkle_preview(request):
    """새 스프링클의 미리보기
         check_sprinkles 함수가 사용되는지 확인하지는 않는다.
    """
    sprinkle = Sprinkle.objects.all()
    return render(request, 
        "sprinkles/sprinkle_preview.html", 
        {"sprinkle": sprinkle})
~~~

## 9.3 편리한 데코레이터

간편표기법(syntacitc sugar): 표현이나 가독성을 좋게 하기 위해 프로그래밍 언어에 추가되는 문법을 나타낸다.

~~~python
# 간단한 데코레이터 템플릿
import functools

def decorator(view_func):
    @functools.wraps(view_func)
    def new_view_func(request, *args, **kwargs):
        # 여기에서 request(HttpRequest) 객체를 수정하면 된다.
        response = view_func(request, *args, **kwargs)
        # 여기에서 response(HttpResponse) 객체를 수정하면 된다.
        return response
    return new_view_func
~~~

~~~python
# sprinkles/decorators.py
from functools import wraps

from . import utils

def check_sprinkles(view_func):
    """사용자가 스프링클을 추가할 수 있는지 확인한다."""
    @wraps(view_func)
    def new_view_func(request, *args, *kwargs):
        # request 객체를 utils.can_sprinkle()에 넣는다 
        request = utils.can_sprinkle(request)
        
        # 뷰 함수를 호출
        response = view_func(request, *args, *kwargs)
        
        # HttpResponse 객체를 반환
        return response
    return new_view_func
~~

~~~python
# view.py
from django.shortcuts import get_object_or_494, render

from .decorators import check_sprinkles
from .models import Sprinkles

# 뷰에 데코레이터를 추가
@check_sprinkles
def sprinkle_detail(request, pk):
    """표준 상세 뷰"""
    sprinkle = get_object_or_404(Sprinkle, pk=pk)
    
    return render(request, 
        "sprinkles/sprinkle_detail.html", 
        {"sprinkle": sprinkle})
~~~

> - functools.wraps()는 무엇인가
> docstrings 같은 중요한 데이터를 포함한 메타데이터를 새로이 데코레이션 되는 함수에 복사해 주는 편리한 도구
> [functools.wraps](https://docs.python.org/2/library/functools.html#functools.wraps)

### 9.3.1 데코레이터 남용하지 않기

너무 많은 데코레이터를 사용하면 코드가 난해해지므로, 수를 한정짓자.

## 9.4 HttpResponse 객체 넘겨주기

## 9.5 요약

HttpRequest 객체를 받고 HttpResponse 객체를 반환!


