## Chapter15. Django Templates and Jinja2

Django 1.8부터 multi template engine이 지원된다. 지금 django template system에서 사용할 수 있는 내장 백엔드는 DTL(Django template language)과 Jinja2뿐이다. 



### 15.1 문법적인 차이

Jinja2가 DTL로부터 영감을 얻었기 때문인지 문법적으로 매우 비슷하다. (209p, Table 15.1 참고)



### 15.2 바꿔야하나요?

settings.TEMPLATES 설정을 통해 혼용할 수 있기 때문에 둘중 하나를 선택 해야 할 이유가 없다. 예를들면 DTL로 작성된 기존 템플릿들은 유지한 채 필요한 부분에 Jinja2를 사용함으로써 Jinja2의 장점만을 취할 수 있다.




#### 15.2.1 DTL의 장점

- Django docs에 쉽고 명확하게 문서화 되어있으며, 예제 코드도 많아 쉽게 따라하기 좋다.
- DTL+Django 조합이 Jinja2+Django 조합보다 검증되어있고 성숙하다.
- 대부분 third-party app은 DTL을 사용하므로, jinja2로 바꾸는데 추가 작업이 필요하다.
- 많은 양의 DTL을  Jinja2로 바꾸는데 큰 작업이 필요하다.





#### 15.2.2 Jinja2의 장점

- Django와 독립적으로 사용 가능하다.
- Python 문법과 비슷하여 이해하기 쉽다.
- 더 명시적이다. (e.g. function call을 할 때 괄호를 사용)
- Logic에 임의의 제약이 덜하다. Filter에 DTL의 경우 1개의 인자만 사용할 수 있지만 Jinja2는 무한대로 사용 가능하다. 
- 벤치마크 성능과 경험적으로 봤을때 일반적으로 Jinja2가 더 빠르다.




####  15.2.3 승자는?

- 뉴비는 DTL을 씁시다.
- 이미 많은 양의 코드로 작성된 프로젝트의 경우 성능 개선이 필요한 몇 페이지를 제외하고는 계속 DTL을 쓰는 것을 원할 것이다. 
- Djangonauts들은 둘다 시도해보고, 비교해본 뒤 결정을 내려보세요.





### 15.3 Jinja2를 사용할 때 고려해야 할 것

#### 15.3.1 CSRF and Jinja2

Jinja2에서는 CSRF를 포함시키기 위해서 특별한 HTML이 필요하다. (212p, Example 15.1 참고)



#### 15.3.2 Jinja2에서 template tag 사용하기

Django style의 template tag는 사용이 불가능 하다. 따라서 특정 템플릿 태그의 기능이 필요하다면 아래 방법을 통해 변환을 해야 한다.

- 기능을 function으로 변환한다.
- Jinja2 Extension을 작성한다. [참고: jinja.pocoo.org/docs/dev/extensions/#module-jinja2.ext](http://jinja.pocoo.org/docs/dev/extensions/#module-jinja2.ext)



#### 15.3.3 Jinja2 template에서 DTL filter 사용하기

Django의 filter들은 그저 function이기 때문에, 쉽게 커스텀 Jinja2 환경에 템플릿 필터들을 포함시킬 수 있다.

```python
# core/jinja2.py

from django.contrib.staticfiles.storage import staticfiles_storage
from django.template import defaultfilters
from django.urls import reverse

from jinja2 import Environment

def environment(**options):
  env = Environment(**options)
  env.globals.update({
      'static': staticfiles_storage.url,
      'url': revers,
      'dj': defaultfilters
  })
  return env
```

DJango template filter를 Jinja2 template에서 사용하는 예

```html
<table>
  <tbody>
    {% for purchase in purchase_list %}
      <tr>
        <a href="{{ url('purchase:detail', pk=purchase.pk) }}">
          {{ purchase.title }}
        </a>
      </tr>
      <tr>
        {{ dj.date(purchase.created, 'SHORT_DATE_FORMAT') }}
      </tr>
      <tr>
        {{ dj.floatformat(purchase.amount, 2) }}
      </tr>
    {% endfor %}
  </tbody>
</table>
```



덜 전역적으로 사용하고 싶다면 mixin을 생성해서 사용하는 방법이 있다.

```python
from django.template import defaultfilters

class DjFilterMixin:
  dj = defaultfilters
```

```html
...
<tr>
  {{ view.dj.date(purchase.created, 'SHORT_DATE_FORMAT') }}
</tr>
<tr>
  {{ view.dj.floatformat(purchase.amount, 2) }}
</tr>
...
```



##### Tip

> [Django 문서](https://docs.djangoproject.com/en/1.11/topics/templates/#django.template.backends.jinja2.Jinja2)를 보면 Jinja2에서 context processors를 사용을 권장하지 않는다고 한다. context processor가 유용한 이유는 DTL에서 여러 인자를 사용한 function을 허용하지 않기 때문인데, Jinja2는 그런 제약이 없으므로 필요한 곳에서 function을 호출하여 사용하면 된다. 



#### 15.3.4 Environment object는 정적으로 간주된다 

15.1에서 Jinja2 Environment class를 사용하는 것을 보았는데, 이 object는 jinja2의 설정, 필터, 테스트, 전역값 등등을 공유하는 곳이다. 프로젝트에서 첫 템플릿이 로드되면 Jinja2 는 해당 클래스를 static object로 instantiate한다.