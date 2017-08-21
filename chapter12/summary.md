# 13. 템플릿의 모범적인 이용
- 초기 장고 디자인의 방향은 템플릿 언어의 기능에 많은 제약을 두자는 것

## 13.1 대부분의 템플릿은 templates/에 넣어 두자
- 메인 템플릿 디렉토리에 위치시키자(어느 방법이든 상관없지만 관리가 어려울수있다)
- 올바른 예제
~~~python
templates/
  base.html
  ... (여타 사이트 전반에 걸친 템플릿은 여기에 둔다)
  freezers/
    ... ('freezers' 앱 템플릿은 여기에 둔다)
~~~

- 나쁜 예제
~~~python
freezers/
  templates/
    freezers/
      ... ('freezers' 앱 템플릿은 여기에 둔다)
templates/
  base.html
   ... (여타 사이트 전반에 걸친 템플릿은 여기에 둔다)
~~~

- 예외 : 플러그인 패키지로 설치된 장고 앱을 작업할 때는 메인 'templates/' 디렉토리 안의 템플릿으로 오버라이딩하여 사용하자.
## 13.2 템플릿 아키텍처 패턴
일반적으로 2중, 3중 구조로 이루어진 템플릿 형태가 이상적

### 13.2.1 2중 템플릿 구조의 예
- 모든 템플릿은 하나의 base.html 파일로 부터 상속받음

### 13.2.2 3중 템플릿 구조의 예
- 각 앱은 base_<app_name>.html 템플릿을 가지고, 앱 레벨 베이스 템플릿은 모두 base.html이라는 공통 템플릿을 공유
~~~python
templates/
  base.html
  dashboard.html # base.html 확장
  profiles/
    base_profiles.html # base_profiles.html 확장
    profile_detail.html # base_profiles.html 확장
    profile_form.html # base_profiles.html 확장
~~~
섹션별로 레이아웃이 다른 경우 최적화 된 구성

### 13.2.3 수평구조가 중첩된 구조보다 좋다
- 블록들을 단순한 상속 관계로 구성해야 템플릿 작업하기 쉽고 유지 관리하기 편해진다.
- 파이썬의 도
~~~
product$ python -c 'import this'
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
~~~

## 13.3 템플릿에서 프로세싱 제한하기
- 템플릿에서 처리하는 프로세싱은 적을수록 좋다.
- 템플릿 레이어에서의 쿼리 수행과 이터레이션은 특히 문제가 된다.
  - 쿼리세트가 얼마나 큰가?
  - 얼마나 큰 객체가 반환되는가? 모든 필드가 정말로 다 템플릿에 필요한가?
  - 각 이터레이션 루프 때마다 얼마나 많은 프로세싱이 벌어지느가?
- 캐시도 괜찮지만 그 전에 원인을 먼저 분석해 고쳐보자.

- 지금부터 나오는 피해야 할 몇 가지 주의사항을 살펴보자.

### 13.3.1 주의 사항 1: 템플릿상에서 처리하는 aggregation 메소드
- 장고 템플릿에서 비지니스 로직을 처리하는 방법들은 피하라.

### 13.3.2 주의 사항 2: 템플릿상에서 조건문으로 하는 필터링
- 템플릿상에서 거대한 루프문과 if문을 돌리지 말자.
- PostgreSQL, MySQL은 데이터 필터링에 최적화된 기능을 가진다. 장고 ORM은 이런 기능을 이용하는데 유용하게 쓰일 수 있다.
- view에서 미리 처리하고, 템플릿은 결과를 호출하자

### 13.3.3 주의 사항 3: 템플릿상에서 복잡하게 얽힌 쿼리들
- select_related() 메서드를 이용하자.

~~~python
#3번의 쿼리
pet = Pet.objects.get(id=1)
person = pet.person
country = person.country

#1번의 쿼리
pet = Pet.objects.select_related('person__country').get(id=1)
person = pet.person
country = person.country
~~~

### 13.3.4 주의 사항 4: 템플릿에서 생기는 CPU 부하
- 간단한 템플릿 코드라도 상당한 프로세싱을 필요로 하는 객체가 호출되어 부하가 상당한 부하를 일으킬 수 있다.
- 많은 양의 이미지나 데이터를 처리하는 프로제긑는 뷰나 모델, 헬퍼 메서드, 또는 셀러리 등을 이용한 비동기 메시지 큐 시스템으로 처리해야된다.

### 13.3.5 주의 사항 5: 템플릿에서 숨겨진 REST API 호출
- REST : Representational State Tranfer : 장비간의 통신을 위해 HTTP를 사용하는 것이 목적
  - URI라는 Resource로 HTTP Method를 사용함
- 호출이 많으면 로딩 시간이 길어진다

### 13.3.6 HTML 코드를 정돈하는 데 너무 신경 쓰지 말라
- 최적화를 위해 난해하게 제작된 코드보다는 가독성 높은 코드를 선호
- 일일이 손으로 하는 것 보다 나은 기능을 제공하는 압축, 소형화 도구가 존재(24장 참고)

