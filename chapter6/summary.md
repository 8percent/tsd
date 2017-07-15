# 장고에서 모델 이용하기 

### 한번 더 깊이 생각해 보고 프로젝트의 토대를 가능한 한 탄탄하고 안전하게 다질 수 있는 방향의 디자인을 고려해야한다.


##### 모델을 작업하며 이용하는 장고 패키지들

```
django-model-utils : 일반적인 패턴 처리에 이용 (e.g.TimeStampedModel)
django-extensions : 모든 앱의 모델 클래스를 자동으로 로드해주는 shell_plus 관리 명령 지원 (너무 많은 기능을 포함하고 있는 것이 단점)
```

## 6.1 시작하기 

### 6.1.1 모델이 너무 많으면 앱을 나눈다 
앱 하나에 모델이 스무 개 이상 있다면 크기가 작은 앱으로 분리하는 것을 고려해야한다. 

**저자의 의견**  
각 앱이 가진 모델의 수가 다섯 개를 넘지 않아야 한다.

### 6.1.2 모델 상속에 주의하자 

#### 장고에서 제공하는 모델 상속 방법

- 추상화 기초 클래스(abstract base class)
- 멀티테이블 상속(multitable inheritance)
- 프락시 모델(proxy model)

 
| 모델 상속 스타일 | 장점 | 단점 |
| --- | --- | --- |
| 추상화 기초 클래스 | 공통적인 부분을 추상화된 클래스로 만들기 때문에 한 번만 작성하면 됨 | 부모 클래스를 독립적으로 사용할 수 없음 |
| 멀티테이블 상속 | 각 모델에 대해 매칭되는 테이블이 생성되기 때문에, 부모 또는 자식 모델 어디로든 쿼리를 할 수 있음 | 자식 테이블에 대한 각 쿼리에 대해 부모 테이블로의 조인이 필요하여 **상당한 부하**가 발생하므로 **이용하지 않는 것을 권함** |
| 프락시 모델 | 각기 다른 파이썬 작용(behavior)을 하는 모델들의 별칭을 가질 수 있다 | 모델의 필드를 변경할 수 없다 |

#### 어떤 종류의 상속이 언제 이용될까?

- 모델 사이 중복되는 내용이 최소라면 상속은 필요하지 않음
- 모델들 사이 중복된 필드가 많으면, 공통 필드 부분을 추상화 기초모델로 이전할 수 있게 리팩터링 
- 프락시 모델은 다른 두 가지 모델 상속 방식과는 다르게 작동한다는 점을 주의
- 멀티테이블 상속은 혼란과 상당한 부하를 일으키므로 반드시 피하자!  
	대신 `OneToOneField` 또는 `ForeignKeys`를 이용하여 컨트롤할 수 있다.

### 6.1.3 실제로 모델 상속해 보기: TimeStampedModel

장고의 모든 모델에서 create와 modified 타임스탬프 필드 생성하는 것은 일반적이다.

#### 추상화 기초 클래스 만들기

```python
# core/models.py
form django.db import models

class TimeStampedModel(Models.Model):
	created = models.DateTimeField(auto_now_add=True)
	modified = models.DateTimeField(auto_now=True)
	
	class Meta:	
		abstract = True
```
TimeStampedModel은 추상화 기초 클래스로 선언했기 때문에 마이그레이션을 실행할 때 core_timestampedmodel 테이블이 생성되지 않는다.

#### 상속해 보기

```python
# flavors/models.py
from django.db import models
from core.models import TimeStampedModel

class Flavor(TimeStampedModel):
	title = models.CharField(max_length=200)
```
flavors_flavor 테이블만 생성이 된다.

### 6.1.4 데이터베이스 마이그레이션 

#### 마이그레이션을 생성하는 팁 

- 새로운 앱 또는 모델이 생성되면 `python manage.py makemigrations`로 마이그레이션 코드를 생성한다.
- 마이그레이션 코드가 생성되면 실행하기 전에 생성된 코드를 점검하자.  
	 `sqlmigrate`을 이용하면 어떤 SQL문이 실행되는지 확인할 수 있다.
