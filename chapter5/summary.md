# 5. settings와 requirements 파일
* 장고의 setting의 새로운 적용은 서버를 재시작해야만 적용기 가능
- 버전 컨트롤 시스템으로 모든 설정 파일을 관리해야 한다.
- 반복되는 설정들을 없애야 한다. 기본 세팅 파일로부터 상속
- 암호나 비밀 키 등은 안전하게 보관해야 한다. 이는 버전 컨트롤 시스템에서 제외.

## 5.1. 버전 관리되지 않은 로컬 세팅은 피하도록 한다.
- SECRET_KEY가 외부에 알려지면 심각한 보안 취약점을 야기

## 5.2. 여러 개의 settings 파일 이용하기
- settings/ 디렉토리를 사용하여 아래 여러개의 셋업파일을 구성하는 방법
~~~
settings/
    __init__.py
    base.py
    local.py
    staging.py
    test.py
    production.py
~~~
* base.py: 프로젝트의 모든 인스턴스에 적용되는 공용 세팅파일
* local.py: 로컬 환경에서 작업할 때 쓰이는 파일, 디버그모드, 로그레벨, django-debug-toolbar 같은 도구 활성화 등이 설정
* staging.py: 운영환경으로 코드가 완전히 이전되기 전, 관리자들과 고객들의 확인을 위한 시스템
* test.py: 테스트러너, 인메모리 데이터베이스 정의, 로그 세팅등을 포함한 테스트를 위한 세팅
* production.py: 운영서버에서 실제로 운영중인 세팅 파일

~~~
$ python manage.py shell --setting=<project_root>.settings.local
$ python manage.py runserver --setting=<project_root>.settings.local
~~~

- DJANGO_SETTINGS와 PYTHONPATH 를 이용하는 방법

### 5.2.1. 개발환경의 settings 파일 예제
유일하게 import * 구문을 이용해도 되는 파일,

### 5.2.2. 다중 개발 환경 세팅

## 5.3 코드에서 설정 분리하기 