## 13.5 템플릿의 상속
- base.html
  - title : Rwo Scoops of Django
  - stylesheets : 사이트 전반에 이용되는 project.css 파일에 대한 링크
  - content 블록 : <h1>Two Scoops</h1>
  - 템플릿 태그들이 이용되어 있다.
여기서 커스텀 타이틀, 기본 스타일시트아 추가 스타일시트, 기본 헤더, 서브헤더, 본문, 자식블록의 이용, 
{{block.super}} 템플릿 변수의 이용을 포함한 about.html을 base.html을 상속받아 만들어 보자

~~~python
{% extends "base.html" %}
{% load staticfiles %}
{% block title %}About Audrey and Daniel{% endblock title %}
{% block stylesheets %}
  {{block.super}}
  <link rel="stylesheet" type="text/css"
    href="{% static "css/about.css" %}">
  {% endblock stylesheets %}
{% block content %}
  {{block.super}}
  <h2>About Audrey and Daniel</h2>
  <p>They enjoy eating ice cream</p>
{% endblock content %}
~~~

이 템플릿을 렌더링 하면 다음의 HTML이 생성된다

~~~python
<html>
<head>
  <title>
    About Audrey and Daniel
  </title>
    <link rel="stylesheet" type="text/css"
      href="/static/css/project.css">
    <link rel="stylesheet" type="text/css"
      href="/static/css/about.css">
</head>
<body>
  <div calss="content">
    <h1>Two Scoops</h1>
    <h2>About Audrey and Daniel</h2>
    <p>They enjoy eating ice cream</p>
  </div>
</body>
</html>
~~~

- 템플릿 태그
  - {% load %}: 정적 파일의 내장 템플릿 태그 라이브러리를 로드
  - {% block %}: base.html이 부모가 되는 템플릿이기 때문에 해당 블록을 자식 템플릿에서 이용할 수 있게한다.
  - {% static %}: 정적 미디어 서버에 이용될 정적 미디어 인자
  - {% extend %}: 장고에게 about.html이 base.html을 상속, 확장한 것임을 알려준다
  - {% block %}: about.html은 상속한 자식 템플릿이기 때문에 block은 base.html의 내용으로 오버라이드 된다.
  - {{block.super}}: 자식 템플릿 블록에 위치하여 부모의 내용이 블록 안에 그대로 존재하게 해 준다.


## 13.6 강력한 기능의 block.super
- 이용하지 않는다면 같은 이름을 가진 베이스 파일을 새로 생성해야한다.
- 이용 한다면 html파일을 확장하여 이용하면 {{ block.super}}를 이용해 템플릿들을 관리할 수 있다.

## 13.7 그 외 유용한 사항들 
- 템플릿 개발 시 기억해두면 유용하다

### 13.7.1 파이썬 코드와 스타일을 긴밀하게 연결하지 않도록 한다
- CSS와 자바스크립트를 이용하여 모든 템플릿 렌더링 스타일을 구현하는 것을 목표로 하자
### 13.7.2 일반적인 관례
- 이름과 스타일 관례들을 잘 알아두자
### 13.7.3 템플릿의 위치
- 일반적으로 장고프로젝트 루트에서 앱과 같은 레벨에 존재해야 한다.
- 예외는 서드 파티 패키지에 앱을 번들하는 경우
### 13.7.4 콘텍스트 객체에 모델의 이름을 붙여 이용하기
- implicit한 {{object}}, {{object_list}} 대신에 explicit한 {topping_list}}, {{toppin}}을 이용할 수 있다.
### 13.7.5 하드 코딩 된 경로 대신 URL 이름 이용하기
~~~python
#나쁜예
<a href="/flavors/">

#좋은 예(URLConf 파일 안의 이름을 참조하는 %url 태그이용)
<a href="{% url 'flavors_list' %}">
~~~

### 13.7.6 복잡한 템플릿 디버깅하기
- TEMPLATES 세팅의 OPTTIONS에서 string_if_invalid 옵션을 설정함으로써 좀 더 자세한 에러 메세지를 받아 볼 수 있따.

## 13.8 에러 페이지 템플릿
- 예상치 못한 에러들을 처리하여 사용자들에게 보여준다
- 최소 404.html, 500.html 템플릿은 구현해 놓아야 한다.

## 13.9 미니멀리스트 접근법을 따르도록 한다
- 템플릿이 아닌 장고 코드에 더 많은 비즈니스 로직을 구현하도록 하자

## 13.10 요약
- {{block.super}}의 이용을 비롯한 템플릿의 상속
- 관리가 편리하며 가독성이 뛰어난 템플릿 제작하기
- 템플릿 성능 최적화를 위한 방법
- 템플릿 프로세싱의 한계에 따른 이슈들
- 에러 페이지 템플릿
- 템플릿에 대한 여러 가지 유용한 팁
