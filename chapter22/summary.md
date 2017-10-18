## 22.1 테스트, 정말 거추장스럽고 낭비일까?
충분한 테스트가 이루어지지 않는다면 직장, 돈 그리고 심하게는 생명까지도 한순간에 위험에 처할 수 있다. 오늘날 장고가 다양한 어플리케이션이 쓰이면서 자동화된 테스트에 대한 요구가 중요한 문제로 대두되었다.

## 22.2 어떻게 테스트를 구축할 것인가
우선 해야할 일은 django-admin startapp 명령으로 기본 생성되었던 불필요한 test.py 모듈을 지우는 것이다.

test\_forms.py, test\_models.py, test\_views.py 모듈을 직접 새로 생성한다.
다음과 같은 구조가 생성될 것이다.

```
application/
    __init__.py
    admin.py
    forms.py
    models.py
    test_forms.py
    test_models.py
    test_views.py
    views.py
```

 물론 forms.py, models.py, views.py 외에 다른 종류의 파일들이 있다면 그에 해당하는 테스트 파일을 같은 방식으로 생성하자. 이는 '파이썬의 도'의 '수평적인 것이 중첩된 것보다 낫다(Flat is better than nested)' 철학에 기반을 두고 구성한 것이다. 이렇게 구성하면 장고 앱을 훨씬 쉽게 살펴볼 수 있다.

> 테스트 모듈은 `test_` 접두어를 붙여 생성하자 <br>
그렇게 하지 않으면 장고의 테스트 러너가 해당 테스트 파일을 인지하지 못 한다.

## 22.3 단위 테스트 작성하기
### 22.3.1 각 테스트 메서드는 테스트를 한 가지씩 수행해야 한다
- 테스트 메서드는 그 테스트 범위가 좁아야 한다. 
- 하나의 단위 테스트는 절대로 여러 개의 뷰나 모델 폼 또는 한 클래스 안의 여러 메서드에 대한 테스트를 수행해서는 안 된다. 
- 하나의 테스트에서는 뷰, 모델, 폼 메서드가 작동하는 데 있어서 단 하나의 기능에 대한 정의, 즉 뷰면 뷰, 모델이면 모델 식으로 하나의 기능에 대해서만 테스트가 이루어져야 한다.

하지만 뷰 하나만으로도 모델, 폼, 메서드 그리고 함수가 줄줄이 연관지어 호출되는데, 어떻게 뷰에 대한 테스트만을 딱 잘라서 하나의 테스트로 할 수 있단 말인가?

이에 대한 방법은 특별한 테스트에 대한 환경을 완전히 최소한으로 구성하는 것이다.

```python
# flavors/test_api.py
import json

from django.core.urlresolvers import reverse
from django.test import TestCase

from flavors.models. import Flavor

class FlavorAPITests(TestCase):
    def setUp(self):
        Flavor.objects.get_or_create(title="A Title", slug="a-slug")
        
    def test_list(self):
        url = reverse("flavor_object_api")
        response = self.client.get(url)
        self.assertEquals(response.status_code, 200)
        data = json.loads(response.content)
        self.assertEquals(len(data), 1)
```

아래 코드는 좀 더 확장된 예제이다.

