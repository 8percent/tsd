## 8장 함수 기반 뷰와 클래스 기반 뷰
### 함수 기반 뷰(FBV)와 클래스 기반 뷰(CBV)는 각각 언제 사용할까?
뷰를 구현할 때 마다 `함수 기반 뷰로 하는 게 나을지, 클래스 기반 뷰로 하는 게 더 나을지`를 고민하자

#### 클래스 기반 뷰를 사용할 때
- 대부분의 경우 선호
- 널리 사용되는 클래스 뷰들 중 하나가 이미 머리에 떠올랐다.
- 속성을 **오버라이딩**하는 것만으로 클래스 기반 뷰가 가능하다.
- 다른 뷰를 생성하기 위해 **서브클래스**를 만들어야 한다.

#### 함수 기반 뷰를 사용할 때
- 클래스 기반 뷰로 구현하기 위해 장고 소스 코드까지 들여다볼 정도로 **난해**하다.
- 클래스 기반 뷰로 처리할 경우 극단적으로 **복잡**해진다. 예를 들어 뷰가 한 개 이상의 폼을 처리할 경우

> **개인 프로젝트에 적용해보기** - 페이스북 소셜 로그인

#### FBV
뷰 자체에서 페이스북 사용자 정보를 받고, 로그인까지 실행한다.

```
def facebook_login(request):

    # 페이스북 로그인 버튼의 URL 을 통하여 facebook_login view 가 처음 호출될 때, 'code' request GET parameter 받으며, 'code' 가 없으면 오류 발생한다.
    code = request.GET.get('code')

    ##
    # 액세스 토큰 얻기
    ##

    # code 인자를 받아서 Access Token 교환을 URL 에 요청후, 해당 Access Token 을 받는다.
    def get_access_token(code):

        # Access Token 을 교환할 URL
        exchange_access_token_url = 'https://graph.facebook.com/v2.9/oauth/access_token'

        # 이전에 요청했던 URL 과 같은 값 생성(Access Token 요청시 필요)
        redirect_uri = '{}{}'.format(
            settings.SITE_URL,
            request.path,
        )
        
        # Access Token 요청시 필요한 파라미터
        exchange_access_token_url_params = {
            'client_id': settings.FACEBOOK_APP_ID,
            'redirect_uri': redirect_uri,
            'client_secret': settings.FACEBOOK_SECRET_CODE,
            'code': code,
        }
        print(exchange_access_token_url_params)

        # Access Token 을 요청한다.
        response = requests.get(
            exchange_access_token_url,
            params=exchange_access_token_url_params,
        )
        result = response.json()
        print(result)

        # 응답받은 결과값에 'access_token'이라는 key 가 존재하면,
        if 'access_token' in result:
            # access_token key 의 value 를 반환한다.
            return result['access_token']
        elif 'error' in result:
            raise Exception(result)
        else:
            raise Exception('Unknown error')

    ##
    # 액세스 토큰이 올바른지 검사
    ##
    def debug_token(token):
        app_access_token = '{}|{}'.format(
            settings.FACEBOOK_APP_ID,
            settings.FACEBOOK_SECRET_CODE,
        )

        debug_token_url = 'https://graph.facebook.com/debug_token'
        debug_token_url_params = {
            'input_token': token,
            'access_token': app_access_token
        }

        response = requests.get(debug_token_url, debug_token_url_params)
        result = response.json()

        if 'error' in result['data']:
            raise DebugTokenException(result)
        else:
            return result


    ##
    # 에러 메세지를 request 에 추가, 이전 페이지로 redirect
    ##
    def add_message_and_redirect_referer():
        error_message = 'Facebook login error'
        messages.error(request, error_message)

        # 이전 URL 로 리다이렉트
        return redirect(request.META['HTTP_REFERER'])

    ##
    # 발급받은 Access Token 을 이용하여 User 정보에 접근
    ##
    def get_user_info(user_id, token):
        url_user_info = 'https://graph.facebook.com/v2.9/{user_id}'.format(user_id=user_id)
        url_user_info_params = {
            'access_token': token,
            'fields': ','.join([
                'id',
                'name',
                'email',
            ])
        }
        response = requests.get(url_user_info, params=url_user_info_params)
        result = response.json()
        return result

    ##
    # 페이스북 로그인을 위해 정의한 함수 실행하기
    ##

    # code 가 없으면 에러 메세지를 request 에 추가하고 이전 페이지로 redirect
    if not code:
        return add_message_and_redirect_referer()

    try:
        access_token = get_access_token(code)
        debug_result = debug_token(access_token)
        user_info = get_user_info(user_id=debug_result['data']['user_id'], token=access_token)
        user = User.objects.get_or_create_facebook_user(user_info)

        django_login(request, user)
        return redirect('books:main')
    except GetAccessTokenException as e:
        print(e.code)
        print(e.message)
        return add_message_and_redirect_referer()
    except DebugTokenException as e:
        print(e.code)
        print(e.message)
        return add_message_and_redirect_referer()

```

#### CBV
`apiView`를 상속해서 token 값을 반환한다. 

