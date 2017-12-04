보안문제에 대해서 라면 장고는 좀 괜찮아. 문서도 잘 쓰여져 있고, 핵심 개발자들도 신경 많이 쓰거든. 하지만 니가 못짜면 방법은 없어.
이번장은 장고앱의 보안을 위한 것들을 소개할께. 하지만 완벽한 보안은 없다는 것. 알지?

### 26.1 Reference Security Sections in Other Chapters

사실 지금까지 몇몇장에서 보안에 관련된 이야기를 해왔아. 잘 찾아봐.

### 26.2 Harden Your Servers

일단 서버부터 잘 운영해야해. 방화벽은 설정 뿐 아니라, SSH 포트도 변경하고, 사용하지 않는 서비스는 쓰지 않는것이 좋겠지?

### 26.3 Know Django’s Security Features

Django 1.11 은 다음과 같은 것들을 가지고 있어.
- Cross-site scripting (XSS) protection
- Cross-site request Forgery (CSRF) protection
- SQL injection protection
- Clickjacking protection
- Support for TLS/HTTPS/HSTS, including secure cookies
- Secure password storage, using the PBKDF2 algorithm with a SHA256 hash by default
- Automatic HTML escaping
- An expat parser hardened against XML bomb attacks
- Hardened JSON, YAML, and XML serialization/deserialization tools

뭐가 뭔지 잘 모르겠지? 니가 설정 하지 않아도 대충 돌아. 하지만 설정해야 할것도 있어. 우리는 이 중 몇가지에 집중 할께. 하지만 알지? 우리의 공식문서를 읽어줘. 

### 26.4 Turn Off DEBUG Mode in Production

운영환경에서 DEBUG 키지마. DEBUG 모드에서는 해커가 필요한 여러가지들을 얻게 돼. DEBUG 모드를 끌 때에는 ALLOWED_HOSTS 설정을 해주어야 해. 그렇지 않으면 SuspiciousOperation 에러를 만나게 될꺼야.
- https://docs.djangoproject.com/en/1.11/ref/settings/#allowed-hosts
- https://docs.djangoproject.com/en/1.11/ref/exceptions/#suspiciousoperation

### 26.5 Keep Your Secret Keys Secret

SECRET_KEY 털리면 끝장난다. Git 에도 넣지마. (앞에서 배웠지?)

### 26.6 HTTPS Everywhere

HTTP 쓰면 너네 서버랑 클라이언트 사이에 오가는 데이터를 해커들이 다 까본다?
혹은 중간에 바꿔치기 해서 사람들이 이상한걸 보게 될지도 모른다?
HTTPS 싸이트에서는 HTTPS 만 접속해야해. 아니면 경고가 뜬다. 
HTTP로 접근하면 HTTPS 로 연결해줘. 장고 보다는 웹서버, 웹서버 보다는 LB에서 해주는것이 더 좋겠지?
스스로 싸인한 인증서를 쓰지 말고 공인된 곳에 만들어 준 인증서를 쓰렴. (공짜도 있어)

### 26.6.1 Use Secure Cookies

SESSION_COOKIE_SECURE, CSRF_COOKIE_SECURE 를 쓰면 http 일때에는 cookie 를 보내지 않아. 
Https 를 쓰고 있다고 해도 최초에 http로 접속을 하면 이 값을 보내게 되겠지?

### 26.6.2 Use HTTP Strict Transport Security

웹서버가 말하기를 “야. 이 사이트는 http로 접근도 하지마” 
chrome://net-internals/#hsts

### 26.6.3 HTTPS Configuration Tools

https://www.ssllabs.com/ssltest/analyze.html?d=8percent.kr&latest 에서 테스트 해보고 높은 점수 받아보렴

### 26.7 Use Allowed Hosts Validation

ALLOWED_HOSTS 를 쓰자. 와일드 카드는 쓰지마. 

### 26.8 Always Use CSRF Protection With HTTP Forms That Modify Data

데이터 수정할때는 CSRF Protection!

### 26.9 Prevent Against Cross-Site Scripting(XSS) Attacks

사용자가 입력한 스크립트를 실행시키지 않도록 >, <, & 등을 escaping 해주고 있단다. 

### 26.9.1 Use format_html Over mark_safe

format_html 를 쓰면 parameter 를 잘 escaping 해줘. 

### 26.9.2 Don’t Allow Users to Set Individual HTML Tag Attributes

사용자가 HTML Attributes 를 설정할 수 있게 해주면 뭘 넣을지 모른다?

### 26.9.3 Use JSON Encoding for Data Consumed by JavaScript

JSON Encoding 을 쓰는게 클라이언트에서 사용하기 편한것만 아니라 더 안전해.
Pickling 을 바로 한다고 생각해 보면 unserialize 를 할때 어떤 코드가 실행될지 몰라. 

### 26.9.4 Beware Unusual JavaScript