```python
# flavors/test_api.py
import json

from django.core.urlresolvers import reverse
from django.test import TestCase

from flavors.models. import Flavor

class DjangoRestFrameworkTests(TestCase):
    def setUp(self):
        Flavor.objects.get_or_create(title="title1", slug="slug1")
        Flavor.objects.get_or_create(title="title2", slug="slug2")
        self.create_read_url = reverse("flavor_rest_api")
        self.read_update_delete_url = \
            reverse("flavor_rest_api", kwargs={"slug": "slug1"})
            
    def test_list(self):
        response = self.client.get(self.create_read_url)
        # 타이틀 둘 다 콘텐츠 안에 존재하는가?
        self.assertContains(response, "title1")
        self.assertContains(response, "title2")
        
    def test_detail(self):
        response = self.client.get(self.read_update_delete_url)
        data = json.loads(response.content)
        content = {"id": 1, "title": "title1", "slug": "slug1", "scoops_remaining": 0}
        self.assertEquals(data, content)
        
    def test_create(self):
        post = {"title": "title3", "slug": "slug3"}
        response = self.client.post(self.crate_read_url, post)
        data = json.loads(response.content)
        self.assertEquals(response.status_code, 201)
        content = {"id": 3, "title": "title3", "slug": "slug3", "scoops_ramaining": 0}
        self.assertEquals(data, content)
        self.assetEquals(Flavor.objects.count(), 3)
    
    def test_delete(self):
        response = self.client.delete(self.read_update_delete_url)
        self.assertEquals(response.status_code, 204)
        self.assertEquals(Falvor.objects.count(), 1)    
```

