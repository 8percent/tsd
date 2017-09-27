## admin.site.register() 코드 따라가보기


```python
# admin.py

admin.site.register(IceCream)

# or

class IceCreamAdmin(admin.ModelAdmin):
	...
admin.site.register(IceCream, IceCreamAdmin)

# or

@admin.register(IceCream)
class IceCreamAdmin(admin.ModelAdmin):
	...
```

```python
# __init__.py

...
def autodiscover():
    autodiscover_modules('admin', register_to=site)


default_app_config = 'django.contrib.admin.apps.AdminConfig'
```

```python
# module_loading.py

def autodiscover_modules(*args, **kwargs):

from django.apps import apps

    register_to = kwargs.get('register_to')
    for app_config in apps.get_app_configs():
        for module_to_search in args:
            try:
                if register_to:
                    before_import_registry = copy.copy(register_to._registry)

                import_module('%s.%s' % (app_config.name, module_to_search))
            except Exception:
                if register_to:
                    register_to._registry = before_import_registry
                    raise
```

```python
# site.py

class AdminSite:
    ...
	
    def __init__(self, name='admin'):
        self._registry = {}
        self.name = name
        self._actions = {'delete_selected': actions.delete_selected}
        self._global_actions = self._actions.copy()
        all_sites.add(self)	

    ...
	
    def register(self, model_or_iterable, admin_class=None, **options):
        
        if not admin_class:
            admin_class = ModelAdmin

        if isinstance(model_or_iterable, ModelBase):
            model_or_iterable = [model_or_iterable]
            
        for model in model_or_iterable:
            if model._meta.abstract:
                raise ImproperlyConfigured(
                    'The model %s is abstract, so it cannot be registered with admin.' % model.__name__
                )

            if model in self._registry:
                raise AlreadyRegistered('The model %s is already registered' % model.__name__)

            if not model._meta.swapped:
                if options:
                    options['__module__'] = __name__
                    admin_class = type("%sAdmin" % model.__name__, (admin_class,), options)

                self._registry[model] = admin_class(model, self)
                
site = AdminSite()  
```

- `_registry`에는 model class를 admin_class의 인스턴스로 변환하여 저장하고, `autodiscover_modules` 메서드를 통해 모듈 import에 사용된다.   

- Abstract Model에 대해서는 admin을 정의할 수 없음  

> swapped?  


## ModelAdmin
BaseModelAdmin을 상속받는다.  
Admin register 시 admin class를 할당하지 않으면, 기본으로 ModelAdmin을 사용하게 되어있음.

- `list_display` : 리스트 페이지에서 보여줄 필드 
	
	```python
	class IceCreamAdmin(admin.ModelAdmin):
	
        list_display = ('id', 'name', 'without_vat')
		
        def without_vat(self, obj):
            price = obj.price * 0.9
            return round(price)
	```
	> 가격 계산은 임의로 넣은 것이니 너무 괘념치 마소서..

- `list_display_links` : 상세페이지 이동 링크를 연결할 필드를 설정. 기본은 첫번째 필드에 적용.

    ```python
    # detail view 접근 막기
    
    class IceCreamAdmin(admin.ModelAdmin):
        list_display_links = (None, )
    ```

...

> 필요에 따라 찾아보는 것이 좋을 것 같아요 ^.^ 너모 많네요  
> 대체로 필요로하는 대부분의 기능들을 지원하는 것이 장점!

## action 정의하기
기본적으로 delete action이 설정되어있고 이는 global action에 정의되어있다.  
admin class에 필요한 action을 메서드로 정의하고, actions 리스트에 담는다.

```python 
class IceCreamAdmin(admin.ModelAdmin):

    def close_item(self, request, queryset):
        for q in queryset:
            q.status = 'close'
            q.save()
        self.message_user(request, "%s 개의 아이스크림이 판매종료 처리되었습니다." % queryset.count())
	
    close_item.short_description = '판매종료 상품으로 전환'
			
    actions = [close_item, ]
```

## 특정 조건의 데이터만 보고싶다
queryset을 변경

```python 
class IceCreamAdmin(admin.ModelAdmin):

    def queryset(self):
        qs = IceCream.objects.filter(status='open')
        return qs
```

## 상위 모델과 같은 페이지에서 편집하고 싶다

```python
# models.py

class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    title = models.CharField(max_length=100)
```

Book 모델의 Author필드가 Author 모델을 FK로 가지고 있는 경우 
Author 어드민 페이지에서 Book 모델을 편집할 수 있다.
	
```python
class BookInline(admin.TabularInline):
    model = Book
	
class AuthorAdmin(admin.ModelAdmin):
    inlines = [BookInline,]
```
	
InlineModelAdmin을 상속받는 StackedInline 클래스와 TabularInline 클래스가 존재.
두 클래스의 차이는 렌더링되는 템플릿의 차이이다.


## 폼 변경도 하고싶다

```python
class IceCreamForm(forms.ModelForm):
    class Meta:
        model = IceCream
    # Form 정의 

class IceCreamAdmin(admin.ModelAdmin):
    form = IceCreamForm
```


- 추가할 때와 변경할 때의 폼이 달라야 한다면?

    ```python
    class IceCreamForm(forms.ModelForm):
        class Meta:
            model = IceCream

        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)
			
            if 'instance' in kwargs:
                self.fields['price'].widget.attrs['readonly'] = True
	```

- 특정 필드에 초기 값을 설정해야한다면?

	```python
	def __init__(self, *args, **kwargs):
        self.fields['size'].initial = 'single'
	```

- Form field의 queryset을 설정해보자.  
	아이스크림의 스페셜 토핑을 선택하기 위해서, 토핑 모델의 모든 데이터를 선택지에 올릴 필요는 없다. 
	토핑 객체 중에 special 카테고리가 할당된 것들만 선택 옵션으로 보여줄 수 있다.
	
	```python
	class IceCreamForm(forms.ModelForm):

        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)
            self.fields['special_topping'] = forms.ModelChoiceField(queryset=Topping.objects.filter(
            category='special'), label='스페셜 토핑', required=False)
	
        class Meta:
            model = IceCream
            fields = ('id', 'name', 'special_topping')
	```

## 추가/수정/삭제 Permission 설정

```python
class IceCreamAdmin(admin.ModelAdmin):
    ...
    def has_add_permission(self, request):
        return False
	    
    def has_change_permission(self, request):
        return False
	
    def has_delete_permission(self, request):
        return False
```

**add permission = False**  
데이터를 추가할 수 없음. 추가 버튼이 사라짐.
	
**change permission = False**  
상세페이지에서 데이터를 수정할 수 없음.  
상세페이지는 입력을 받는 form 이 아닌, 읽기 전용으로 해당 데이터를 보여줌.
	
**delete permission = False**  
delete action을 사용할 수 없음  
	
action 재정의로 객체 삭제를 커스텀할 수 있다.  
특정 객체만 바로 삭제하는 경우에 문제가 되는 케이스에 이용할 수 있음. (재정의한 action에서 연관된 데이터를 처리할 수 있다.)

## 하나의 모델에 대해 두 개의 어드민 페이지를 만들 수 있을까?
model은 admin register는 한번만 가능.
중복의 경우 AlreadyRegistered 에러가 발생.


> [Documentation - admin](https://docs.djangoproject.com/en/1.11/ref/contrib/admin/) 참고 




