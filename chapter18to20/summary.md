## 18. Tradeoffs of Replacing Core components

짧은 대답 : 할 이유가 없다. 

긴 대답 : 

- 서드파티 django 패키지들을 포기할 수 있다면
- django-admin을 버려도 문제가 없다면
- 이미 django로 할 수 있는 많은 시도를 했지만 해결할 수 없는 벽을 만났을 때
- 이미 문제를 해결하기 위한 코드를 분석하여 문제를 해결하기 위한 원인을 찾았을 때 
- 캐싱, 비정규화 등의 옵션들을 다 고려하였을 때
- 프로젝트가 거대한 유저들이 이용하고 있는, 진짜 라이브 프로덕션이라, '빠른 최적화'가 아니라고 확신하는 경우
- SOA를 고려해보았지만 Django에서 다루는데 문제가 있어 보류중일 때
- Django를 업그레이드하며 계속 이용하는게 엄청나게 고통스럽거나 불가능해도 감수할 수 있을 때 




### 18.1 The temptation to Build FrankenDjango

매년, django component를 새로운 것으로 교체하려는 유행에 민감한 개발자들이 있다. 그리고 보아왔던 유행들을 요약해본다.

| Fad                     | Reasons                                  |
| ----------------------- | ---------------------------------------- |
| 성능의 이유로 NoSQL로 간다.      | - 설득력 없는 이유 : 앞으로 많은 사용자가 생길 것이다.        |
|                         | - 설득력 있는 이유 : 이미 우리는 5000만의 사용자가 있다.     |
| 데이터 프로세싱을 위해 NoSQL로 간다. | - 설득력 없는 이유 : SQL은 형편 없다. MongoDB같은 도큐먼트 기반 db를 쓰고싶다 |
|                         | - 설득력 있는 이유 : psql에서 지원되는 hstore 데이터 복제 기능이 MongDB에서 제공되는 스토리지 시스템과 흡사하지만, 기본적으로 제공되는 맵 리듀스 기능을 쓰고 싶다. |
| DTL을 Jinja2나 Mako로 바꾼다. | - 설득력 없는 이유 : 해커뉴스에서 빠르다고 하니까 그걸로 바꾸자 ; 템플릿 안에 로직을 넣도록 해달라! |
|                         | - 설득력 있는 이유 : 무거운 페이지가 상당히 있는데, 성능을 위해 무거운 페이지만 Jinja2로 렌더링하려 한다. |



### 18.2 Non-Relational Databases vs. Relational Databases

영속 데이터 저장을 RDB로 하고 있는 Django 프로젝트 조차 비관계형 database에 의존한다. 캐싱을 위해 Memcached를 사용하거나 queuing에 redis를 쓰고있다면, 비관계형 database를 쓰고 있는 것이다. 

면밀히 고려하지 않고 NoSQL으로 완전히 대체하여 사용할 때 문제가 발생한다.



#### 18.2.1 모든 비관계형 database가 ACID를 충족하진 않는다

ACID는, 다음의 속성을 가진다. 

- 원자성
- 일관성
- 고립성
- 지속성

대부분 비관계형 database는 위 속성이 약하거나 없기 때문에 데이터가 오염될 위험이 높다.



#### 18.2.2 관계형 작업에 비관계형 database를 사용하지 마라

하려는 작업의 성격을 따져보고 사용하여야 한다.



#### 18.2.3 유행을 무시하고 스스로 연구를 해보아라

유행을 무작정 따르기보다 현재 프로젝트에 적합한지 면밀히 검토 후, 벤치마크를 해본다. 또한 성공-실패 사례들을 조사해보고 작은 프로젝트에 충분히 실험한 뒤 메인코드에 적용하자.



#### 18.2.4 우리는 비관계형 database를 django와 어떻게 이용하는가

- cache, queue 혹은 비정규화가 필요한 데이터를 다룰때나 짧은 순간 데이터를 저장할 때 제한적으로 사용한다. 
- 장기적인 데이터 보관과 비정규화 데이터가 필요할 때 이용한다.




### 18.3 Django Template 언어를 바꾸는 것은 어떨까?

15장에서 이미 다루었다.





## 19 Working with the Django Admin

Django가 주는 최고의 장점은? 어.드.민.



### 19.1 End User를 위한 것은 아니다

잘 만들어져 있어보이지만, 관리자를 위한 기능이지 최종 사용자를 위한 기능은 아니다. 



### 19.2 Admin Customization Vs. New Views

커스터마이징 하는 것보다 새로운 View를 만드는 것이 더 편할 수도 있다. 



### 19.3 객체 이름 보여주기

Django admin 페이지에서는 기본적으로 객체의 이름을 문자열로 표현해서 보여준다.



####19.3.1 \_\_String\_\_() 사용하기

Django model에 \_\_String()\_\_ 을 구현함으로써, 객체를 읽기 쉬운 상태로 만들 수 있다. 

```python
IceCream.objects.all()
[<IceCream object>, <IceCream object>, <IceCream object>]

IceCream.objects.all()
[<IceCream: 바닐라맛>, <IceCream: 파맛>, <IceCream: 체리맛>]
```



#### 19.3.2 list_display 사용하기

admin list page에서 보여줄 field를 정의할 수 있다.



### 19.4 ModelAdmin class에 Callable 추가하기

Django ModelAdmin에 호출 가능한 method나 function을 차가할 수 있다. 이를 통해 프로젝트에 더 적합하도록 수정할 수가 있다. 

