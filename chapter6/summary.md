# 장고에서 모델 이용하기 

### 한번 더 깊이 생각해 보고 프로젝트의 토대를 가능한 한 탄탄하고 안전하게 다질 수 있는 방향의 디자인을 고려해야한다.


##### 모델을 작업하며 이용하는 장고 패키지들

~~~
django-model-utils : 일반적인 패턴 처리에 이용 (e.g.TimeStampedModel)
django-extensions : 모든 앱의 모델 클래스를 자동으로 로드해주는 shell_plus 관리 명령 지원 (너무 많은 기능을 포함하고 있는 것이 단점)
~~~

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

~~~python
# core/models.py
form django.db import models

class TimeStampedModel(Models.Model):
	created = models.DateTimeField(auto_now_add=True)
	modified = models.DateTimeField(auto_now=True)
	
	class Meta:	
		abstract = True
~~~

TimeStampedModel은 추상화 기초 클래스로 선언했기 때문에 마이그레이션을 실행할 때 core_timestampedmodel 테이블이 생성되지 않는다.

#### 상속해 보기

~~~python
# flavors/models.py
from django.db import models

from core.models import TimeStampedModel

class Flavor(TimeStampedModel):
	title = models.CharField(max_length=200)
~~~

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

### 6.2.2 캐시와 비정규화 

### 6.2.3 반드시 필요한 경우에만 비정규화를 하도록 하자 

### 6.2.4 언제 널을 쓰고 언제 공백을 쓸 것인가 

### 6.2.5 언제 BinaryField를 이용할 것인가? 

### 6.2.6 범용 관계 피하기  

### 6.2.7 PostgreSQL에만 존재하는 필드에 대해 언제 널을 쓰고 언제 공백을 쓸 것인가 

## 6.3 모델의 _meta API 

## 6.4 모델 매니저 

## 6.5 거대 모델 이해하기 
### 6.5.1 모델 행동(믹스인) 

### 6.5.2 상태 없는 헬퍼 함수 

### 6.5.3 모델 행동과 헬퍼 함수 

## 6.6 요약 