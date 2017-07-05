# 2. 최적화된 장고 환경 꾸미기

## 2.1. 같은 데이터베이스를 이용하라.
다른 데이터베이스를 이용할 경우 아래와 같은 문제를 겪을 수 있다.
1. 운영 데이터를 완전히 똑같이 로컬에서 구동할수 없다
2. 다른 종류의 데이터베이스 사이에는 다른 성격의 필드 타입과 제약 조건이 존재한다
 최적의 조합: 장고 + PostgreSQL
3. 픽스쳐(Fixture)는 데이터 이전을 위한 적절한 툴이 아님

## 2.2. pip와 virtualenv 이용하기
* pip: 파이썬 패키지 인덱스와 그 미러 사이트에서 파이썬 패키지를 가져오는 도구, virtualenv 지원
* virtualenv: 파이썬 패키지 의존성을 유지할 수 있도록 독립된 파이썬 환경을 제공하는 도구
 - virtualenvwrapper 추천

## 2.3. pip를 이용하여 장고와 의존 패키지 설정하기
pip를 이용하여 requirements.txt 파일에 나열되어 있는 패키지를 가상환경안에 설치 가능하다.

* PYTHONPATH 설정
https://docs.djangoproject.com/en/1.11/ref/django-admin/#default-options

## 2.4. 버전 관리 시스템 이용하기
git 또는  머큐리얼

## 2.5. 동일한 환경 구성
베이그런트와 버츄얼박스
