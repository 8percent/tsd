# 3. 어떻게 장고 프로젝트를 구성할 것인가

## 3.1. 장고 1.8의 기본 프로젝트 구성
- startproject와 startapp 실행시 생성되는 기본 구성
~~~
django-admin.py startproject mysite
cd mysite
django-admin.py startapp my_app
~~~

~~~
mysite/
    manage.py
    my_app/
        __init.__py
        admin.py
        models.py
        tests.py
        views.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
~~~

## 3.2 우리가 선호하는 프로젝트 구성

~~~
<repository_root>/
    <django_project_root>/
        <configuration_root>/
~~~
### 3.2.1. 최상위 레벨: 저장소 루트
- <django_project_root> 이외에 README.rst, docs/, gitignore, requirements.txt 그외에 배포에 필요한 다른 파일등이 위치

### 3.2.2. 두번째 레벨: 프로젝트 루트
- 모든 파이썬 코드가 위치
- django-admin.py startproject 명령어를 저장소 루트에서 실행하여 생성된 장고 프로젝트가 프로젝트 루트가 됨

### 3.2.3. 세번째 레벨: 설정 루트
- setting 모듈과 urls.py 가 저장되는 장소
- 유효한 파이썬 패키지 형태(__init__.py 모듈이 존재해야 함)

## 3.3. 예제프로젝트 구성

### 3.3.1. <repository_root>내 구성
 * .gitignore: 깃이 처리하지 않을 파일과 디렉토리 
 * README.rst, docs/: 개발자를 위한 프로젝트 문서   
 * Makefile: 간단한 배포 작업 내용과 매크로를 포함한 파일, 복잡한 구성의 경우는 Fabric등의 도구 이용   
 * requirements.txt: 프로젝트에 이용되는 파이썬 패키지 목록 
 * <django_project_root>: 프로젝트 루트 

### 3.3.2. <django_project_root>내 구성
* config: 프로젝트 전바에 걸친 settings 파일, urls.py, vwgi.py 모듈이 자리
* manage.py: 
* media: 사용자가 올리는 사진 등의 미디어파일이 올라가는 장소, 큰 프로젝트의 경우 미디어 파일은 독립된 서버에 호스팅
* static: CSS, 자바스크립트, 이미지 등 사용자가 올리는 것 이외에 정적 파일들을 위치시키는 곳, 큰 프로젝트는 독립된 서버에 호스팅
* template: 시스템 통합 템플릿 파일 저장소
* <app_root>

## 3.4. virtualenv 설정
- ~/.virtaulenv/ 가 가상환경 경로의 기본값(virtualenvwrapper의 경우)

## 3.5. startproject 살펴보기
### 3.5.1. 쿠키커터로 프로젝트 구성 탬플릿 만들기
- 쿠키커터는 장고 프로젝트 구성을 위한 발전된 형태의 프로젝트 구성 탬플릿 도구
~~~
$ cookiecutter https://github.com/pydanny/cookiecutter-django
~~~
- 대안 탬플릿: django-kevin, 등등

## 3.6. 요약
- **어떤 구성을 택하더라도 반드시 명확하게 문서로 남겨야 함!**
