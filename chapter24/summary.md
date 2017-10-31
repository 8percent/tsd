# 장고 성능 향상시키기
## 24.1 정말(성능이란 것이) 신경 쓸 만한 일일까?
사이트가 중소규모이고, 페이지 로딩 속도에 문제가 없다면 건너뛰어도 무방하다. 서툰 최적화는 해가 될 수 있다.    
사용자가 빠르게 증가하고있거나, 유명한 브랜드 사이트와 제휴를 맺을 예정이라면 고려하자.

## 24.2 쿼리로 무거워진 페이지의 속도개선

- 너무 많은 쿼리로 인한 병목 현상을 줄이는 방안
- 예상과 달리 느리게 반응하는 쿼리로 생긴 문제를 해결하는 방안  

> 먼저 읽어볼 것 : [데이터베이스 접속 최적화에 대한 django 공식 문서](https://docs.djangoproject.com/ko/1.11/topics/db/optimization/)

### 24.2.1 django-debug-toolbar를 이용하여 문제가 되는 쿼리 찾아내기
django-debug-toolbar로 쿼리의 출처와 아래와 같은 병목 구간을 찾는다.

1. 페이지에서 중복된 쿼리들
2. 예상보다 많은 양의 쿼리를 호출하는 ORM 호출
3. 느린 쿼리

> **프로파일링과 성능 분석을 위한 패키지들**  
 
> **django-cache-panel**  
장고에서 캐시의 이용을 시각화 하여 보여준다. [link]((https://pypi.python.org/pypi/django-cache-panel))  

> **django-extensions**  
장고의 프로파일링 도구를 활성화하여 run-server 명령을 시작하는 RunProfileServer 라는 도구를 제공한다. [link](http://django-extensions.readthedocs.io/en/latest/runprofileserver.html)  

> **silk**  
사용자에게 인터페이스를 보여주기 이전에 HTTP 요청과 데이터베이스 쿼리를 저장하여 실시간 프로파일링 가능하도록 한다. [link](https://github.com/mtford90/silk)

### 24.2.2 쿼리 수 줄이기
- **ORM에서 여러 쿼리를 조합하기**  
	ForeignKey, OneToOne 관계의 경우 `select_related()`를 이용   
	ManyToMany, ManyToOne 관계의 경우 `prefetch_related()`를 이용

- **중복 쿼리 처리하기**  
	템플릿 하나당 중복된 쿼리가 호출된다면 해당 쿼리를 파이썬 뷰로 이동시켜 콘텍스트(context) 자체를 변수로 처리하고, 이 콘텍스트 변수에서 템플릿 ORM이 호출될 수 있도록 해보자.

- **캐시 이용하기**  
	키/값 형식을 이용할 수 있는 캐시를 구현하거나 [Memcahced](https://memcached.org/) 등을 이용해보자.
<!--뷰 단에서 쿼리들을 실행해 볼 수 있는 테스트를 작성해보자. [(TransactionTestCase.assertNumQueries)](http://2scoops.co/1.8-test-num-queries)
-->

- `django.utils.functional.cached_property` 데코레이터를 이용하여 객체 인스턴스의 메서드 호출 결과를 메모리 캐시에 저장해보자.(29.3.5 django.utils.funcional.cached_property 참조)

### 24.2.3 일반 쿼리 빠르게 하기
개별 쿼리 속도를 높여 병목을 해결해보자.

- **인덱스로 최적화 하기**  
	- 쿼리에서 가장 빈번히 필터되고 정렬되는 항목에 대해 인덱스를 설정해보자. 
	- 실제 상용 환경에서 생성된 인덱스가 정확히 어떤 역할을 하는지 살펴보자. 개발 환경과 상용 환경이 완벽하게 일치하는 경우는 없다. 
	
- **쿼리에서 생성된 쿼리 계획(query plan)을 살펴보기**

- **느린 쿼리 확인하기**  
	데이터베이스에서 느린 쿼리 로깅(slow query logging)기능을 활성화 하여 빈번히 발생하는 느린 쿼리를 확인하자
	
- **느려질 가능성이 있는 쿼리 찾기**  
	django-debug-toolbar를 이용하여 상용 환경에 실제 적용되기 이전에 느려질 가능성이 있는 쿼리를 찾아내자.

#### 수정할 쿼리를 찾았다면?

- 가능한 작은 크기의 쿼리 결과가 반환되도록 로직을 재구성
- 인덱스가 좀 더 효과적으로 작동할 수 있도록 모델을 재구성
- SQL이 ORM에 의해 생성된 쿼리보다 더 효과적일 수 있는 부분에 SQL을 직접 이용

> **EXPLAIN ANALYZE/EXPLAIN 이용**  
> [PostgreSQL의 EXPLAIN ANALYZE](http://www.revsys.com/writings/postgresql-performance.html)   
> [MySQL의 EXPLAIN Syntax](https://dev.mysql.com/doc/refman/5.7/en/explain.html)

### 24.2.4 ATOMIC_REQUETST 비활성화하기 
병목 지점 분석 결과 트랜잭션에서 많은 지연이 나타난다면 세팅 변경을 고려해보아야 한다.(7.7.2 명시적인 트랜잭션 선언에서 해당 세팅에 대한 가이드라인 참고)
> 대부분 프로젝트에서 ATOMIC_REQUESTS 설정을 True로 설정해도 무방함  
모든 데이터베이스 쿼리를 트랜잭션으로 처리해도 부하 문제는 거의 인지할 수 없는 수준

## 24.3 데이터베이스의 성능 최대한 이용하기
데이터베이스 접근 최적화에 대해 고려해보자.

### 24.3.1 데이터베이스에서 삼가야 할 것들 
Resolution Systems의 Frank Wiles에 따르면 규모가 큰 사이트의 관계형 데이터베이스에 포함해서는 안되는 두 가지가 있다.

- **로그**  
	로그 데이터가 커지면서 DB 성능이 전체적으로 느려질 것이다.   
	로그에 대해 복잡한 질의를 해야한다면 [Splunk](https://www.splunk.com), [Loggly](https://www.loggly.com) 과 같은 서드파티 서비스를 이용하거나, 도큐먼트 기반의 NoSQL 데이터베이스를 추천.
	
- **일시적 데이터**  
	빈번하게 rewriting하는 데이터의 경우 관계형 데이터베이스 이용을 피하자. django.contrib.sessions, django.contrib.messages, metrics 데이터가 이에 해당함.  
	이러한 데이터는 Memcached, Redis, Riak 등 다른 비관계형 스토리지 시스템을 이용하자. 
	
> **바이너리 데이터**  
> 도 포함되어서는 안되는 것이라고 했음. 바이너리 데이터를 DB에 저장하면 django.db.models.FileField에 의해 처리되는데 이는 AWS 클라우드 프론트나, S3처럼 파일 서버에 파일을 저장하는 것과 같은 작동을 하게됨. 예외적으로 저장하는 경우는 6.2.5 언제 BinaryField를 이용할 것인가?에서 다뤘음.

### 24.3.2 PostgreSQL 최대한 이용하기
PostgreSQL을 이용한다면 상용 서비스 환경 세팅에 주의하자.

- [Detailed installation guides](https://wiki.postgresql.org/wiki/Detailed_installation_guides)
- [Performance Tuniing PostgreSQL](http://www.revsys.com/writings/postgresql-performance.html)
- [Understanding Postgres Performance](http://www.craigkerstiens.com/2012/10/01/understanding-postgres-performance/)
- [More on Postgres Performance](http://www.craigkerstiens.com/2013/01/10/more-on-postgres-performance/)

> 더 자세한 정보는 [Postgres9.6 High Performance](https://www.amazon.com/PostgreSQL-High-Performance-Ibrar-Ahmed/dp/1784392979)를 읽어보자

### 24.3.3 MySQL 최대한 이용하기 
MySQL은 상대적으로 이용하기 쉬운 서버이지만 상용 서비스 환경에 최적화하여 설치하는 것에는 경험과 시스템에 대한 깊은 이해가 필요하다.  

> [High Performance MySQL](http://amzn.to/188VPcL)을 읽어보자


## 24.4 Memcached나 레디스를 이용하여 쿼리 캐시하기
간단한 세팅만으로 장고의 내장 캐시 시스템을 Memcached나 레디스와 연동할 수 있다.   


```
 　∧＿∧  　
（｡･ω･｡)つ━☆・*。  
⊂　　 ノ 　　　・゜+.  
　しーＪ　　　°。+ *´¨)  
　　　　　　　　　.· ´¸.·*´¨) ¸.·*¨) 
　　　　　　　　　　(¸.·´ (¸.·'* ☆  엄청난 성능 향상
```
　　　　　　　　　　

Memcached, 레디스와 바인딩 할 수 있는 파이썬 패키지 설치 후 프로젝트에서 설정하면 끝.

> 참고  
> [Django’s cache framework](https://docs.djangoproject.com/en/1.11/topics/cache/)  
> [django-redis-cache](https://github.com/sebleier/django-redis-cache)

## 24.5 캐시를 이용할 곳 정하기
#### 캐시를 이용할 곳을 정할 때 고려해야할 사항들

- 가장 많은 쿼리를 포함하고 있는 뷰와 템플릿
- 가장 많은 요청을 받는 URL
- 캐시를 삭제해야할 시점은 언제일까?

## 24.6 서드 파티 캐시 패키지
#### 서드 파티 캐시 패키지가 제공하는 기능들
- QuerySet 캐시
- 캐시 삭제 세팅과 메커니즘
- 다양한 캐시 백엔드

#### 인기있는 장고 캐시 패키지
- django-cache-machine
- johnny-cache

더 다양한 옵션은 [여기](https://djangopackages.org/grids/g/caching/)를 참고.

> **서드 파티 캐시 라이브러리를 너무 신뢰하지는 말자**

## 24.7 HTML, CSS, 자바스크립트 압축과 최소화하기 
웹 페이지 렌더링에 사용하는 HTML, CSS, 자바스크립트, 이미지 파일들은 이용자의 네트워크를 잡아먹고 페이지를 느리게 한다.  

**압축과 최소화**  
django에는 `GZipMiddleware` 와 `{$spaceless$}` 템플릿 태그를 지원한다.
또한 WSGI 미들웨어를 이용해 이와 같은 작업을 처리할 수도 있다.  
위와 같은 장고와 파이썬 자체 압축, 최소화 작업의 문제점은 해당 작업에 시스템 리소스를 많이 사용하게 되면 압축과정이 병목지점이 될 수 있다는 것.  

Apache, Nginx와 같은 웹서버를 이용해 외부로 나가는 컨텐츠를 압축하는것이 더 나은 방안이다.

저자는 서드 파티 장고 라이브러리를 이용하여 CSS와 자바스크립트 압축과 최소화를 미리 처리하는 것을 선호. (장고 코어 개발자 야니스 라이델의 django-pipline)

> 라이브러리 도구에 대한 참고자료  
- 아파치와 Nginx의 압축 모듈  
- django-pipline  
- django-compressor  
- django-htmlmin  
- 장고에 내장된 [spaceless](https://docs.djangoproject.com/ko/1.11/ref/templates/builtins/#spaceless) 태그   
- [Asset Managers](https://djangopackages.org/grids/g/asset-managers/)


## 24.8 업스트림 캐시나 CDN 이용하기 
**upstream chache**  
웹서버 앞단에서 구동하며 웹페이지나 컨텐츠를 빠르게 제공한다.  

- [Varnish](http://varnish-cache.org/) 

**CDN(content delivery network)**  
이미지, 비디오, CSS, 자바스크립트와 같은 정적 미디어를 서비스해준다. 
앱에서 정적 콘텐츠를 서비스하는 것 보다 CDN을 사용함으로써 프로젝트 속도를 향상시킬 수 있다. 

- Fastly  
- Akamai  
- Amazone cloud front  



## 24.9 기타 다른 참고 자료들 
스케일링, 성능, 튜닝, 최적화에 대한 깊이 있는 기술들은 책의 범위를 벗어나기에 몇 가지 출발점을 제시한다.

- [Yslow의 Web Performance Best Practices and Ruels.](http://developer.yahoo.com/yslow)
- [구글의 Web Performance Best Practice.](https://developers.google.com/speed/docs/best-practices/rules_intro)
- [High Performances Django](https://highperformancedjango.com)
- [DjangoCon, PyCon에 올라온 다른 개발자들의 경험들](http://lanyrd.com/search/?q=django+scaling)

## 24.19 요약 
- 페이지와 쿼리 프로파일링
- 쿼리 최적화
- 데이터베이스 잘 이용하기
- 쿼리 캐시하기
- 어떤 것들을 캐시할 것인가?
- HTML, CSS, 자바스크립트 압축하기

