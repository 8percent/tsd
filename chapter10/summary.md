# 클래스 기반 뷰의 모범적인 이용

### django의 view 
요청 객체를 받고 응답 객체를 반환하는 내장 함수

### 함수 기반 뷰
뷰 함수 자체가 내장 함수

### 클래스 기반 뷰
뷰 클래스가 내장 함수를 반환하는 as_view() 클래스 메서드를 제공.  
django.views.generic.View 에서 아래와 같은 매커니즘이 구현되고, 모든 클래스 기반 뷰는 이 클래스를 직간접적으로 상속받아 이용한다. 

- as_view()에서 클래스의 인스턴스를 생성 
- 인스턴스의 dispatch() 메서드 호출
- dispatch() 메서드에서 HTTP GET/POST 메서드를 알아냄
- 인스턴스 내 해당 메서드로 중개를 한다
- 메서드가 정의되어 있지 않으면 HttpResponseNotAllowed 예외 발생

### 제네릭 클래스 기반 뷰
장고는 대부분의 웹 프로젝트에서 공통적으로 사용되는 부분을 추상화하여 제네릭 클래스 기반 뷰로 제공한다. 

### django-braces 라이브러리
[django-braces Document](https://django-braces.readthedocs.io/en/latest/)
> **장고의 제네릭 클래스 기반 뷰에서 빠진 부분**  
> 장고의 기본형에는 제네릭 클래스 기반 뷰를 위한 중요한 믹스인들이 빠져있다. (eg.Authentication)  
> `django-braces` 라이브러는 제네릭 클래스 기반 뷰를 쉽고 빠르게 개발하기 위한 명확한 믹스인들을 제공한다.

### django.contrib.auth.views  
django 1.11에서 사용할 수 있다. 

- LoginView
- LogoutView
- PasswordChangeDoneView
- PasswordChangeView
- PasswordResetCompleteView
- PasswordResetConfirmView
- PasswordResetDoneView
- PasswordResetView

[ccbv 참고](http://ccbv.co.uk/)


## 10.1 클래스 기반 뷰를 이용할 때의 가인드라인
- 뷰 코드의 양은 적으면 적을수록 좋다.
- 뷰 안에서 같은 코드를 반복적으로 이용하지 말자.
- 뷰는 프레젠테이션 로직에서 관리하도록 하자. 비지니스 로직은 모델에서 처리하자. 매우 특별한 경우에는 폼에서 처리하자.
- 뷰는 간단 명료해야한다.
- 403, 404, 500 에러 핸들링에 클래스 기반 뷰는 이용하지 않는다. 대신 함수 기반 뷰를 이용하자.
- 믹스인은 간단 명료해야한다.

## 10.2 클래스 기반 뷰와 믹스인 이용하기
### 믹스인
실체화된 클래스가 아닌, 상속해 줄 기능들을 제공하는 클래스를 의미함.
다중 상속을 해야 할 때, 믹스인을 사용하면 클래스에 더 나은 역할과 기능을 제공할 수 있다.

> 실체화 = 인스턴스화(instantiation)  
> 클래스의 객체에 대해 메모리를 할당하여 인스턴스를 만드는 것

#### 상속에 관한 규칙
케네스 러브(Kenneth Love)가 제안한 상속에 관한 규칙  
파이썬의 메서드 처리 순서(MRO)에 기반을 둔 것으로, 왼쪽에서 오른쪽의 순서로 처리한다고 표현할 수 있음

- **규칙1.** 장고가 제안하는 기본 뷰는 '항상' 오른쪽으로 진행한다.  
- **규칙2.** 믹스인은 기본 뷰에서부터 왼쪽으로 진행한다.  
- **규칙3.** 믹스인은 파이썬의 기본 객체 타입을 상속해야만 한다.

#### 예제

~~~python 
from django.views.generic import TemplateView

class FreshFruitMixin(object):

	def get_context_data(self, **kwargs):
		context = super(FreshFruitMixin, self).get_context_data(**kwargs)
		context['has_fresh_fruit'] = True
		return context

class FruityFlavorView(FreshFruitMixin, TemplateView):
	template_name = 'fruity_flavor.html'
~~~

FruityFlavorView 클래스는 FreshFruitMixin과 TemplateView를 상속하고있다.

- **규칙1.** TemplateView는 장고에서 제공하는 기본 클래스이기 때문에 가장 오른쪽에 위치한다.  
- **규칙2.** FreshFruitMixin은 TemplateView 왼쪽에 위치한다.
- **규칙3.** FreshFruitMixin은 object를 상속하고 있다.

### MRO | Method Resolution Order
다중 상속을 가진 클래스에서 사용할 올바른 메소드를 찾기 위해 Python에서 사용하는 클래스 검색 경로를 정의하는 것.

![mro](./imgs/mro.png)

![old-new](./imgs/old-new.png)  

![new](./imgs/new.png)  

> 1. python3 에서는 각 클래스가 기본적으로 파이썬의 기본 객체 `object`를 상속하기 때문에 명시하지 않아도 된다.  
> 2. `model.__mro__` 또는 `model.mro()`를 사용하면 어떤 경로로 메소드를 찾고 있는지 확인할 수 있다. (new style에서만 적용됨)

## 10.3 어떤 장고 제네릭 클래스 기반 뷰를 어떤 태스크에 이용할 것인가?
제네릭 클래스 기반 뷰의 장점은 단순화를 희생해서 얻은 결과다.  
제네릭 클래스 기반 뷰는 최대 8개의 슈퍼클래스가 상속되기도 하는 복합적인 상속 체인이다.

[Built-in class-based views API](https://docs.djangoproject.com/en/1.11/ref/class-based-views/)

- Base views
	- View
	- TemplateView
	- RedirectView

- Generic display views
	- DetailView
	- ListView


- Generic editing views
	- FormView
	- CreateView
	- UpdateView
	- DeleteView

- Generic date views
	- ArchiveIndexView
	- YearArchiveView
	- MonthArchiveView
	- WeekArchiveView
	- DayArchiveView
	- TodayArchiveView
	- DateDetailView


#### 장고 클래스 기반 뷰의 이용 표

| 이름 | 목적 |
| --- | --- |
| View | 어디에서든 이용 가능한 기본 뷰 | 
| RedirectView | 사용자를 다른 URL로 리다이렉트 | 
| TemplateView | 장고 HTML 템플릿을 보여줄 때  | 
| ListView | 객체 목록 | 
| DetailView | 객체를 보여줄 때 | 
| FormView | 폼 전송 | 
| CreateView | 객체를 만들 때 | 
| UpdateView | 객체를 업데이트 할 때 | 
| DetailView | 객체를 삭제 | 
| generic date view | 시간 순서로 객체를 나열해 보여줄 때 | 


### 장고 클래스 기반 뷰/제네릭 클래스 기반 뷰의 이용에 대한 세 가지 의견
#### 1. 제네릭 뷰의 모든 종류를 최대한 이용하자
작업양을 최소화 하기 위해 제네릭 뷰가 제공하는 모든 종류의 뷰를 최대한 이용하기를 장려한다.
#### 2. 심플하게 django.views.generic.View 하나로 모든 뷰를 처리하자 
장고의 기본 클래스 뷰로도 충분히 원하는 기능을 소화할 수 있다.   
진정한 클래스 기반 뷰란 모든 뷰가 제네릭 클래스 기반 뷰이어야 한다.  
1번의 리소스 기반 접근 방식이 실패한 난해한 태스크에 대해서 효율적이다.
#### 3. 뷰를 정말 상속할 것이 아닌 이상 그냥 무시하자
읽기 쉽고 이해하기 쉬운 FBV로 시작하고, 반드시 필요한 경우에만 CBV를 이용하자.


## 10.4 장고 클래스 기반 뷰에 대한 일반적인 팁

### 10.4.1 인증된 사용자에게만 장고 클래스 기반 뷰/제네릭 클래스 기반 뷰 접근하게 하기

### 10.4.2 뷰에서 유효한 폼을 이용하여 커스텀 액션 구현하기

### 10.4.3 뷰에서 부적합한 폼을 이용하여 커스텀 액션 구현하기 

### 10.4.4 뷰 객체 이용하기 


## 10.5 제네릭 클래스 기반 뷰와 폼 사용하기

### 10.5.1 뷰 + 모델폼 예제

### 10.5.2 뷰 + 폼 예제 


## 10.6 django.views.generic.View 이용하기 

## 10.7 추가 참고 자료

## 10.8 요약