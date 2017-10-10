# 코어
## [Sphinx](http://www.sphinx-doc.org/en/stable/index.html)
파이썬 프로젝트 문서화 도구 프로젝트 (예시: [파이썬 doc](https://docs.python.org/3/))<br>
- [기존 프로젝트 문서화](http://www.sphinx-doc.org/en/stable/invocation.html#invocation-apidoc)
~~~
$ sphinx-quickstart [options] [projectdir]
~~~
- Sphinx는 Python Style Guide 중, ReStructuredText 형식을 기본적으로 지원하지만 [Napoleon](http://www.sphinx-doc.org/en/stable/ext/napoleon.html)을 
사용하면 Google 형식도 사용가능하다.

# 비동기
## [celery](http://docs.celeryproject.org/en/latest/index.html)
분산 테스크 큐, 사실상 장고의 표준

## [flower](https://flower.readthedocs.io/en/latest/)
celery 테스크 관리 및 모니터링 도구

## [rq](http://python-rq.org/docs/)
가벼우면서 간단한 백그라운드 작업 생성과 프로세싱 라이브러리

## [django-background-tasks](http://django-background-tasks.readthedocs.io/en/latest/)
데이터베이스 기반의 비동기 테스크 큐

## 테스크 큐 소프트웨어 선택하기(25.2) - tsd에서의 추천
- 용량이 작은 프로젝트부터 용량이 큰 프로젝트까지 대부분 레디스 큐를 추천
- 테스크 관리가 복잡한 대용량 프로젝트에는 셀러리 추천
- 특별히 시간이 주기적인 배치 작업을 위한 소규모프로젝트에는 django-background-tasks를 추천
<br>
다른 의견: https://stackoverflow.com/questions/13440875/pros-and-cons-to-use-celery-vs-rq

# 데이터베이스

## [psycopg2](http://initd.org/psycopg/docs/)
PostgreSQL 데이터베이스 어댑터

## [django-maintenancemode-2](https://github.com/alsoicode/django-maintenancemode-2)
Maintainance Mode 로의 전환을 쉽게 해줌

# 로깅
## [logutils](http://pythonhosted.org/logutils/)
로깅에 유용한 핸들러를 추가해준다.

## [Sentry]