```
class FaceBookLogin(APIView):
    def get(self, request):
        access_token = access_token_test(request)
        debug_result = debug_token(access_token)
        print(debug_result)

        def get_user_info(user_id, token):
            url_user_info = 'https://graph.facebook.com/v2.9/{user_id}'.format(user_id=user_id)
            url_user_info_params = {
                'access_token': token,
                'fields': ','.join([
                    'id',
                    'name',
                    'picture',
                ])
            }
            response = requests.get(url_user_info, params=url_user_info_params)
            result = response.json()
            print(result)
            return result

        user_info = get_user_info(user_id=debug_result['data']['user_id'], token=access_token)
        user = MyUser.objects.get_or_create_facebook_user(user_info)
        token, created = user.get_user_token(user.pk)
        return Response({'token': token.key})
```

#### 각 뷰를 사용한 후 느낀점
- 클래스 기반 뷰가 더 명료하고, 짧은 코드로 많은 기능을 수행할 수 있었다.
- 하지만 함수 기반 뷰를 사용해보고 동작하는 플로우를 알고 있었기 때문에 클래스 기반 뷰를 용이하게 활용할 수 있었고, 그 장점이 돋보일 수 있었던 것 같다.

---

### URLConf로부터 뷰 로직 분리하기
 **URL**은 최대한 유연하고 느슨하게 구성되어야 한다. 따라서 장고는 단순하고 명료하게 URL 라우트를 구성하는 방법을 제공한다.
 - 뷰 모듈은 뷰 로직을 포함해야한다.
 - URL 모듈을 URL 로직을 포함해야한다.

#### 느슨한 결합(loose coupling)을 해야하는 이유
- 뷰와 url, 모델 사이에 상호 단단하게 종속적인 결합을 이뤘을 경우, 
- 뷰에서 정의된 내용이 재사용되기 어렵다.
- url의 무한 확장성을 파괴시킨다. 따라서 CBV의 최대 장점인 클래스 상속이 불가능해진다.

#### 느슨한 결합 유지하기
[app-name/views.py]

```
[import...]
class TasteListView(ListView):
	model = Tasting
	
class TasteDetailView(DetailView):
	model = Tasting
	
[...]
```

[app-name/urls.py]

```
[import...]
urlpatterns = [
	url(
		regex=r'^$',
		view=views.TasteListVeiw.as_view(),
		name='list',
	),
	url(
		regex=r'^(?P<pk>\d+)/$',
		view=views.TasteDetailVeiw.as_view(),
		name='detail',
	),
	
	[...]
]
```
- 이로써 파일이 분리됐고, 오히려 코드는 더 늘어났다.

##### 이 방식이 과연 괜찮은가?
- 뷰들 사이에서 인자나 속성이 `중복 사용되지 않음`으로써 반복되는 작업을 줄일 수 있다.
- URLConf로부터 모델과 템플릿 이름을 전부 제거했다. `View는 View여야하고 URLConf는 URLConf`여야 하기 때문이다. 또한 하나 이상의 URLConf에서 뷰들이 호출될 수 있게 되었다.	
- 다른 클래스에서 우리의 뷰를 얼마든지 상속해서 쓸 수 있게되어 클래스 기반이라는 것에 대한 장점을 살리게 된다.
- URLConf는 한 번에 한 가지씩 업무를 명확하고 매끄럽게 처리해야 한다. 즉, URLConf는 URL 라우팅이라는 한 가지 명확한 작업만 처리해야하고 위 코드는 그것이 가능하다.

> 늘 URLConf로부터 로직을 분리 운영하도록 하자!

---

### URL namespace
- URL 이름공간은 앱 레벨 또는 인스턴스 레벨에서의 구분자를 제공한다.
- 'appname_list, appname_detail 등'과 같이 뷰 이름을 따라서 URL 이름을 짓지만, namescape를 이용한 경우 'list', 'detail'과 같은 명확한 이름을 짓게된다. 
- 또한 앱 이름을 입력하거나 부를 필요가 더 이상 없으니 시간이 절약되는 효과도 있다.

#### 검색, 업그레이드, 리팩터링 쉽게 하기
- 'appname_list, appname_detail' 같은 코드나 이름은 검색 결과가 나왔을 때 이것이 뷰 이름인지, URL 이름인지 알 수가 없다.
- 반면에 'appname:list, appname:detail'이라는 이름은 검색 결과를 좀 더 명확하게 해준다.
- 따라서 새로운 서드 파티 라이브러리와 상호 연동 시에 앱과 프로젝트를 좀 더 쉽게 업그레이드하고 리팩터링하게 만들어 준다.

---

### 장고의 뷰와 함수
- 기본적으로 장고의 뷰는 HTTP를 요청하는 객체를 받아서 HTTP를 응답하는 객체로 변경하는 함수다.
- 클래스 기반 뷰의 경우 함수 기반 뷰와 매우 다를 것으로 착각하기 쉽지만, URLConf에서 View.as_view()라는 클래스 메서드는 실제로 호출 가능한 뷰 인스턴스를 반환한다. 즉, 요청/응답 과정을 처리하는 콜백 함수 자체가 함수 기반 뷰와 동일하게 작동한다.

#### 뷰의 기본 형태
```
# 함수 기반 뷰
def function_based_view(request):
	return HttpResponse("FBV")

# 클래스 기반 뷰
class ClassBasedView(View):
	def get(self, request, *args, **kwargs):
		# 비지니스 로직
	return HttpResponse("CBV")
```
- 클래스 기반 뷰를 이용할 때 객체 상속을 이용함으로써 코드를 재사용하기 쉬워지고 디자인을 좀 더 유연하게 할 수 있다.
