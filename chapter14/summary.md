# 템플릿 태그와 필터

#### 장고가 제공하는 기본 템플릿 태그와 필터의 특성
- 모든 기본 템플릿과 태그의 이름은 명확하고 직관적이어야 한다.
- 모든 기본 템플릿과 테그는 각각 한 가지 목적만을 수행한다.
- 기본 템플릿과 태그는 영속 데이터에 변형을 가하지 않는다.

## 14.1 필터는 함수다 

- 필터는 하나 또는 두개의 인자를 받는 함수이다.   
- 장고 템플릿의 작동을 통제하는 기능을 제공하지는 않는다.  
- 필터는 장고 템플릿 안에서 파이썬을 이용할 수 있게 해주는 데코레이터를 가진 함수 일 뿐이다.  

[참고: 템플릿 필터 커스텀하기](https://anohk.github.io/2017-07-08/custom-template-filter/)

### 14.1.1 필터들은 테스트하기 쉽다 
'22장 테스트, 정말 거추장스럽고 낭비일까?' 에서 다루듯 테스팅 함수를 제작하는 것 만으로 필터를 테스트할 수 있다. 

### 14.1.2 필터와 코드의 재사용 
장고의 [defaultfilter.py 소스 코드](https://github.com/django/django/blob/master/django/template/defaultfilters.py)를 보면 알 수 있듯, 대부분의 필터 로직은 다른 라이브러리로부터 상속되어 왔다.     
`django.template.defaultfilters.slugify`를 임포트할 필요 없이, `django.utils.text.slugify`를 바로 임포트하여 사용하면 된다. 

~~~python
# template/defaultfilters.py

from django.utils.text import slugify as _slugify

@register.filter(is_safe=True)
@stringfilter
def slugify(value):
    return _slugify(value)
~~~

~~~python
# /utils/text.py

@keep_lazy(str, SafeText)
def slugify(value, allow_unicode=False):
    value = str(value)
    if allow_unicode:
        value = unicodedata.normalize('NFKC', value)
    else:
        value = unicodedata.normalize('NFKD', value).encode('ascii', 'ignore').decode('ascii')
    value = re.sub(r'[^\w\s-]', '', value).strip().lower()
    return mark_safe(re.sub(r'[-\s]+', '-', value))
~~~

> 필터 자체를 임포트해도 상관 없으나 추상화된 레벨이 하나 더해지기 때문에 훗날 디버깅시 복잡해질 수 있다.

필터들은 파이썬 함수이기 때문에 정말 단순한 로직이 아니라면 전부 utils.py 모듈같은 유틸리티 함수로 옮길 수 있다. 

### 14.1.3 언제 필터를 사용해야하는가
필터는 데이터의 외형을 수정하는 데 매우 유용하다. REST API와 다른 출력 포멧에서도 손쉽게 재사용할 수 있다. 
> 두 개의 인자만 받을 수 있는 기능적 제약으로 인해 필터를 복잡하게 이용하는 것이 사실상 불가능하다. 

## 14.2 커스텀 템플릿 태그 

> 너무 많은 템플릿 태그는 자제해 주세요. 디버깅하기가 너무 고통스러워요.  
> - 대니얼의 코드를 디버깅하던 중에 오드리가...

템플릿 태그 필터에 너무 많은 로직을 넣을 경우, 겪을 수 있는 문제들은 다음과 같다. 
 
### 14.2.1 템플릿 태그들은 디버깅하기가 쉽지 않다 
복잡한 템플릿 태그들은 디버깅하기 까다롭다. 
이 경우 로그와 테스트를 통해 도움을 받을 수 있다.

### 14.2.2 템플릿 태그들은 재사용하기가 쉽지 않다 
REST API, RSS 피드 또는 PDF/CSV 생성을 위한 출력 포맷 등을 동일 템플릿 태그를 이용해 처리하기 쉽지않다.  
여러 종류의 포맷이 필요하다면 템플릿 태그 안의 로직을 utils.py로 옮기는 것을 고려해보자. 

### 14.2.3 템플릿 태그의 성능 문제 
템플릿 태그들은 심각한 성능 문제를 야기할 수도 있다.  
커스텀 템플릿 태그가 많은 템플릿을 로드한다면, 로드된 템플릿을 캐시하는 방법을 고려할 수 있다. 

### 13.2.4 언제 템플릿 태그를 이용할 것인가 

#### 새로운 템플릿 태그를 추가하려고 할 때 고려할 사항들  
- 데이터를 읽고 쓰는 작업을 할 것이라면 모델이나 객체 메서드가 더 나은 장소일 것이다. 
- 프로젝트 전반에서 일관된 작명법을 이용하고 있기 때문에 추상화 기반의 클래스 모델을 core.models 모듈에 추가할 수 있다. 프로젝트의 추상화 기반 클래스 모델에서 어떤 메서드나 프로퍼티가 우리가 작성하려는 커스텀 템플릿 태그와 같은일을 하는가? 

#### 결론
새로운 템플릿 태그를 작성하기 좋은 때는 HTML을 렌더링하는 작업이 필요할 때이다. 

## 14.3 템플릿 태그 라이브러리 이름짓기 
#### 템플릿 태그 라이브러리 작명 관례  
`<app_name>_tags.py`

이 책의 예제에서 사용하는 이름들  

- flavors_tags.py
- blog_tags.py
- events_tags.py
- tickets_tags.py

> **! 주의**   
> 템플릿 태그 라이브러리 이름을 앱 이름과 똑같이 짓지 말자
> events 앱의 템플릿 라이브러리를 events.py로 지으면 장고가 템플릿 태그를 로드하는 방식으로 인해 문제가 생길 수 있다. 

## 14.4 템플릿 태그 모듈 로드하기 
~~~
{% extends "base.html" %}
{% load flavors_tags %}
~~~

### 14.4.1 반드시 피해야 할 안티 패턴 한가지 
~~~python
# 이 코드는 이용하지 말자!
# 악마 같은 안티 패턴이다!

from django import template
template.add_to_builtins("flavors.templatetags.flavors_tags")
~~~

이 안티 패턴은 **Don't Repeat Yourself, DRY** 라는 해결책을 제시하는 척 하면서 앞에서 이용된 명시적인 로드 메서드를 암시적인 방향으로 변경한다. 하지만 아래와 같은 문제점들로 인해 **DRY**의 장점은 무의미해진다.

- 템플릿 태그 라이브러리를 `django.template.Template`에 의해 로드된 모든 템플릿에 로드함으로써 오버헤드를 일으킨다. 이는 모든 상속된 템플릿, 템플릿 `{% include %}`, include_tag등에 영향을 미친다.
- 템플릿 태그 라이브러리가 암시적으로 로드되기 때문에 디버깅이 쉽지않다. **Explicit is better than Implicit** 
- `add_to_builtins` 메서드는 관례적으로 이용되지 않는다.

## 14.5 요약 
템플릿 태그와 필터들은 반드시 데이터를 표현하는 단계에서 데이터에 대한 수정을 위해서만 쓰여야 한다.