- 자체적인 django.db.migrations 스타일로 이루어지지 않은 외부 앱에 대해 마이그레이션을 처리하려면 `MIGRATION_MODULES` 세팅을 이용한다.
- 마이그레이션 개수가 너무 많아 불편하다면 `squashmigration`을 이용한다.

#### 마이그레이션의 배포와 관리 

- 배포 전 rollback 할 수 있는지 확인하자.
- 테이블에 수백만 개의 데이터가 존재한다면 스테이징 서버에서 비슷한 크기의 데이터에 대해 충분히 테스트하자.
- MySQL을 이용한다면
	- 스키마 변환 전 반드시 데이터베이스를 백업한다. (스키마 변경에 대해 트랜잭션을 지원하지 않으므로 롤백이 불가능함)
	- 가능하다면 데이터베이스 변환 이전에 프로젝트를 읽기 전용(read-only) 모드로 전환해둔다.
	- 큰 테이블의 경우 주의하지 않으면 스키마 변경에 상당한 시간이 소요된다.

## 6.2 장고 모델 디자인 
### 6.2.1 정규화하기 
#### 데이터베이스 정규화
- [Database normalization](http://en.wikipedia.org/wiki/Database_normalization)
- [Relational Database Design](http://en.wikibooks.org/wiki/Relational_Database_Design/Normalization)

이미 모델에 포함된 데이터들이 중복되어 다시 다른 모델에 포함되지 않도록 한다.

### 6.2.2 캐시와 비정규화 
`24장 장고 성능 향상시키기`에서 자세히 다룬다.

### 6.2.3 반드시 필요한 경우에만 비정규화를 하도록 하자 
`24장 장고 성능 향상시키기`에서 제시한 방법들로도 해결할 수 없을 때 비정규화에 대해 고민해도 늦지않다.

### 6.2.4 언제 널을 쓰고 언제 공백을 쓸 것인가 
모델 필드 인자들에 대한 가이드 참고하기

- django 표준은 빈 값(empty value)을 빈 문자열(empty string)로 저장한다.  
- BooleanField 에는 NullBooleanField를 사용한다. 

> IPAddressField 대신 GenericIPAddressField를 사용하자.

### 6.2.5 언제 BinaryField를 이용할 것인가? 
#### BinaryField
- raw binary data 또는 byte를 저장하는 필드
- filter, exclude, 기타 SQL 액션들이 적용되지 않음

#### Using
- 메시지팩 형식의 콘텐츠ch
- 원본 센서 데이터
- 압축된 데이터


> 바이너리 데이터는 크기가 방대할 수 있고, 이로 인하여 데이터베이스가 느려질 수도 있다.  
> **BinaryField로 부터 파일을 직접 서비스 하는 것은 금물!**

### 6.2.6 범용 관계 피하기  
####범용 관계
한 테이블로부터 다른 테이블을 서로 `제약 조건 없는 외부 키 (unconstrained foreign key, GenericForeignKey)`로 바인딩 하는 것.

#### 발생할 수 있는 문제점 
- 모델 간의 인덱싱이 존재하지 않으면 쿼리 속도에 손해를 가져온다.
- 다른 테이블에 존재하지 않는 레코드를 참조할 수 있는 데이터 충돌의 위험성이 존재함.

#### 장점 
- 기존에 만들어 둔 여러 모델 타입과 상호작용을 하는 앱을 새로 제작할 때 수월해진다.  
예) 즐겨찾기, 평점 매기기, 투표, 메시지, 태깅 등

> 제약 없이 연결된 부분이 프로젝트에서 중요한 데이터를 처리하게 되면 문제가 심각해질 수 있다.

#### 추천방법
- 범용 관계와 GenericForeignkey 이용을 피한다.
- 범용 관계가 필요하다면, 모델 디자인을 바꾸거나 새로운 PostgreSQL 필드로 해결할 수 있는지 확인한다.
- 불가피한 경우 서드 파티 앱을 고려해보자.


### 6.2.7 PostgreSQL에만 존재하는 필드에 대해 언제 널을 쓰고 언제 공백을 쓸 것인가 

## 6.3 모델의 _meta API 

`_meta` 의 원래 목적: 모델에 대한 부가적인 정보를 장고 내부적으로 이용하는 것   

- [모델 _meta 문서](http://docs.djangoproject.com/en/1.8/ref/models/meta/)
- [모델 _meta 에 대한 장고 1.8 릴리스 노트](https://docs.djangoproject.com/en/1.8/releases/1.8/#model-meta-api)


## 6.4 모델 매니저 
모델에 질의하면 장고의 ORM을 통한다. 이 때 모델 매니저를 호출하게됨.
### model manager
- 데이터베이스와 연동하는 인터페이스  
- 원하는 클래스의 제어를 위해 모델 클래스의 모든 인스턴스 세트에 작동

#### Custom model manager 예제

##### 생성

```python
from django.db import models
from django.utils import timezone

class PublishedManager(models.Manager):
	
	use_for_related_fields = True
	
	def published(self, **kwargs):
		return self.filter(pub_date__lte=timezone.now(), **kwargs)
		

class FlavorReview(models.Model):
	review = models.CharField(max_length=255)
	pub_date = models.DateTimeField()
	
	# 커스텀 모델 매니저를 추가
	objects = PublishedManager()
```

##### 사용 

```python
>>> from reviews.models import FlavorReview
>>> FlavorReview.objects.count()
35
>>> FlavorReview.objects.published().count()
31
```

### 기존 모델 매니저를 교체할 때 주의할 것
- 모델을 상속받아 이용 시 추상화 기초 클래스의 자식들은 부모 모델의 매니저를 받고, 접합 기반 클래스의 자식들은 부모 모델의 매니저를 받지 못한다.
- 모델 클래스에 적용되는 첫 번째 매니저는 장고가 기본값으로 취급하는 매니저이다. : 파이썬의 일반벅인 패턴을 무시하는 것이기에 Queryset으로 부터의 결과를 예상할 수 없게 된다.

**objects = models.Manager() 는 항상 새로은 커스텀 모델 매니저 위에 정의하자.**  

> [추가적인 읽을거리](https://docs.djangoproject.com/en/1.11/topics/db/managers/)


## 6.5 거대 모델 이해하기 

### 거대 모델 (fat model)

**코드 재사용을 개선할 수 있는 최고의 방법**  
데이터 관련 코드를 모델 메서드, 클래스 메서드, 프로퍼티, 매니저 메서드 안에 넣어 캡슐화 하는 것.

##### 문제점
- 모델 코드의 크기를 신의 객체(god object) 수준으로 증가시킨다.
- 복잡해지므로 코드의 이해, 테스트 또는 유지보수에 어려움을 겪게 된다.

> 모델의 크기가 너무 커지면, 코드를 분리한다.    
> 메서드, 클래스 메서드, 프로퍼티들을 유지하고 해당 로직을 모델 행동(model behavior) 또는 상태 없는 헬퍼 함수(stateless helper function)으로 이전.

### 6.5.1 모델 행동(믹스인) 
- [코드 중복을 막는 작성법 - kevin Stone](http://blog.kevinastone.com/django-model-behaviors.html)
- 10.2 클래스 기반 뷰와 믹스인 이용하기

### 6.5.2 상태 없는 헬퍼 함수 
- 로직을 유틸리티 함수로 작성하면 독립적 구성이 가능
- 로직에 대한 테스트가 쉬워짐
- stateless 이므로 함수에 더 많은 인자를 필요로 하게 된다.

> `29장 유틸리티들에 대해`를 참고

### 6.5.3 모델 행동과 헬퍼 함수 
완벽하진 않지만 적절히 사용한다면 도움이 된다.

## 6.6 요약 

- 모델은 신중하게 디자인하자
- 다른 선택지를 충분히 고려 후 방법이 없을 경우에 비정규화를 고려하자
- 인덱스를 사용하자
- 상속은 접합 모델(concrete model)이 아닌 추상화 기초 클래스로 부터 상속하자
- null=True, blank=True 옵션은 가이드라인을 참고
- 거대 모델은 모델 전부를 신의 객체가 될 위험도 있다.