이상한 자바 스크립트는 조심하자. 이상한 문법으로도 동작하니까. 

### 26.9.5 Add Content Security Policy Headers

https://developers.google.com/web/fundamentals/security/csp/?hl=ko 를 봐봐. Header 에 다양한 보안정책을 지정할 수 있어. 

### 26.9.6 Additional Reading 

읽어보고 공부하자. 

### 26.10 Defend Against Python Code Injection Attacks

#### 26.10.1 Python Built-Ins That Execute Code 

조심해. eval, exec, execfile. 사용자가 여기에 어떤 값을 넣게 하지마.

#### 26.10.2 Python Standrad Library Modules That Can Execute Code

사용자가 넘겨준것을 unpicking 하다가 코드가 실행된다.

#### 26.10.3 Third-Party Libraries That Can Execute Code

외부 라이브러리도 조심!

#### 26.10.4 Be Careful With Cookie-Based Sessions

Cookie에 session 을 직접 두는것에 대해서는 더 조심하자. 보통 hash 값만 주고 session 을 DB나 Cache 에 보관하는게 더 좋아 

### 26.11 Validate All Incoming Data With Django Forms

Form 으로 들어오는 데이터들도 다 꼼꼼하게 체크해 주렴 

### 26.12 Disable the Autocomplete on Payment Fields

카드 입력등을 위한 필드에는 자동완성을 쓰지 말아줘. 패스워드도 고민해봐 주겠니? 공항에서 접속한다고 생각해봐.

### 26.13 Handle User-Uploaded Files Carefully

사용자 파일들은 서버에 두지 않는것이 좋겠다. 그냥 CDN 을 통해서 처리하렴 

#### 26.13.1 When a CDN IS Not an Option

CGI, PHP 파일을 업로드하고 http 로 그파일을 접근한다고 생각해봐. 어유 무서워.

#### 26.13.2 Django and User-Uploaded Files

최선을 다해서 니가 원하는 파일만 받으려고 노력해야해. (예를 들어 이미지 파일임을 열심히 확인해)

### 26.14 Don’t Use ModelForms.Meta.exclude 

Model 변경을 해놓고 form 변경을 빼먹어서 의도하지 않은 field 에대한 쓰기가 되니 쓰지마!

#### 26.14.1 Mass Assignment Vulnerabilities

명백하게 수정될 수 있는 필드만 정의하는게 맞아. 

### 26.15 Don’t Use ModelForms.Meta.fields = “__all__”

Excludes 도 쓰지말라는 마당에 all 이 왠말이니?

### 26.16 Beware of SQL Injection Attacks 

Django ORM 에서는 적당히 esacaping 을 해준단다. 하지만 니가 직접 .raw() 를 쓴다면 방법은 없지.

### 26.17 Never Store Credit Card Data

민감정보는 안들고 있는게 좋아. 

### 26.18  Monitor Your Sites

모니터링 도구를 설치해 놓고 이상동작들을 확인하는게 좋아 

### 26.19 Keep Your Dependencies Up-to-Date

버전은 제때 제때 올리자. 특히 보안 배치가 있을 때는 말야.

### 26.20 Prevent Clickjacking

Iframe 에 상품 구매 버튼을 놓고 그 위에 페북을 올려놓고 페북 좋아요를 누르면 상품 구매가 일어나게 할 수 있어. 
이걸 막으려면 X-Frame-Option을 사용하자 

### 26.21 Guard Against XML Bombing With defusexml

XML 공격에 취약한 녀석들이 있으니 defusexml 을 쓰자 

### 26.22 Explore Two-Factor Authentication

2중 인증을 하자. 로그인할 때 문자 받는거 알지?

### 26.23 Embrace SecurityMiddleware

Django.moddleware.security.SecurityMiddleware 를 써

### 26.24 Force the Use of Stroing Passwords

사람들 비번 아무거나 못넣게 해. 1234 넣는다.

### 26.25 Give Your Site a Security Checkup

체크해 주는 도구들이 있으니까 사용해 보자. 
https://www.ponycheckup.com/result/?url=8percent.kr

### 26.26 Put Up a Vulnerability Reporting Page

신고 페이지를 만들어 보렴. 혹시 알아 알려줄지도 몰라

### 26.27 Never Display Sequential Primary Keys

하나식 증가하는 값은 사람들이 손쉽게 들어와서 살펴볼꺼야. 

#### 26.27.1 Lookup By Slug

Slug 를 사용하자. 

#### 26.27.2 UUIDs

models.UUIDField 도 있어. 

### 26.28 Reference Our Security Settings Appendix

형이 한번 해봤으니. 참고 하렴 

### 26.29 Review the List of Security Packages

뒤에 보면 보안 패키지들 리스트가 있어. 역시나 참고하렴

### 26.30 Keep Up-to-Date on General Security Practices

항상 보안에 대해서 관심을 가져야 해
