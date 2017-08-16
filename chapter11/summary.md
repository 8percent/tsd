# 11. 장고 폼의 기초
- 어떠한 데이터든 간에 입력 데이터라고 한다면 장고 폼을 이용하여 유효성 검사를 해야한다는 것.

## 11.1 장고 폼을 이용하여 모든 입력 데이터에 대한 유효성 검사하기

- 나쁜예제 (다른 프로젝트로부터 CSV 파일을 받아 모델에 업데이트하는 장고 앱)
~~~python
import csv
import StringIO

from .models import Purchase

def add_csv_purchases(rows):

  rows = StringIO.StringIO(rows)
  records_added = 0
  
  # 한 줄당 하나의 dict를 생성. 단 첫 번째 줄은 키값으로 함
  for row in csv.DictReader(rows, delimiter=","):
    # 절대 따라하지 말 것. 유효성 검사 없이 바로 모델로 데이터 입력.
    Purchase.objects.create(**row)
    records_added += 1
    return records_added
~~~
Purchase 모델에서 문자열 값으로 저장되어 있는 seller가 실제로 존재하는 seller인지 체크하지 않음

장고의 폼을 이용하여 입력 데이터에 대해 유효성 검사를 하는 방식을 사용!

~~~python
import csv
import StringIO

from django import forms

from .models import Purchase, Seller

class PurchaseForm(forms.ModelForm):
  
  class Meta:
    model = Purchase
  
  def clean_seller(self):
    seller = self.cleaned_data["seller"]
    try:
      Seller.objects.get(name=seller)
    except Seller.DoesNotExist:
      msg = "{0} does not exist in purchase #{1}.".format(
        seller,
        self.cleaned_data["purchase_number"]
      )
      raise forms.ValidationError(msg)
    return seller
  
  def add_csv_purchases(rows):
    
    rows = StringIO.StringIO(rows)
    
    records_added = 0
    errors = []
    # 한 줄당 하나의 dict 생성. 단 첫 번째 줄은 키 값으로 함
    for row in csv.DictReader(rows, delimiter=","):
      
      # PurchaseForm에 원본 데이터 추가.
      form = PurchaseForm(row)
      # 원본 데이터가 유효한지 검사.
      if form.is_valid():
        # 원본 데이터가 유효하므로 해당 레코드 저장.
        form.save()
        records_added += 1
      else:
        errors.append(form.errors)
    
    return records_added, errors
~~~

> code 파라미터는 어떻게 할것인가?
> ValicationError에 code 파라미터를 전달해 줄 것을 추천
> 에러 원인을 명확히 할 수 있음.

## 11.2 HTML 폼에서 POST 메서드 이용하기
데이터를 변경하는 모든 HTML 폼은 POST 메서드를 이용하여 데이터를 전송하게 된다.

~~~html
<form action="{% url "flavor_add" %}" method="POST">
~~~

## 11.3 데이터를 변경하는 HTTP 폼은 언제나 CSRF 보안을 이용해야 한다.

### 11.3.1 AJAX를 통해 데이터 추가하기
절대 AJAX 뷰를 CSRF에 예외 처리하지 말기 -> 17.6 에서 자세히 설명

## 11.4 장고의 폼 인스턴스 속성을 추가하는 방법 이해하기

때때로 장고 폼의 clean(), clean_FOO(), save() 메서드에 추가로 폼 인스턴스 속성이 필요할 때가 있다.

폼 예시
~~~python
from django import forms

from .models import Taster

class TasterForm(forms.ModelForm):
  
  class Meta:
    model = Taster
  
  def __init__(self, *args, **kwargs):
    # user 속성 폼에 추가하기
    self.user = kwargs.pop('user') # 바로 kwargs['user'] 대신 pop을 사용하여 kwargs에서 
    super(TasterForm, self).__init__(*args, **kwargs)
~~~
super()를 호출하기 이전에 self.user가 추가

뷰 예시
~~~python
from django.views.generic import UpdateView

from braces.views import LoginRequiredMixin

from .forms import TasterForm
from .models import Taster

class TasterUpdateView(LoginRequiredMixin, UpdateView):
  model = Taster
  form_class = TasterForm
  success_url = "/someplace/"
  
  def get_form_kwargs(self):
    """키워드 인자들로 폼을 추가하는 메서드"""
    # 폼의 #kwargs 가져오기
    kwargs = super(TasterUpdateView, self).get_form_kwargs()
    # kwargs의 user_id 업데이트
    kwargs['user'] = self.request.user
    return kwargs
~~~


## 11.5 폼이 유효성을 검사하는 방법 알아두기