### 22.3.2 뷰에 대해서는 가능하면 요청 팩터리를 이용하자
- [django.test.client.RequestFactory](https://docs.djangoproject.com/en/1.8/topics/testing/advanced/#module-django.test.client)는 모든 뷰에 대해 해당 뷰의 첫 번째 인자로 이용할 수 있는 `request` 인스턴스를 제공한다. 이는 일반 장고 테스트 클라이언트보다 독립된 환경을 제공한다. 
- RequestFactory는  get(), post(), put(), delete(), head(), options(), trace()와 같은 HTTP 메서드만 접근 가능하다.
- 하지만 생성된 요청이 세션과 인증을 포함한 미들웨어를 지원하지 않기 때문에 테스트를 작성할 때 추가적인 작업이 필요하다.

테스트하고자 하는 뷰가 세션을 필요로 할 때는 다음과 같이 작성할 수 있다.

```python
from django.contrib.auth.models import AnonymousUser
from django.contrib.sessions.middleware import SessionMiddleware
from django.test import TestCase, RequestFactory

from .views import cheese_flavors

def add_middleware_to_request(request, middleware_class):
    middleware = middleware_class()
    middleware.process_request(request)
    return request
    
def add_middleware_to_response(reqeust, middleware_class):
    middleware = middleware_class()
    middleware.process_request(request)
    return request
    
class SavoryIceCreamTest(TestCase):
    def setUp(self):
        # 모든 테스트에서 이 요청 팩토리로 접근할 수 있어야 한다.
        self.factory = RequestFactory()
        
    def test_cheese_flavors(self):
        request = self.factory.get('/cheesy/broccoli/')
        request.user = AnonymousUser()
        
        # 요청 객체에 세션을 가지고 표식을 달도록 한다.
        request = add_middleware_to_request(request, SessionMiddleware)
        
        # 요청에 대한 처리와 테스트를 진행한다.
        response = cheese_flavors(request)
        self.assertContains(response, "bleah!")
```

### 22.3.3 테스트가 필요한 테스트 코드는 작성하지 말자
테스트는 가능한 한 단순하게 작성해야한다. 테스트 케이스 안의 코드가 복잡하거나 추상적이라면 문제가 있는 것이다.

### 22.3.4 같은 일을 반복하지 말라는 법칙은 테스트 케이스를 쓰는 데는 적용되지 않는다
- setUp() 메서드는 테스트 클래스의 모든 테스트 메서드에 대해 재사용이 가능한 데이터를 만드는 데 큰 도움이 된다. 
- 그러나 때론 비슷하지만 각 테스트 메서드에 대해 각기 다른 데이터가 필요하기도 하고 종종 이러한 이유로 기능이 화려한 테스트 유틸리티를 만드는 실수에 빠지기도 한다. 
- 이러한 경우에 같거나 비슷한 코드를 여러 번 반복해서 쓰고 또 쓰는 것이 해법이다.

### 22.3.5 픽스처를 너무 신뢰하지 말자
- 프로젝트의 데이터가 바뀌어 감에 따라 픽스처 자체를 유지하기 어렵다. JSON 파일이 손상되었거나 데이터베이스를 제대로 반영하고 있지 않는 상태라면 JSON 파일의 로드 프로세스에서 매우 큰 어려움을 겪게 된다.
- 이럴 경우 픽스처 보다는 ORM에 의존하는 코드를 제작하는 편이 훨씬 쉬울 것이다. 서드 파티 패키지를 이용하는 것을 선호하는 사람도 있다.

> **테스트 데이터를 생성해 주는 도구** <br>
- factory boy: 모델 테스트 데이터 생성 <br>
- model mommy: 모델 테스트 데이터 생성 <br>
- mock: 장고뿐만 아니라 다른 환경에서도 이용 가능한 시스템의 일부를 대체할 수 있는 목(mock) 객체 생성

### 22.3.6 테스트해야 할 대상들
 물론 `전부 다` 테스트해야 하는 대상이다. 다음을 포함해서 테스트 가능한 것은 모두 다 테스트해야 한다.

- **뷰:** 데이터 뷰, 데이터 변경 그리고 커스텀 클래스에 기반을 둔 뷰 메서드
- **모델:** 모델의 생성, 수정, 삭제 및 모델의 메서드와 모델 관리 메서드
- **폼:** 폼 메서드, clean() 메서드, 커스텀 필드
- **유효성 검사기:** 본인이 제작한 커스텀 유효성 검사기에 대해 다양한 테스트 케이스를 심도 깊게 작성하라. 
- **시그널:** 시그널은 원격에서 작동하기에 테스트를 하지 않을 경우 문제를 야기하기 쉽다.
- **필터:** 필터들은 기본적으로 한 개 또는 두 개의 인자를 넘겨받는 함수이기 때문에 테스트를 제작하기에 그리 어렵지 않다.
- **템플릿 태그:** 템플릿 태그는 그 기능이 막강하고 또한 템플릿 콘텍스트를 허용하기 때문에 테스트 케이스를 작성하는 것이 까다롭다. 이는 즉 정말 테스트해야할 대상이라는 것이다.
- **기타:** 콘텍스트 프로세서, 미들웨어, 이메일, 그리고 이 목록에 포함되지 않은 모든 것
- **실패:** 만약 위의 경우 중 어느 하나라도 실패하면 어떻게 될까?

프로젝트 내에서 테스트가 필요 없는 부분은 장고 코어 부분과 서드 파티 패키지에서 이미 테스트되어 있는 부분 정도일 것이다.

### 22.3.7 테스트의 목적은 테스트의 실패를 찾는 데 있다
테스트 시나리오보다 이러한 예외적인 경우에 대한 테스트가 더욱 중요하다. 성공 시나리오에서의 실패는 사용자에게 불편을 야기하겠지만 바로 보고되는 사항들이다. 반면 실패 시나리오에서의 실패는 인지하지도 못하는 보안상의 문제점을 만들어내고, 이러한 문제들이 발견됐을 때는 이미 너무 늦은 경우가 허다하다.
 
### 22.3.8 목(Mock)을 이용하여 실제 데이터에 문제를 일으키지 않고 단위 테스트 하기
Mock 소셜 로그인이나 결제 시스템과 같은 외부 리소스 디펜던시가 있는 경우, 우리 서버가 아닌 다른 서버에서 작동하는 것을 테스트에 적용하기는 힘들다. 해당 테스트를 가짜로 적용해주는 것을 말한다.
 단위 테스트는 단위 테스트 자체가 호출하는 함수나 메서드 이외의 것은 테스트하지 않도록 구성되어 있다. 이는 외부 API를 이용하는 기능들에 대해 단위 테스트를 작성하는 데 큰 난제로 나타나게 된다.

- 선택 1: 단위 테스트 자체를 통합 테스트(Intergration Test)로 변경한다.
- 선택 2: 목(Mock) 라이브러리를 이용하여 외부 API에 대한 가짜 응답을 만든다.

목(Mock) 라이브러리는 우리가 테스트를 위한 특정 값들의 반환을 필요로 할 때 매우 빠르게 이용할 수 있는 몽키 패치(monkey_patch) 라이브러리를 제공한다. 

> **몽키 패치(Monkey Patch)란?** <br>
런타임 중인 프로그램 메모리의 소스 내용을 직접 바꾸는 것이다. 원래 게릴라 패치였는데 발음의 유사성으로 사람들이 고릴라 패치라고 쓰기 시작했다. 하지만 고릴라라고 하면 좀 위험하게 들리기 때문에 고릴라보다 덩치가 작은 원숭이 패치로 부르게 됐다고... 일반적으로 개발자들 사이에서 몽키패치는 안티 패턴으로 인식된다.
 
이는 우리가 외부 API에 대한 유효성을 테스트하는 것이 아닌 우리 코드의 로직에 대한 검사를 하게 되는 것이다.

```python
import mock
import unittest

import icecreamapi

from flavors.exceptions import CantListFlavors
from flavors.utils import list_flavors_sorted

class TestCreamSorting(unittest.TestCase):
    # icecreamapi.get_flavors() 몽키 패치 세팅
    @mock.patch.object(icecreamapi, "get_flavors")
    def test_flavor_sort(self, get_flavors):
        # icecreamapi.get_flavors()가 정렬되어 있지 않은 리스트를 생성하도록 설정
        get_flavors.return_value = ['chocolate', 'vanilla', 'strawberry', ]
        
        # list_flavors_sorted()가 icecreamapi.get_flavors() 함수 호출
        # 몽키 패치를 했으므로 항상 반환되는 값은 ['chocolate', 'vanilla', 'strawberry', ]가 되며,
        # 이는 list_flavors_sorted()에 의해 자동으로 정렬된다.
        flavors = list_flavors_sorted()
        
        self.assertEqual(
            falvors,
            ['chocolate', 'strawberry', 'vanilla', ]
        )
```

### 22.3.9 좀 더 고급스러운 단언 메서드 사용하기
두 개의 리스트(또는 튜플)를 서로 비교하는 것은 흔히 보는 테스트 케이스다. 하지만 `두 리스트가 서로 다른 정렬 형식`을 가지고 있다면 리스트를 매치하기 위해 `재정렬`해야 한다. 

이럴 경우에 **set.assertEqual(control\_list, candidate\_list)와 같이 사용하는 것이 옳은 것일까?** `unittest의 ListItemsEqual()`이라는 Assert Method를 알고 있다면 "아니다"라고 이야기할 것이다.

### 22.3.10 각 테스트 목적을 문서화하라
문서화되지 않은 코드 때문에 프로젝트 관리가 어려워지는 것처럼 문서화되지 않은 테스트 코드는 테스트를 불가능하게 할 수 있다. 디버그가 불가능한 문제를 해결하는 현명한 방법은 그 해당 문제에 관한 테스트를 문서화하는 것임을 명심하자.

## 22.4 통합 테스트(Integration Tests)란?
개별적인 소프트웨어 모듈이 하나의 그룹으로 조합되어 테스트되는 것을 의미하며, 단위 테스트가 끝난 후에 행하는 것이 가장 이상적이다.

- 어플리케이션이 브라우저에 잘 작동하는지 확인하는 셀레늄(Selenium) 테스트
- 서드 파티 API에 대한 가상 목(mock) 응답을 대신하는 실제 테스팅
- 외부로 나가는 요청에 대한 유효성 검사를 위해 requestb.in이나 http://httpbin.org 등과 연동하는 경우
- API가 기대하는 대로 잘 작동하는지 확인하기 위해 runscope.com을 이용하는 경우

### 통합 테스트의 문제점
- 시간이 많이 소요될 수 있으며 단위 테스트와 비교하면 테스트 속도가 느리가.
- 통합 테스트로부터 반환된 에러의 경우, 에러 이면에 숨어 있는 에러의 원인을 찾기가 단위 테스트보다 어렵다.
- 단위 테스트에 비해 많은 주의를 요구하여 작은 변경으로도 전반에 문제가 나타날 수 있다.