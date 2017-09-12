# 16장. REST API 구현하기

REST(representational state transfer) API는 다양한 환경과 용도에 맞는 데이터를 제공하는 디자인을 정의하고 있다. <br/>
<a href="http://meetup.toast.com/posts/92">REST API</a>
<a href="http://www.django-rest-framework.org/tutorial/quickstart/">DRF 문서</a>

## 16.1 기본 REST API 디자인의 핵심
REST API는 HTTP를 기반으로 삼고 있으므로 각 액션에 알맞는 HTTP 메서드를 사용하면 된다.
- 읽기전용 API만 구현한다면 GET 메서드만 구현
- 읽기/쓰기 API를 구현한다면 최소 POST 구현, PUT과 DELETE 또한 고려
- 단순화하기 위해 때로 GET과 POST 만으로도 구현되도록 설계
- API가 PUT 요청을 지원한다면 PATCH 또한 구현하는 것이 좋다.

## 16.2 간단한 JSON API 구현하기

flaver 앱 예제를 기반으로 HTTP 요청을 통한 create, read, update, delete 를 처리하는 방법을 구현
(django-rest-framework 이용)

~~~python
# flavors/models.py
from django.core.urlresolvers import reverse
from django.db import models

class Flavor(models.Model):
    title = models.CharField(max_length=255)
    slug = models.SlugField(unique=True)
    scoops_remaining = models.IntegerField(default=0)

    def get_absolute_url(self):
        return reverse("flavors:detail", kwargs={"slug": self.slug})
~~~

serializer 클래스

~~~python
from rest_framework import serializers

from .models import Flavor

class FlavorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Flavor
        fields = ('title', 'slug', 'scoops_remaining')
~~~

view

~~~python
# flavors/views
from rest_framework.generics import ListCreateAPIView
from rest_framework.generics import RetrieveUpdateDestroyAPIView
from .models import Flavor
from .serializers import FlavorSerializer

class FlavorCreateReadView(ListCreateAPIView):
    queryset = Flavor.objects.all()
    serializer_class = FlavorSerializer
    lookup_field = 'slug'

class FlavorReadUpdateDeleteView(RetrieveUpdateDestroyAPIView):
    queryset = Flavor.objects.all()
    serializer_class = FlavorSerializer
    lookup_field = 'slug'
~~~

urls.py
~~~python
# flavors/urls.py
from django.conf.urls import url

from flavors import views

urlpatterns = [
    url(
        regex=r"^api/$",
        view=views.FlavorCreateReadView.as_view(),
        name="flavor_rest_api"
    ),
    url(
        regex=r"^api/(?P<slug>[-\w]+)/$",
        view=views.FlavorReadUpdateDeleteView.as_view(),
        name="flavor_rest_api"
    )
]
~~~

전통적인 REST 스타일의 API 정의

~~~
flavors/api/
flavors/api/:slug/
~~~

# 16.3 REST API 아키텍쳐

## 16.3.1 프로젝트 코드들은 간결하게 정리되어 있어야 한다.
api 만 전담해서 처리하는 앱을 따로 구성(모든 시리얼라이저, 랜더러, 뷰가 위치하고 이름에 해당 API버전을 명시)하는 경우 특정 api가 위치한 곳을 찾기는 용이하지만, 해당 앱이 너무 커질수 있고 개별 앱으로 부터 해당 API 앱이 단절 될 수도 있음.


## 16.3.2 앱의 코드는 앱 안에 두자
일반적인 뷰가 따르는 가이드라인을 따라 __init__.py를 포함하는 viewset 패키지에 나눠 넣으면, 모듈들이 너무 나뉘고 수가 많아질 경우 api 컴포넌트들을 관리하기 어려워 질수 있다.

## 16.3.3 비지니스 로직을 API뷰에서 분리하기
API 뷰 또한 뷰의 한종류이기 때문에 비지니스 로직과 최대한 분리 필요

## 16.3.4 API URL을 모아 두기
앱 단위가 아닌 프로젝트 전반에 걸친 REST API 뷰의 경우 core/api.py 의 형태로 구현된 모듈을 URLConf를 이용하여 urls.py 안에 포함

~~~python
# core/api.py
"""프로젝트 루트 폴더에 위치한 urls.py의 URLConf에서 호출됨, 따라서:
        url(r"^api/", include("core.api", namespace="api")),
"""
from django.conf.urls import url

from flavors import views as flavor_views
from users import views as user_views

urlpatterns = [
    # {% url "api:flavors" %}
    url(
        regex=r"^flavors/$",
        view=flavor_views.FlavorCreateReadView.as_view(),
        name="flavors"
    ),
    # {% url "api:flavors" flavor.slug %}
    url(
        regex=r"^flavors/(?P<slug>[-\w]+)/$",
        view=flavor_views.FlavorReadUpdateDeleteView.as_view(),
        name="flavors"
    ),
    # {% url "api:users" %}
    url(
        regex=r"^users/$",
        view=user_views.UserCreateReadView.as_view(),
        name="users"
    ),
    # {% url "api:users" user.slug %}
    url(
        regex=r"^users/(?P<slug>[-\w]+)/$",
        view=user_views.UserReadUpdateDeleteView.as_view(),
        name="users"
    ),
]
~~~

## 16.3.5 API 테스트하기 
22장에서...

## 16.3.6 API 버저닝하기
api의 URL에 버전 정보를 나타내는 것은 매우 유용.<br /> 기존 사용자를 고려하여 이전 버전의 api를 유지하는 것도 중요하다.<br /> 또한 사전에 이전 api의 사용이 중단됨을 충분한 시간을 가지고 공지해야한다.

# 16.4 서비스 지향 아키텍처(SOA, service-oriented architecture)
웹 애플리케이션은 독립적이고 분리된 컴포넌트로 구성된다. <br />
참고링크 : <a href="http://channy.creation.net/articles/microservices-by-james_lewes-martin_fowler#.WbgZINNJbUo">마이크로서비스</a> <br/>

각 장고 앱이 독립된 장고 프로젝트로 나뉠 수 있음 <br/>

# 16.5 외부 API 중단하기
## 1단계: 사용자들에게 서비스 중지 예고하기
## 2단계: 410 에러뷰로 API 교체(새 api 엔드포인트링크, 새 api 문서의 링크, 서비스 중지에 대한 세부사항을 알려주는 문서링크 포함)

예시
~~~python
# core/apiv1_shutdown.py
from django.http import HttpResponseGone

apiv1_gone_msg = """APIv1 was removed on April 2, 2015. Please switch to APIv3:
<ul>
    <li>
        <a href="https://www.example.com/api/v3/">APIv3 Endpoint</a>
    </li>
    <li>
        <a href="https://example.com/apiv3_docs/">APIv3 Documentation</a>
    </li>
    <li>
        <a href="http://example.com/apiv1_shutdown/">APIv1 shut down notice</a>
    </li>
</ul>
"""

def apiv1_gone(request):
    return HttpResponseGone(apiv1_gone_msg)
~~~

# 16.6 API에 접속 제한하기
## 16.6.1 제한 없는 API 접속은 위험하다.
애플리케이션에 장애를 일으킬 수 있다.
## 16.6.2 REST 프레임워크는 반드시 접속 제한을 해야한다.

## 16.6.3 비지니스 계획으로서의 접속 제한
접근 수준에 따른 가격 정책

# 16.7. 자신의 REST API 알리기
## 16.7.1 문서 
쉬운 설명과 예제가 포함, 23장에서 좀 더 자세히
## 16.7.2 클라이언트 SDK 제공
많은 언어를 지원할 수록 좋음.