- form.is_valid() 가 호출될 때
1. 폼이 데이터를 받으면 form.is_valid()는 form.full_clean() 메서드를 호출한다.
2. form.full_clean()은 폼 필드들과 각각의 필드 유효성을 하나하나 검사하면서 다음과 같은 과정을 수행한다.
  
  a. 필드에 들어온 데이터에 대해 to_python()을 이요하여 파이썬 형식으로 변환하거나 변환할 때 문제가 생기면 ValidationError를 일으킨다.
  
  b. 커스텀 유효성 검사기(validator)를 포함한 각 필드에 특별한 유효성을 검사한다. 문제가 있을 때 ValidationError를 일으킨다.
  
  c. 폼에 clean_<field>() 메서드가 있으면 이를 실행한다.

~~~python
def _clean_fields(self):
    for name, field in self.fields.items():
        # value_from_datadict() gets the data from the data dictionaries.
        # Each widget type knows how to retrieve its own data, because some
        # widgets split data over several HTML fields.
        if field.disabled:
            value = self.get_initial_for_field(field, name)
        else:
            value = field.widget.value_from_datadict(self.data, self.files, self.add_prefix(name))
        try:
            if isinstance(field, FileField):
                initial = self.get_initial_for_field(field, name)
                value = field.clean(value, initial)
            else:
                value = field.clean(value)
            self.cleaned_data[name] = value
            if hasattr(self, 'clean_%s' % name):
                value = getattr(self, 'clean_%s' % name)()
                self.cleaned_data[name] = value
        except ValidationError as e:
            self.add_error(name, e)
~~~
3. form.full_clean()이 form.clean() 메서드를 실행한다.
~~~python
def clean(self):
    """
    Hook for doing any extra form-wide cleaning after Field.clean() has been
    called on every field. Any ValidationError raised by this method will
    not be associated with a particular field; it will have a special-case
    association with the field named '__all__'.
    """
    return self.cleaned_data
~~~
4. ModelForm 인스턴스의 경우 form.post_clean()이 다음 작업을 한다.
  
  a. form.is_valid()가 True나 False로 설정되어 있는 것과 관계없이 ModelForm의 데이터를 모델 인스턴스로 설정한다.
  
  b. 모델의 clean() 메서드를 호출한다. 참고로 ORM을 통해 모델 인스턴스를 저장할 때는 메델의 clean() 메서드가 호출되지는 않는다.
  
## 11.5.1 모델폼 데이터는 폼에 먼저 저장된 이후 모델 인스턴스에 저장된다.
폼 데이터가 폼 인스턴스로 변하는 것은 버그가 아닌 의도된 동작
ModelForm에서 폼 데이터는 두 가지 각기 다른 단계를 통해 저장된다.
1. 폼 데이터가 폼 인스턴스에 저장
2. 폼 데이터가 모델 인스턴스에 저장

위 특성을 통해 폼 입력 시도 실패에 대해 좀 더 자세한 사항이 필요할 때, 사용자가 입력한 폼의 데이터와 모델 인스턴스의 변화 둘 다 저장할 수 있다.

~~~python
# core/models.py
from django.db import models

class ModelFormFailureHistory(models.Model):
  form_data = models.TextField()
  model_data = models.TextField()
~~~

~~~python
# flavors/views.py
import json

from django.contrib import messages
from django.core import serializers
from core.models import ModelFormFailureHistory

class FlavorActionMixin(object):

  @property
  def success_msg(self):
    return NotImplemented
  
  def form_valid(self, form):
    messages.info(self.request, self.success_msg)
    return super(FlavorActionMixin, self).form_valid(form)
  
  def form_invalid(self, form):
    """실패 내역을 확인하기 위해 유효성 검사에 실패한 폼과 모델을 저장한다"""
    form_data = json.dumps(form.cleaned_data)
    
    model_data = serializers.serialize("json", [form.instance])[1:-1]
    
    ModelFormFailureHistory.objects.create(
      form_data=form_data,
      model_data=model_data
    )
    return super(FlavorActionMixin, self).form_invalid(form)
~~~

## 11.6 Form.add_error()를 이용하여 폼에 에러 추가하기
Form.add_error() 메서드를 사용하여 Form.claen()을 더 간소화할 수 있게 되었다.
~~~python
from django import forms

class IceCreamReviewForm(forms.Form):
  # tester 폼의 나머지 부분이 이곳에 위치
  ...
  def clean(self):
    cleaned_data = super(TasterForm, self).clean()
    flavor = cleaned_data.get("flavor")
    age = cleaned_data.get("age")
    
    if flavor == 'coffee' and age < 3:
      # 나중에 보여줄 에러들을 기록
      msg = u"Coffee Ice Cream is not for Babies."
      self.add_error('flavor', msg)
      self.add_error('age', msg)
    # 항상 처리된 데이터 전체를 반환한다
    return cleaned_data
~~~

## 요약
일단 폼을 작성하기 시작했다면 코드의 명료성과 테스트를 염두에 두자.
폼은 장고 프로젝트에서 주된 유효성 검사 도구이며, 불의의 데이터 충돌에 대한 중요한 방어 수단이기도 하다.
