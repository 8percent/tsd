# Chapter4. 장고 앱 디자인의 기본


### 기본 용어 체크
~~~
- 장고 프로젝트 : 장고 웹 프레임워크를 기반으로 한 웹 애플리케이션을 지칭
- 장고 앱 : 프로젝트의 한 기능을 표현하기 위해 디자인 된 작은 라이브러리
- INSTALLED_APPS : 프로젝트에서 이용하려고 INSTALLED_APP 세팅에 설정한 장고 앱들
- 서드 파티 장고 패키지 : 파이썬 패키지 도구들에 의해 패키지화된, 재사용 가능한 플러그인 형태로 이용 가능한 장고 앱
~~~
<br>

## 4.1 장고 앱 디자인의 황금률
- 각 앱이 그 앱의 주어진 임무에만 집중할 수 있어야 한다.(유닉스 철학)
<br>

### 4.1.1 실제 예를 통한 프로젝트 앱 정의
- 프로젝트(아이스크림 상점)
  - flavors 앱 : 상점의 모든 아이스크림 종류가 기록되고 그 목록을 웹사이트에 보여주는 앱
  - blog 앱 : 상점의 공식 블로그
  - events 앱 : 상점 행사 내용을 상점 웹사이트로 보여주는 앱
  - shop 앱 : 온라인 주문을 통해 아이스크림을 판매하는 앱
  - tickets 앱 : 무제한 아이스크림 행사에 이용될 티켓 판매를 관리하는 앱
  
 - 여기서 주목할 점은 서로 연관이 있어도( 예시. event-tickest, events-blog ) 모든 역할을 다하는 앱보다 세분화 된 몇가지 앱을 구성하는것이 더 바람직하다.
 
 <br>
 
 ## 4.2 장고 앱 이름 정하기
 - 가능한 한 단어 ( ex. flavors, animals, blogm polls, dreams ...)
 - 일반적으로 앱의 중심이 되는 모델 이름의 복수형태 (blog 같이 복수가 목적과 맞지않는다면 예외)
 - URL 주소가 어떻게 되는지 고려
 - PEP-8 규약에 맞게 import될 수 있는 이름 (소문자로 구성된 숫자, 대시, 마침표, 스페이스, 특수문자 포함하지 않는 짧은 단어)
 
 <br>
 
 ## 4.3 확신 없는 앱 확장 지양
 - 앱들을 될 수 있으면 작게 유지하려 하자.
 
 <br>
 
 ## 4.4 앱 안에 위치한 모듈들
 
 ### 4.4.1 공통 앱 모듈
 * 빗금(/)으로 끝나는 모듈은 하나 또는 여러 모듈을 내장할 수 있는 파이썬 패키지를 의미
 
 ~~~
 scoops/
    __init_.py
    admin.py
    forms.py
    management/
    migrations/
    models.py
    templatetags/
    tests/
    urls.py
    views.py
~~~
(공통 앱 모듈 들은일반적으로 이 규칙을 따른다)

<br>

### 4.4.2 비공통 앱 모듈
~~~
scoops/
    behaviors.py
    constants.py
    context_processors.py
    decorators.py
    db/
    exceptions
    fields.py
    factories.py
    helpers.py
    managers.py
    middleware.py
    signals.py
    utils.py
    viewmixins.py
~~~
(각 모듈에 대한 자세한 설명 책참고)

## 4.5 요약

- ***각 앱은 그 앱 자체가 지닌 한가지 역할에 초점을 맞추어야 한다.***
- ***이름은 단순하고 쉽게 기억되도록***
- ***앱의 기능이 너무 복잡하다면 여러 개의 작은 앱으로 나누자***
