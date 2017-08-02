8.함수 기반 뷰와 클래스 기반 뷰
 ===============================
 
 -	장고 1.8은 **함수 기반 뷰-FBV**와 **클래스 기반 뷰-CBV**를 둘다 지원한다.
 -	이 두 가지 타입을 어떻게 이용해야 할 지에 대해 설명한다.
 
 8.1 함수 기반 뷰와 클래스 기반 뷰를 각각 언제 이용할 것인가?
 ------------------------------------------------------------
 
 -	매번 뷰를 구현할 때마다 어떤 것이 적당할지 고민해야 한다.
 -	어떤 뷰를 사용해야 하는지 잘 모르겠다면 아래의 순서도의 도움을 받아보길 바란다. ![flowchart](http://i.imgur.com/lAu1qcO.jpg)
 
 > 순서도는 클래스 기반 뷰를 선호하는 필자의 취향을 기반으로 그려졌다.
 > - 클래스 뷰로 구현했을 때 더 복잡해지는 경우, 커스텀 에러 뷰들에 대해서만 함수 기반 뷰를 이용하고 있다.
 >
 > 클래스 기반 뷰의 사용이 익숙하지 않다면 함수 기반 뷰를 사용해도 전혀 문제 될 것이 없다.
 > - 어떠한 개발자들은 대부분의 뷰를 함수 기반 뷰로 처리하며, 클래스 기반 뷰는 서브클래스가 필요한 경우에만 제한적으로 이용한다.
 
 8.2 URLConf로부터 뷰 로직 분리하기
 ----------------------------------
 
 -	뷰와 URL의 결합은 최대한의 유연성을 제공하기 위해 느슨하게 구성되어야 한다.
 
 	-	뷰 모듈은 뷰 로직을 포함해야한다.
 	-	URL 모듈은 URL 로직을 포함해야한다.
 
 -	**나쁜 예제**
 
 ```python
 from django.conf.urls import url
 from django.views.generic import DetailView
 
 from tastings.models import Tasting
 
 urlpatterns = [
     url(r"^(?P<pk>\d+)/$",
       DetailView.as_view(
         model=Tasting,
         template_name="tastings/detail.html"),
       name="detail"),
     url(r"^(?P<pk>\d+)/result$",
       DetailView.as_view(
         model=Tasting,
         template_name="tastings/result.html"),
       name="result")
 ]
 ```
 
 -	뷰와 url, 모델 사이에 단단하게 종속적인 결합이 되어있어, 뷰에서 정의된 내용이 **재사용**되기 어렵다.
 -	같거나 비슷한 인자들이 계속 이용되어 **반복되는 작업을 하지 말라**는 철학에 위배된다.
 -	URL들의 무한한 확장성이 파괴되어 **클래스 상속**이 불가능하게 되었다.
 
 8.3 URLConf에서 느슨한 결합 유지하기
 ------------------------------------
 - **뷰 파일 추가**
 
 ```python
 # tastings/views.py
 from django.views.generic import ListView, DetailView, UpdateView
 from django.core.urlresolvers import reverse
 
 from .models import Tasting
 
 class TasteListView(ListView):
 	model = Tasting
 
 class TasteDetailView(DetailView):
 	model = Tasting
 
 class TasteResultsView(TasteDetailView):
 	template_name = "tastings/results.html"
 
 class TasteUpdateView(UpdateView):
 	model = Tasting
 
 	def get_success_url(self):
  	return reverse("tastings:detail",
 			kwargs={"pk": self.object.pk})
 ```
 - **url 정의**
 
 ```python
 # tastings/urls.py
 from django.conf.urls import url
 from . import views
 urlpatterns = [
 	url(
 		regex = r"^$",
 		view=views.TasteListView.as_view(),
 		name="list"
 	),
 	url(
 		regex = r"^(?P<pk>\d+)/$",
 		view=views.TasteDetailView.as_view(),
 		name="detail"
 	),
 	url(
 		regex = r"^(?P<pk>\d+)/result/$",
 		view=views.TasteResultsView.as_view(),
 		name="result"
 	),
 	url(
 		regex = r"^(?P<pk>\d+)/update/$",
 		view=views.TasteUpdateView.as_view(),
 		name="update"
 	)
 ]
 ```
 
 - **반복되는 작업 하지 않기** : 뷰들 사이에 인자나 속성이 중복되지 않는다.
 - **느슨한 결합** : URLConf로부터 모델과 템플릿 이름을 전부 제거했다.
 - URLConf는 **URL 라우팅**이라는 한 가지 명확한 작업만 처리한다.
 - **상속**이 가능하다.
 - **무한한 유연성**을 제공한다.
 
 ### 8.3.1 클래스 기반 뷰를 사용하지 않는다면?
 
 -	함수 기반 뷰를 사용해도 URLConf로 부터 **뷰 로직을 분리**하자.
 
 8.4 URL 이름공간 이용하기
 -------------------------
 
 -	URL 이름공간은 앱 레벨 또는 인스턴스 레벨에서의 구분자(:)를 제공한다.
 
 -	**사용방법**
 
 	- urls.py : <br/>
 		tastings_detail이라고 name을 정의하는 대신 namespace를 tastings (tastings:detail)이라고 정의한다.
 		```python
 		# 프로젝트 루트에 있는 urls.py
 		urlpatterns+= [
 		  url(r'^tastings/', include('tastings.urls', namespace='tastings')),
 		]
 		```
 	- views.py
 	```python
 	# tastings/views.py 코드조각
 	class TasteUpdateView(UpdateView):
 		model = Tasting
 
 		def get_success_url(self):
 			return reverse("tastings:detail",
 				kwargs={"pk":self.object.pk})			
 	```
 	- template
 	```python
 	{% for taste in tastings %}
 	<li>
 		<a href="{% url "tastings:detail" taste.pk}">{{taste.title}}</a>
 		<small>
 			(<a href="{% url "tastings:update" taste.pk}">update</a>)
 		</small>
 	</li>
 	{% endfor %}
 	```
 
 ### 8.4.1 URL 이름을 짧고 명확하고, 반복되는 작업을 피해서 작성하는 방법
 
 -	모델의 이름이나 앱의 이름을 복사한 URL 대신 좀 더 명확한 이름이 보인다.
 -	코드를 입력하는 시간을 절약할 수 있다.
 
 ### 8.4.2 서트 파티 라이브러리와 상호 운영성을 높이기
 
 -	<app_name>_detail 등의 이름으로 호출할 때 <app_name> 부분이 동일한 문제의 해결방안
 
 ```python
 # 서드 파티 라이브러리의 contact앱이 존재하며, 두번째 contact 앱을 추가해야할 경우
 # 프로젝트 루트에 있는 urls.py
 urlpatterns += [
   url(r'^contact/', include('contactmoger.urls', namespace='contactmoger')),
   url(r'^report-problem/',include('contactapp.urls', namespace='contactapp')),
 ]
 ```
 
 -	템플릿에서의 이용
 
 ```python
 <a href="{% url "contactmoger:creaate" %}">Contact Us</a>
 <a href="{% url "contactapp:report" %}">Report a Problem</a>
 ```
 
 ### 8.4.3 검색, 업그레이드, 리팩터링을 쉽게 하기
 
 -	tastings_detail 같은 코드나 이름은 용도가 애매모호하다.
 -	대신 tastings:detail 이라고 쓰면 검색 결과를 명확하게 해준다.
 
 ### 8.4.4 더 많은 앱과 템플릿 리버스 트릭을 허용하기
 
 -	대부분의 트릭은 프로젝트의 복잡성을 높이는 결과를 가져온다.
 -	참고할 만한 트릭
 	-	디버그 레벨에서 내부적인 검사를 실행하는 개발도구(django-debug-toolbar)
 	-	최종 사용자에게 ‘모듈‘을 추가하게 하여 사용자 계정의 기능을 변경하는 프로젝트
 
 8.5 URLConf에서 뷰를 문자열로 지목하지 말자
 -------------------------------------------
 
 -	**나쁜 예제**
 
 ```python
 # 절대 따라하지 말자!
 # polls/urls.py
 from django.conf.urls import patterns, url
 
 urlpatterns = patterns('',
   url(r'^$', 'polls.views.index', name='index'),
 )
 ```
 
 -	장고가 뷰의 함수, 클래스를 임의로 추가하여 디버그가 어려워진다.
 -	urlpatterns 변수의 초기값에서의 공백문자열(‘’)에 대해 설명해야 한다. <br/>
 
 > **patterns의 공백문자열(‘’)의 의미?** [(출처)](https://stackoverflow.com/questions/40835760/blank-url-pattern-in-django)
 >
 > -	patterns의 첫번째 인자가 접두사로서의 기능을 한다.
 >
 > ```python
 > # 단순화 하기
 > urlpatterns = patterns('polls.views',
 > (r'^$', 'index', name='index'),
 > )
 > ```
 
 -	**좋은 예제**
 
 ```python
 # polls/urls.py
 from django.conf.urls import url
 
 form . import views
 
 urlpatterns = [
   # 뷰를 명시적으로 정의
   url(r'^$', views.index, name='index'),
 ]
 ```
 
 8.6 뷰에서 비즈니스 로직 분리하기
 ---------------------------------
 
 -	모델 메서드, 매니저 메서드, 유틸리티 헬퍼 함수들을 이용하여 비즈니스 로직을 구현하자.
 -	장고의 뷰에서 표준적으로 이용되는 구조 외에 덧붙여진 비즈니스 로직을 발견할 때마다 해당 코드를 뷰 밖으로 이동시키자.
 
 8.7 장고의 뷰와 함수
 --------------------
 
 -	기본적으로 장고의 뷰는 HTTP를 요청하는 객체를 받아서 HTTP를 응답하는 객체로 변경하는 **함수**다.
 -	수학에서 이야기 하는 함수와 개념상 매우 비슷하다.
 
 ```python
 # 함수로서의 장고 함수 기반 뷰
 HttpResponse = view(HttpRequest)
 
 # 수학에서 이용한 함수 식
 y = f(x)
 
 # 클래스 기반 뷰
 HttpResponse = View.as_view()(HttpRequest)
 ```
 
 > **as_view() 메서드** [(출처)](http://ruaa.me/django-view/)
 >
 > -	as_view() 메소드에서 클래스의 인스턴스를 생성한다.
 >
 > -	생성된 인스턴스의 dispatch() 메소드를 호출한다.
 >
 > -	dispatch() 메소드는 요청을 검사해서 HTTP의 메소드(GET, POST)를 알아낸다.
 >
 > -	인스턴스 내에 해당 이름을 갖는 메소드로 요청을 중계한다.
 >
 > -	해당 메소드가 정의되어 있지 않으면, HttpResponseNotAllowd 예외를 발생시킨다.
 >

 ### 8.7.1 뷰의 기본 형태들
 
 ```python
 # simplest_views.py
 from django.http import HttpResponse
 from django.views.generic import View
 
 # 함수 기반 뷰의 기본 형태
 def simplest_view(request):
   # 비즈니스 로직이 여기에 위치한다.
   return HttpResponse('FBV')
 
 # 클래스 기반 뷰의 기본 형태
 class SimplestView(View):
   def get(self, request, *args, **kwargs):
     # 비즈니스 로직이 여기에 위치한다.
    return HttpResponse('CBV')
 ```
 
 **기본 형태가 중요한 이유?**
 
 -	기본 장고 뷰를 이해했다는 것은 장고 뷰의 역활을 명확하게 이해했다는 것이다.
 -	함수 기반 뷰는 HTTP 메서드에 중립적이지만, 클래스 기반 뷰의 경우 HTTP 메서드의 선언이 필요하다는 것을 설명해 준다.
 
 > **메서드 중립적?** [(출처)](http://unifox.tistory.com/33)
 >
 > if 문을 통해 ‘GET’이나 ‘POST’일 때를 나눌 수 있음
 
 ```python
 from django.http import HttPResponse
 
 def my_view(request):
   if request.method == 'GET':
     # 뷰 로직
     return HttPResponse('result')
 ```
 
 8.8 locals()를 뷰 콘텍스트에 이용하지 말자
 ------------------------------------------
 
 -	locals()를 사용하는 것은 안티패턴이다.
 -	명시적이었던 디자인이 암시적 형태로 변하여 뷰를 유지보수하기에 복잡한 형태로 만든다.
 -	**나쁜 예제**
 
 ```python
 # 위의 함수와 아래 함수의 차이점을 찾는데 얼마나 걸리는가?
 def ice_cream_store_display(request, store_id):
   store = get_object_or_404(Store, id=store_id)
   date = timezone.now()
   return render(request, 'melted_ice_cream_report.html', locals())
 
 def ice_cream_store_display(request, store_id):
   store = get_object_or_404(Store, id=store_id)
   now = timezone.now()
   return render(request, 'melted_ice_cream_report.html', locals())
 
 ```
 
 -	**명시적인 컨텐츠를 이용한 뷰**
 
 ```python
 def ice_cream_store_display(request, store_id):
   return render(request, 'melted_ice_cream_report.html', dict{
     'store' : get_object_or_404(Store, id=store_id),
     'now' : timezone.now()
   })
 ```
 
 8.9 요약
 --------
 
 -	이번 장에서는 함수 기반 뷰 또는 클래스 기반 뷰를 언제 이용하는지와 필자가 선호하는 패턴에 대해 다루었다.
 -	클래스 기반 뷰를 이용할 때 객체 상속이 가능하므로 코드를 재사용하기 쉬워지고 디자인을 좀 더 유연하게 할 수 있었다.
 -	다음장에서는 함수 기반 뷰를, 다다음장에서는 클래스 기반 뷰에 대해 더 깊은 내용을 다룰 것이다.
