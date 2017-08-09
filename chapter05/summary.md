# 5. settings와 requirements 파일
* 세팅은 서버가 시작될 때 적용. 세팅값의 새로운 적용은 서버를 재시작해야 적용 가능하므로 서비스 운영중에 임의로 변경할 수 없다.
* 이 책에서 제안하는 최선의 장고 설정 방법
   
    - 버전 컨트롤 시스템으로 모든 설정 파일을 관리해야 한다. 세팅 변화에 대한 기록이 반드시 문서화되어야 한다.
    - 반복되는 설정들을 없애야 한다. 기본 세팅 파일로부터 상속을 통해 이용해야 한다.
    - 암호나 비밀 키 등은 안전하게 보관해야 한다. 보안 관련 사항은 버전 컨트롤 시스템에서 제외한다.

## 5.1. 버전 관리되지 않은 로컬 세팅은 피하도록 한다.
* SECRET_KEY, 아마존 API 키, 스트라이프(Stripe) API 키, 다른 비밀번호 형태의 여러 설정 변수도 보안을 위해 저장소에서 빼야 한다.
* local_settings.py를 버전 컨트롤 시스템에서 제외했을 때 발생하는 문제

  1. 모든 머신에 버전 컨트롤에 기록되지 않는 코드가 존재하게 된다.
  2. 운영 환경의 버그가 로컬에서 구현되지 않는다.
  3. 같은일을 반복해야 한다.(local_settings.py 파일에 변경사항이 있을 경우, 여러 팀원이 복사 붙여넣기를 해서 적용해야 한다.)
* 위와 같은 문제점을 예방하기 위해서 개발 환경, 스테이징 환경, 테스트 환경, 운영 환경 설정을 공통되는 객체로부터 상속받아 구성된 서로 다른 세팅 파일로 나누어 버전 컨트롤 시스템에서 관리하는 방법이 있다. 

## 5.2. 여러 개의 settings 파일 이용하기
* settings/ 디렉토리를 사용하여 아래 여러개의 셋업파일을 구성하는 방법
~~~
settings/
    __init__.py
    base.py
    local.py
    staging.py
    test.py
    production.py
~~~
* base.py: 프로젝트의 모든 인스턴스에 적용되는 공용 세팅 파일
* local.py: 로컬 환경에서 작업할 때 쓰이는 파일, 디버그 모드, 로그 레벨, django-debug-toolbar 같은 도구 활성화 등이 설정되어 있는 개발 전용 로컬 파일
* staging.py: 운영 환경으로 코드가 완전히 이전되기 전, 관리자들과 고객들의 확인을 위한 시스템
* test.py: 테스트 러너, 인메모리 데이터베이스 정의, 로그 세팅등을 포함한 테스트를 위한 세팅
* production.py: 운영서버에서 실제로 운영중인 세팅 파일

~~~
$ python manage.py shell --setting=<project_root>.settings.local
$ python manage.py runserver --setting=<project_root>.settings.local
~~~

- DJANGO_SETTINGS_MODULE과 PYTHONPATH 를 이용하는 방법?

### 5.2.1. 개발환경의 settings 파일 예제
* 유일하게 import * 구문을 이용해도 되는 파일

### 5.2.2. 다중 개발 환경 세팅
* 개발자마다 자기만의 환경이 필요한 경우, 개개인의 요구에 맞게 수정된 여러개의 개발 세팅 파일을 생성하는것이 좋다. (예를 들면, dev_mj.py와 같은 이름을 가진 파일을 생성하여 이용하는 것)
* 짧은 캐시 타임아웃을 설정하는 이유? 
  - CACHE_TIMEOUT = 30

## 5.3 코드에서 설정 분리하기 
* 환경 변수(environment variables)를 비밀 키를 위해 이용하는것을 환경 변수 패턴이라 하는데, 장고(와 파이썬)는 운영 체제의 환경 변수를 손쉽게 설정할 수 있는 기능을 제공한다.

  1. 세팅 파일을 버전 컨트롤 시스템에 추가할 수 있다.
  2. 개발자 모두 버전 컨트롤로 관리되는 local.py를 사용할 수 있다.
  3. 파이썬 코드 수정없이 프로젝트 코드를 쉽게 배치할 수 있다?
  4. 대부분의 PaaS(Platform as a Service)가 설정을 환경 변수를 통해 이용하기를 추천하고, 이러한 기능들을 내장하고 있다.

### 5.3.1 환경 변수에 비밀 키 등을 넣어 두기 전에 유의할 점
* 저장되는 비밀 정보를 관리할 방법
* 서버에서 배시(bash)가 환경 변수와 작용하는 방식에 대한 이해 또는 PaaS 이용 여부

### 5.3.2 로컬 환경에서 환경 변수 세팅하기
* 맥과 리눅스의 경우, virtualenv의 /bin/activate 스크립트의 맨 마지막 부분에 아래 구문을 넣어주면 된다.
~~~
$ export SOME_SECRET_KEY=1c3-cr3am-15-yummy
$ export MJ_FREEZER_KEY=dkfs-3-555-1c3-cr3am-15-yummy
~~~

### 5.3.3 운영 환경에서 환경 변수를 세팅하는 방법
* 장고 프로젝트가 PaaS를 통해 배포된다면 해당 배포 도구의 문서를 확인해야 한다.

### 5.3.4 비밀 키가 존재하지 않을 때 예외 처리하기
* 환경 변수가 존재하지 않을 때 원인을 더 쉽게 알려주는 코드

```python
# settings/base.py
import os

# 일반적으로 장고로부터 직접 무언가를 설정 파일로 임포트해 올 일은
# 없을 것이며 또한 해서도 안 된다. 단 ImproperyConfigured는 예외다.
from django.core.exceptions import ImproperlyConfigured

def get_env_variable(var_name):
    """환경 변수를 가져오거나 예외를 반환한다."""
    try:
        return os.environ[var_name]
    except KeyError:
        error_msg = "Set the {} environment variable".format(var_name)
        raise ImproperlyConfigured(erro.r_msg)
````

## 5.4 환경 변수를 이용할 수 없을 때
* 아파치를 웹(HTTP) 서버로 이용하는 경우, Nginx도 특정 환경에 한해 작동되지 않을 경우
* 비밀 파일 패턴(secrets file pattern) 사용 권장.
    
    1. JSON, Config, YAML 또는 XML 중 한 가지 포맷으로 비밀 파일 생성
    2. 비밀 파일을 관리하기 위한 비밀 파일 로더를 추가한다.
    3. 비밀 파일의 이름을 .gitignore에 추가한다.

### 5.4.1 JSON 파일 이용하기
### 5.4.2 Config, YAML, XML 파일 이용하기
* 파이썬뿐 아니라 다른 언어에서도 다양하게 이용할 수 있는 JSON 파일을 이용하는 방법을 권장

## 5.5 여러 개의 requirements 파일 이용하기
* 각 서버에서 해당 환경에 필요한 컴포넌트만 설치
~~~
requirements/
    base.txt
    local.txt
    staging.txt
    production.txt
~~~

### 5.5.1 여러 개의 requirements 파일로부터 설치하기
~~~
$ pip install -r requirements/local.txt
$ pip install -r requirements/production.txt
~~~

### 5.5.2 여러 개의 requirements 파일을 PaaS 환경에서 이용하기
* 30장에서 자세히 다룰 예정

## 5.6 settings에서 파일 경로 처리하기
* 경로는 절대 하드 코딩하지 않는다!
* Unipath 패키지 또는 os.path 파이썬 기본 라이브러리로 BASE_DIR 경로를 세팅하는 것을 추천



