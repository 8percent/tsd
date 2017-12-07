# 29장 유틸리티들에 대해

## 29.1 유틸리티들을 위한 코어 앱 만들기 
프로젝트 전반에 걸쳐 사용되는 함수들과 객체들을 장고 앱으로 관리하기.

```
core/
    __init__.py
    managers.py  # 커스텀 모델 매니저 포함 
    models.py
    views.py  # 커스텀 뷰 믹스인 포함
```

```python
from core.managers import PublishedManager
from core.views import IceCreamMixin
```

- 코어는 비추상화(non-abstract) 모델을 가지게 된다.
- 코어에 admin [auto-discovery](https://docs.djangoproject.com/ko/1.11/ref/contrib/admin/#django.contrib.admin.autodiscover)를 적용한다.
- 코어에 템플릿 태그와 필터를 위치시킨다.


## 29.2 유틸리티 모듈들을 이용하여 앱을 최적화하기 

### 29.2.1 여러 곳에서 공통으로 쓰이는 코드를 저장해두기 
공통으로 쓰이는 함수 혹은 클래스이지만 특정 이름의 모듈에 이를 넣을 수 없을 때 utils.py 모듈에 위치시킨다.

### 29.2.2 모델을 좀 더 간결하게 만들기 

1. Flavor 모델을 빈번히 사용한다.
2. 필드들을 연이어 작성한다.
3. 메서드도 연이어 작성한다
4. 수천 줄이 넘는 상태가 된다.

이런 상황을 피하기 위해 `flavors/utils.py`로 캡슐화 할 수 있는 부분을 찾아 분산시키자.  
재사용과 테스팅이 한결 수월해질 것이다. 

### 29.2.3 좀 더 손쉬운 테스팅 
독립적인 모듈로 이전한다는 것은 좀 더 손쉬운 테스팅이 가능해지는 효과가 있다.
비지니스 로직은 덜 복잡해지고, 해당 모듈의 테스트는 수월해진다.

> **유틸리티 코드들은 가능한 한 자신의 주어진 역할에만 집중하도록 한다**  
> 
> - 하나의 함수나 클래스에 여러 기능이나 동작을 넣지 말자.  
> - 모델의 기능과 중복되는 유틸리티 함수를 만드는 것은 피하자.



## 29.3 장고 자체에 내장된 스위스 군용 칼 

**django.utils** : 여러가지 유용한 헬퍼함수를 내장하고 있는 패키지

사용하려는 헬퍼함수가 안정화 상태인지 살펴보고 사용하자.  
> [Django Utils](https://docs.djangoproject.com/en/2.0/ref/utils/) 참고  
> 대부분 장고 내부적인 이용을 목적으로 제작되었고, 장고 버전에 영향을 받는다. 용도와 내용이 변할 수 있음.

### 29.3.1 django.contrib.humanize
인간 친화적 기능을 제공하는 템플릿 필터이다.  
예) `intcomma` 필터는 정수를 세 자리마다 쉼표를 찍는다.

django.contrib.humanize의 필터는 함수로 따로 임포트해서 이용할 수도 있다.   
REST API를 구성할 때 이와 같은 텍스트 프로세싱 기능들은 매우 유용하다.

### 29.3.2 django.utils.decorators.method_decorator(decorator)
클래스 메서드는 일반 함수와 완전히 같지 않기 때문에 데코레이터를 메서드에 바로 적용할 수 없다. 따라서 메서드 데코레이터로 변환을 해야하는데, 이러한 역할을 하는 것이 `method_decorator()` 이다.

저자가 뽑은 최고의 함수 데코레이터!

[Django 공식문서: Decorating the class](https://docs.djangoproject.com/ko/1.11/topics/class-based-views/intro/#decorating-the-class)

<!--
> **인스턴스 메소드, 클래스 메소드, 스태틱 메소드**  
> 
> - 인스턴스 메소드
>     - 인스턴스를 통해서 호출됨 
>     - 첫 번째 인자로 인스턴스 자신을 전달됨(관습적으로 'self'를 사용)
> - 클래스 메소드
>     - 클래스를 통해서 호출됨  
>     - `@classmethod` 데코레이터로 정의
>     - 첫 번째 인자로 클래스 자신이 전달됨(관습적으로 'cls'를 사용)
> - 스태틱 메소드
>     - 클래스나 인스턴스를 통해서 호출
>     - 클래스 안에서 정의되어 있을뿐 일반 함수와 전혀 다를게 없음. 
> 
> 출처: [클래스 메소드와 스태틱 메소드](http://schoolofweb.net/blog/posts/%ED%8C%8C%EC%9D%B4%EC%8D%AC-oop-part-4-%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A9%94%EC%86%8C%EB%93%9C%EC%99%80-%EC%8A%A4%ED%83%9C%ED%8B%B1-%EB%A9%94%EC%86%8C%EB%93%9C-class-method-and-static-method/)
-->



### 29.3.3 django.utils.decorators.decorator\_from_middleware(middleware)
인자로 주어진 미들웨어 클래스는 view decorator를 반환한다. 이를 통해 view 단위로 미들웨어 기능을 사용할 수 있다.

`decorator_from_middleware_with_args` 사용의 예)

```python
cache_page = decorator_from_middleware_with_args(CacheMiddleware)

@cache_page(3600)
def my_view(request):
    pass
```

### 29.3.4 django.utils.encoding.force\_text(value) 
장고의 무엇이든 python3의 일반 str 형태 또는 python2의 unicode 형태로 변환시킨다.

### 29.3.5 django.utils.functional.cached\_property 
메서드의 결과를 프로퍼티로 메모리에 캐시를 해주는 기능을 한다.  
[장고 공식문서: cached_property](https://docs.djangoproject.com/en/2.0/ref/utils/#django.utils.functional.cached_property)  

- 많은 리소스를 요구하는 연산 결과를 캐시할 때 유용하다. (성능 면에서 혜택을 볼 수 있음)
- 객체가 존재하는 동안 메서드의 값이 항상 일정한지 확인하는 용도로도 쓰인다.  
- 외부 서드파티 API 또는 데이터베이스 트랜잭션 관련 작업을 할 때에도 매우 유용하다.


### 29.3.6 django.utils.html.format\_html(format_str, args, **kwargs) 
HTML을 처리하기 위해 제작되었다.  
python의 `str.format()` 메서드와 유사하다.  
[장고 공식문서: format_html](https://docs.djangoproject.com/en/2.0/ref/utils/#django.utils.html.format_html)

### 29.3.7 django.utils.html.remove\_tags(value, tags) 
'26.28 django.utils.html.remove_tag 이용하지 않기' 에서 보았듯, 보안상의 이유로 더이상 지원되지 않는다.

### 29.3.8 django.utils.html.strip\_tags(value)
사용자로부터 받은 콘텐츠에서 HTML 코드를 분리할 때 사용한다.  
태그 사이의 텍스트를 유지한 채로 태그만 제거할 수 있다.

> **strip_tags에 대한 보안**  
> 결과 문자열에 안전이 보장되지는 않는다. 템플릿에서 자동 이스케이핑 기능이 비활성화 되있다면 더욱!

### 29.3.9 django.utils.six 
벤자민 피터슨(Benjamin Peterson)이 제작한 python2와 python3 호환 라이브러리이다.   
django utils 라이브러리에 포함되어 있지만 독자적 패키지 형태로 찾아볼 수도 있다.

- [pypi에서의 six](https://pypi.python.org/pypi/six)
- [six 문서](http://pythonhosted.org/six/)
- [github의 six 저장소](https://github.com/benjaminp/six)
- [Django의 six](https://github.com/django/django/blob/master/django/utils/six.py)

### 29.3.10 django.utils.text.slugify(value) 
자체적인 `slugify()` 함수를 만드는 것은 자제하라.  
장고 프로젝트의 비일관성으로 후일 데이터에 큰 문제를 초래할 수 있다.  
저자는 자체함수 대신 utils의 [slugify()](https://docs.djangoproject.com/en/1.11/ref/utils/#django.utils.text.slugify)를 사용했다. 

> django.templates.defaultfilters.slugify()를 호출해서 사용하는 방법도 있음


### 29.3.11 django.utils.timezone 
장고의 timezone을 이용할 때, 데이터베이스에는 UTC 포맷 형태로 저장하고, 필요에 따라 지역 시간대에 맞게 변형하여 이용하자.

> [Django timezone 문제 파헤치기(8퍼센트 프로덕트 블로그)](https://8percent.github.io/2017-05-31/django-timezone-problem/)

### 29.3.12 django.utils.translation 
장고에서는 국제화(i18n) 기능이 제공된다. (다국어 지원)

[Django 공식 문서: translation](https://docs.djangoproject.com/en/1.11/ref/utils/#module-django.utils.translation)

> **I18N**  
> 국제화.
> internationalization의 약자. (i와 n사이에 18글자가 있기 때문...)  
> 
> **L10N**  
> 현지화.
> Localization의 약자. (l과 n사이에 10글자가 있기 때문...)


## 29.4 예외
장고는 여러가지 예외를 지원한다.  
[Django 공식문서: Exceptions](https://docs.djangoproject.com/en/2.0/ref/exceptions/)

### 29.4.1 django.core.exceptions.ImproperlyConfigured 
설정상의 문제를 알려준다.  
장고 세팅 모듈을 임포트할 때 세팅 모듈이 문제가 없는지 검사를 하는 장고 코드로 작동한다.

### 29.4.2 django.core.exceptions.ObjectDoesNotExist 
모든 DoesNotExist 예외가 상속하는 기본 Exception 모듈이다. 

기본 모델(generic model) 인스턴스를 페치(fetch)해서 이를 이용하는 유틸함수의 경우 매우 유용하게 사용된다. 

예)

```python
# core/utils.py
from django.core.exceptions import ObjectDoesNotExist

class BorkedObject(object):
    loaded = False
	
def generic_load_tool(model, pk):
    try:
        instance = model.objects.get(pk=pk)
    except ObjectDoesNotExist:
        return BorkedObject()
        
    instance.loaded = True
    return instance    
```

이 예외로 404 대신 HTTP 403 예외를 발생시키는 `django.shortcuts.get_object_or_403` 함수를 만들 수도 있다. 

```python
# core/utils.py
from django.core.exceptions import MultipleObjectsReturned
from django.core.exceptions import ObjectDoesNotExist
from django.core.exceptions import PermissionDenied

def get_object_or_403(model, **kwargs):
    try:
        return model.objects.get(**kwargs)
    except ObjectDoesNotExist:
        raise PermissionDenied
    except MultipleObjectsReturned:
        raise PermissionDenied
```

### 29.4.3 django.core.exceptions.PermissionDenied 
인증된 사용자가 허가되지 않은 곳에 접속을 시도할 때 이용된다.  
해당 예외를 발생시켜 view가 `django.http.HttpResponseForbidden`을 유발게 할 수 있다.

- 보안이 중시되는 프로젝트의 컴포넌트들과 민감한 데이터를 조작하는 함수에서 유용하게 사용된다. 
- 예기치 못한 일이 발생했을 때 단순히 500 에러를 반환하는 대신 `Permission Denied` 화면을 보여줌으로써 사용자에게 경고를 보낼 수도 있다. 

```python
# store/calc.py

def finance_data_adjudication(store, sales, issues):
    if store.something_not_right:
        msg = "Something is not right. Please contact the support team."
        raise PermissionDenied(msg)
```

위와 같이 무언가 제대로 작동하지 않고 있을 때, PermissionDenied 예외로 처리하여 403 에러페이지를 표시할 수 있다. 

프로젝트 루트 디렉토리 URLConf에서 다음과 같이 추가하면 403 에러 페이지를 어떤 뷰에서든 이용할 수 있다. 

```python
# urls.py

# 기본 뷰는 django.views.defaults.permission_denied
# 커스텀 뷰를 사용할 수 있다. 
handler403 = 'core.views.permission_denied_view'
```

## 29.5 직렬화 도구와 역직렬화 도구 
데이터 파일을 생성하거나 단순한 REST API를 제작할 때를 위해   
JSON, 파이썬, YAML, XML 데이터를 위한 유용한 직렬화, 역직렬화 도구를 제공한다. 

모델 인스턴스를 직렬화된 형태의 데이터로 변환하거나, 이를 다시 모델 인스턴스로 변환하는 기능을 포함한다.  

직렬화 예)

```python
# serializer_example.py
from django.core.serializers import get_serializer

from favorites.models import Favorite

JSONSerializer = get_serializer("json")
serializer = JSONSerializer()

favs = Favorite.objects.filter()[:5]

serialized_data = serializer.serialize(favs)

with open("data.json", "w") as f:
    f.write(serialized_data)
```

역직렬화 예)

```python
# deserializer_example.py
from django.core.serializers import get_serializer

from favorites.models import Favorite

favs = Favorite.objects.filter()[:5]

JSONSerializer = get_serializer("json")
serializer = JSONSerializer()

with open("data.txt") as f:
    serialized_data = f.read()

python_data = serializer.deserialize(serialized_data)

for element in python_data:
    # 'django.core.serializers.base.DeserializeObject' 출력
    print(type(element))
    
    print(element.object.pk, element.object.created)
```

장고의 직렬화/역직렬화 도구를 이용할 때 언제나 문제가 발생할 수 있다는 것을 유의해야한다.

**가이드라인**  

- 간단한 데이터에 대해서만 직렬화를 한다.
- 데이터베이스 스키마의 변화는 언제라도 직렬화된 데이터와 문제를 일으킬 수 있다.
- 그냥 단순히 직렬화된 데이터를 임포트하지 말고, 언제나 장고 폼 라이브러리를 이용하여 데이터를 저장하기 전에 확인 작업을 거친다.


### 29.5.1 django.core.serializers.json.DjangoJSONEncoder 
파이썬에 내장된 JSON 모듈에는 날짜, 시간, 소수점 데이터를 인코딩하는 기능이 없다.
장고는 JSONEncoder 클래스를 제공한다.

```python
# json_encoding_example.py
import json

form django.core.serializers.json import DjangoJSONEncoder
from django.utils import timezone

data = {"date": timezone.now()}

# DjangoJSONEncoder 클래스를 추가하지 않으면 
# json 라이브러리가 TypeError를 일으킬 것이다.
json_data = json.dumps(data, cls=DjangoJSONEncoder)

print(json_data)
```

### 29.5.2 django.core.serializers.pyyaml 
장고의 YAML 직렬화 도구

- pyyaml 서드파티 라이브러리의 도움을 받음 
- pyyaml이 지원하지 않는 Python-to-YAML으로 부터 시간 변환을 처리할 수 있다.   
- 역직렬화는 `yaml.safe_load()` 함수를 이용, 코드 인젝션 문제는 걱정하지 않아도 된다.

> '26.9.3 코드를 실행할 수 있는 서드파티 라이브러리들' 참고

### 29.5.3 django.core.serializers.xml_serializer
- 기본적으로 파이썬 내장 XML 핸들러를 이용한다.  
- XML 폭탄 공격을 방지하기 위해 크리스티안 하이메스(Christian Heimes)의 `defusedxml` 라이브러리를 포함하고 있다.

> '26.22 defusedxml을 이용하여 XML 폭탄막기' 참고