```python
@admin.register(IceCream)
Class IceCreamModelAdmin(admin.ModelAdmin):
	...
    readonly_fields = ('show_url',)
    
    def show_url(self, instance):
    	url = reverse('ice_cream_detail', kwargs={'pk': instance.pk})
        response = format_html("""<a href='{0}'>{0}</a>""", url)
        return response

    show_url.short_description = 'ice cream url'
    show_url.allow_tags = True
```



### 19.5 여러 사용자가 이용하는 환경의 복잡함을 인식하자

Django admin에는 Lock을 거는 기능이 없기 때문에 심각한 문제가 발생할 수 있다. 



### 19.6 Django admin 문서 생성기

django.contrib.admindocs 패키지를 통해서 문서화를 할 수 있다. 23장에서 문서화 도구들을 소개하겠지만, 여전히 유용하다.

심지어 프로젝트의 구성 요소에 아무런 docstring이 포함되어 있지 않더라도 단순히 이상한 이름의 custom template tag나 custom filter 등을 보는 것은 복잡한 기존 application의 아키텍처를 탐색하는데 도움이 된다. 



### 19.7 Custom Skin 사용하기

- django-grappelli
- django-suit
- django-admin-bootstrapped

단순히 CSS만 수정해서 되는 것이 아니라, 새로운 Custom theme을 만드는 것은 매우 어려운 일이다. 이런 프로젝트(custom theme)를 개발해본 사람은 django.contrib.admin을 보면 해당 구조에 맞게 요구되는 특이한 비밀의 코드들이 있음을 알 수 있다.

Django-grappelli의 개발자가 [쓴 글](http://sehmaschine.net/blog/django-admin-frontend-framework)을 보면 인터페이스에 대한 내용을 다루고 있다. 



#### 19.7.1 평가 포인트: 문서화가 전부다

앞서 admin custom theme을 만드는 것이 쉬운일이 아님을 이야기했다. 완성된 스킨은 비교적 쉽게 프로젝트에 추가할 수 있지만, 그것이 번뇌의 길로 빠지는 길이기도 하다, 마치 Model admin을 확장하는 것처럼.

그로므로 문서화가 잘 되어있는지 확인하자.



#### 19.7.2 만든 Admin extension에 대한 Test를 작성하라 

우리의 목적대로, 고객들은 현대적인 theme에 더 만족하겠지만, custom skin이 어디까지 확장될 수 있을지 주의를 기울여야 한다. 기본 admin에서 잘 동작하던 기능이 custom skin에서는 동작하지 않을 수 있다. 그러므로 best practice는 admin에서 어떤 변경이 있었다면, 테스트를 꼭 작성하는 것이다.



### 19.8 Django admin 보안

Django admin은 관리자에게 슈퍼파워를 제공해주므로, 철통 보안을 만드는 것이 좋다.



#### 19.8.1 기본 admin url을 바꾸기

*http://yoursite.com/admin/*으로 되어있는 주소를 변경하자.



#### 19.8.2 Django-admin-honeypot을 써라

꿀 없는 [꿀단지](https://github.com/dmpayton/django-admin-honeypot)를 제공해줌으로써 공격을 방어할 수 있다. 



#### 19.8.3 HTTPS를 통해서만 접근하도록 하자

#### 19.8.4 IP로 접근을 제한하자

web server level (e.g. Nginx) 에서 분리하는 것이 좋다. 





## 20 Dealing With the User Model

### 20.1 user model 찾기위해 Django tool을 사용하기

```python
from django.contrib.auth import get_user_model
get_user_model()
```

get_user_model()은 프로젝트 설정에 따라 각기 다른 두개의 user model을 리턴해준다. 그러나 이것이 프로젝트가 2개의 유저모델을 가질 수 있음을 의미하진 않는다.



#### 20.1.1 User Foreign key에는 settings.AUTH_USER_MODEL 을 사용하라

[스택오버플로우: [get_user_model vs settings.AUTH_USER_MODEL]](https://stackoverflow.com/questions/24629705/django-using-get-user-model-vs-settings-auth-user-model)를 참고.



### 20.2 Django 1.11에서의 Custom User fields

> [django-authtools](https://github.com/fusionbox/django-authtools) : custom user model을 쉽게 만들 수 있도록 도와주는 library. 실제로 이용하지 않더라도 코드를 살펴볼 가치가 있다.



#### 20.2.1 Subclass AbstractUser 

Django user model을 그대로 이용하면서 추가로 필드가 필요할 때 쉽게 사용할 수 있는 옵션이다. 새롭게 프로젝트를 시작할 때 접근할 수 있는 첫번 째 방법이다. 쉽고 빠르다.

```python
# profiles/models.py
from django.contrib.auth.models import AbstractUser

class ParPerUser(AbstractUser):
    money = models.IntegerField(default=10000000)
    
# Settings
AUTH_USER_MODEL = 'profiles.ParPerUser'
```



#### 20.2.2 Subclass AbstractBaseUser

가장 최소한의 base 모델로부터 시작한다. password, last_login, is_active 를 제외하고는 모두 새로 만들어야 한다. 



####  20.2.3 관련된 모델로부터 Linking back

Django 1.5에서 profile model을 만드는 테크닉과 매우 유사하다. legacy로서 버려졌지만, 아래와 같은 경우에는 고려해볼만 하다.

- PyPI에 올리기 위해 서드파티 패키지를 만드는 경우
- 패키지가 사용자마다 추가적인 정보를 저장해야 할 필요가 있는 경우
- 기존 프로젝트 코드를 최대한 건드리고 싶지 않은 경우
