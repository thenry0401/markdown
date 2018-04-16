# django - Models

- `model` 은 데이터에 대한 단일 정보 소스이다
- `django.db.models.Model` 로 import한다

> example

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```



## models 사용하기

`manage.py startapp` 명령어를 입력하면 `settings.py`  내부에 새로만든 app을 추가 해줘야한다

```python
INSTALLED_APPS = [
    #...
    'myapp',
    #...
]
```

>app을 추가한 후에는 `manage.py migrate`  명령어를 입력해준다



## Fields

**model** 에서 가장 중요한것은 데이터베이스의 field 리스트이다. field는 클래스 속성에 의해 지정된다. `clean, save, delete` 은 field 이름으로 사용하면 안된다.



> example

```python
from django.db import models

class Musician(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    instrument = models.CharField(max_length=100)

class Album(models.Model):
    artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    release_date = models.DateField()
    num_stars = models.IntegerField()
```



## Field types

model에서 각각의 field는 해당 Field 클래스의 객체여야 한다. 

django에서 field 클래스 유형들

 - column 유형 : 데이터베이스에 어떤 종류의 데이터를 저장할지 알려준다( INTEGER, VARCHAR, TEXT)
 - HTML widet : form field를 렌더링할때 사용한다`( <input type="text">, <select>)`



## Field options

### CharField

- **CharField** 에는 데이터베이스 필드의 크기를 지정하는 **max_length**  인자가 필요하다 



### null

- null이 True라면 django는 빈 값을 NULL로 데이터베이스에 저장한다. False가 기본값이다.



### blank

- 값이 True라면 필드를 비워 둘 수 있다. 기본값은 False이다.
- null은 오직 데이터베이스와 관련된 반면 blank는 유효성검사와 관련있다. 필드에 `blank=True`가 있으면 필드를 비어있는 상태로 입력할 수 있다.


### choices

choices 가 주어진 widget은 텍스트 필드 대신에 선택 상자가 되어 주어진 선택사항으로 선택을 제한한다.

> example

```python
YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
    ('GR', 'Graduate'),
)
```

- 튜플의 첫번 째 요소는 데이터베이스에 저장되는 값이다. 
- 두번 째 요소는 기본 양식 widget 또는 ModelChoiceField에 표시된다. 
- 모델의 객체가 주어지면, choices 필드의 표시값은 get_FOO_display() 메소드를 통해 접근할 수 있다.

> example

```python
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
    
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```



### primary key

`primary_key=True`  로 주어지면, 이 필드는 모델의 primary key이다.

primary key 필드는 읽기전용이다. 기존  개체의 primary key 값을 바꾸고나서 그것을 저장하면, 이전 개체와 함께 새 개체가 만들어진다.

> example

```python
from django.db import models

class Fruit(models.Model):
    name = models.CharField(max_length=100, primary_key=True)
    
>>> fruit = Fruit.objects.create(name='Apple')
>>> fruit.name = 'Pear'
>>> fruit.save()
>>> Fruit.objects.values_list('name', flat=True)
['Apple', 'Pear']
```



## Relationships



### many-to-one relationships

**일대다 관계**

`djnago.db.models.ForeignKey`  클래스를 이용하여 필드를 선언한다

ForeignKey 필드 선언시에 관계될 모델 클래스를 인자로 줘야한다



> example

```python
# Car와 Manufacturer가 일대다 관계이다
# Manu..는 여러개의 Car와 관계를 가지지만
# Car는 단 하나의 Manu..와 관계를 가진다

from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer)
    # ...
```



```python
# 재귀적인 관계
# 단 첫번째 인자를 클래스가 아닌, 클래스 명을 문자열로 전달
# 필드가 선언되는 시점에서는 아직 클래스가 생성되지 않았기 때문

class Employee(models.Model):
    boss = models.ForeignKey('Employee')
    # ...
```



###  many-to-many relationships

**다대다 관계**

> example

```python
# Topping과 pizza는 서로 다대다 관계를 가진다
# ManyTOManyField는 두 클래스중 아무데나 한쪽에만 선언하면 된다

from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```



### One-to-one relationships

일대일 관계를 정의하려면, OneToOneField를 이용하면 됩니다. 다른 관계 필드와 마찬가지로 모델 클래스의 어트리뷰트로 선언하면 됩니다. 

일대일 관계는 다른 모델을 확장하여 새로운 모델을 만드는 경우 유용하게 사용될 수 있습니다.

OneToOneField는 관계를 맺는 모델 클래스를 첫번째 인자로 받습니다.

예를들어, 당신이 "가게(Places)" 정보가 담긴 데이터베이스를 구축하고 있다고 합시다. 아마도 데이터베이스에 주소, 전화번호 등의 정보가 들어가게 되겠지요. 그런데, 맛집 데이터베이스를 추가적으로 구축하게 되었습니다. 이럴때, 새로 Restaurant 모델을 만들어, 주소나 전화번호 등의 필드를 다시 선언할 수도 있겠지만, 이러한 반복을 피하기 위해 Restaurant 모델에 Place 모델에 대한 OneToOneField를 선언할 수도 있습니다. 

ForeignKey 필드와 마찬가지로 자기자신 또는 아직 선언되지 않은 모델에 대해서도 관계를 가질 수 있습니다.

OneToOneField 필드는 parent_link 라는 옵션을 제공합니다. 이 옵션에 대한 설명은 "[model field reference](https://docs.djangoproject.com/en/1.7/ref/models/fields/#ref-onetoone)"를 참고하세요.

OneToOneField 클래스가 자동적으로 모델의 Primary Key가 되었던 적이 있었더랬습니다. 하지만 지금은 더이상 그렇지 않습니다. 물론 직접 primary_key=True 로 지정하여 Primary Key로 만들 수는 있습니다. 어쨋든 결과적으로 이러한 이유 때문에 하나의 모델이 여러개의 OneToOneField를 가질 수 있게 되었습니다.

### Models across files

다른 앱에 선언된 모델과 관계를 가질 수 있습니다. 그렇게 하려면, 다른 앱의 모델을 import 해서 아래와 같이 관계 필드를 선언하면 됩니다.

```python
from django.db import models

from geography.models import ZipCode

class Restaurant(models.Model):

    # ...

    zip_code = models.ForeignKey(ZipCode)
```



### Field name restrictions

Django는 모델 필드이름에 2가지 제약을 두고 있습니다.

파이썬 예약어는 필드이름으로 사용할 수 없습니다. 이 경우 파이썬 문법에러가 발생됩니다.

```python
class Example(models.Model):

    pass = models.IntegerField() # 'pass' is a reserved word!
```

두번째로, 필드이름에 밑줄 두개를 연속으로 사용할 수 없습니다. 이는 Django에서 특별한 문법으로 사용되기 때문입니다. 

```python
class Example(models.Model):

    foobar = models.IntegerField() # 'foobar' has two underscores!
```



데이터베이스 컬럼이름에 밑줄을 두 개 넣어야 하는 상황이라면, db_column 옵션을 이용해서 이러한 제약을 우회할 수 있습니다.

SQL 예약어의 경우(join, where, select 등)에는 필드이름으로 허용됩니다. Django가 쿼리문을 만들때, 모든 컬럼명과 테이블 명을 모두 이스케이프 처리하기 때문입니다. 실제 데이터베이스 엔진에 맞게 알아서 말이죠.



### Custom field types

If one of the existing model fields cannot be used to fit your purposes, or if you wish to take advantage of some less common database column types, you can create your own field class. Full coverage of creating your own fields is provided in Writing custom model fields.

Django에서 제공하는 필드 타입들 중에 당신의 목적에 적합한 타입이 없거나 특정 데이터베이스에서만 제공하는 특별한 타입을 사용하고 싶다면, 필드를 직접 만들어 사용할 수도 있습니다. 이 부분에 대한 내용은 "[Writing custom model fields](https://docs.djangoproject.com/en/1.7/howto/custom-model-fields/)" 문서를 참고하세요.



## Meta options

아래와같이 모델클래스 내부에 Meta 라는 이름의 클래스 선언해서 모델에 메타데이터를 추가할 수 있습니다. 

```python
from django.db import models

class Ox(models.Model):

    horn_length = models.IntegerField()

    class Meta:

        ordering = ["horn_length"]

        verbose_name_plural = "oxen"
```



"모델 메타데이터"란 앞서 보았던 필드 단위의 옵션들과 달리 모델 단위의 옵션이라고 볼 수 있습니다. 예를들면, 정렬 옵션(ordering), 데이터베이스 테이블 이름(db_table), 또는 읽기 좋은 이름이나 복수(plural) 이름을 지정해 줄 수 있습니다(verbose_name, verbose_name_plural). 모델클래스에 Meta 클래스를 반드시 선언해야 하는 것은 아니며, 모든 옵션을 모두 설정해야 하는 것도 아닙니다.

모든 메타데이터 옵션들을 살펴보려면 "[model option reference](https://docs.djangoproject.com/en/1.7/ref/models/options/)" 문서를 참고하세요.



### Model attributes

### objects

모델 클래스에서 가장 중요한 어트리뷰트는 Manager 입니다. Manager 객체는 모델 클래스 선언을 기반으로 실제 데이터베이스에 대한 쿼리 인터페이스를 제공하며, 데이터베이스 레코드를 모델 객체로 인스턴스화 하는데 사용됩니다. 특별히 Manager를 할당하지 않으면 Django는 Default Manager 객체를 클래스 어트리뷰트로 자동 할당 합니다. 이때 어트리뷰트의 이름이 objects 입니다. 

Manager는 모델 클래스를 통해 접근할 수 있으며, 모델 인스턴스(객체)를 통해서는 접근 할 수 없습니다. 



### Model methods

"레코드(row) 단위"의 기능을 구현하려면, 모델 클래스에 메서드를 구현해주면 됩니다. "테이블 단위"의 기능의 경우에는 Manager에 구현해 줍니다.

이러한 규칙은 비즈니스 로직을 모델에서 관리하는데 있어 중요한 테크닉입니다.

예를들어, 아래와 같이 커스텀 메서드를 추가할 수 있습니다.

```python


from django.db import models

class Person(models.Model):

    first_name = models.CharField(max_length=50)

    last_name = models.CharField(max_length=50)

    birth_date = models.DateField()

    def baby_boomer_status(self):

        "Returns the person's baby-boomer status."

        import datetime

        if self.birth_date < datetime.date(1945, 8, 1):

            return "Pre-boomer"

        elif self.birth_date < datetime.date(1965, 1, 1):

            return "Baby boomer"

        else:

            return "Post-boomer"

    def _get_full_name(self):

        "Returns the person's full name."

        return '%s %s' % (self.first_name, self.last_name)

    full_name = property(_get_full_name)
```



"model instance reference"를 보면 모델 클래스에 자동적으로 주어지는 메서드들이 나열되어있습니다. 이러한 메서드를 여러분이 오버라이드해서 사용할 수 있는데 이에 대한 자세한 내용은 다음 절에서 설명합니다.

그 중에서 거의 항상 오버라이드 해줘야 할 메서드들이 있습니다.



**__unicode__() (Python 2)**

모델 객체가 문자열로 표현되어야 하는 경우에 호출됩니다. admin이나 console 에서 모델 객체를 표시하는 경우에 많이 쓰이게 됩니다.  



**__str__() (Python 3)**

파이썬 3의 경우에는, __unicode__() 대신에 __str__() 로 선언합니다.

기본 구현은 아무런 도움이 되지 않는 문자열을 리턴하기 때문에, 모든 모델에 대해 오버라이드 해서 알맞게 구현해주는게 좋습니다.



**get_absolute_url()**

이 메서드는 Django가 해당 모델 객체의 URL을 계산할 수 있도록 합니다. Django는 이 메서드를 모델 객체를 URL로 표현하는 경우에 사용하며, admin 사이트에서도 사용합니다. 

모델 객체가 유일한 URL을 가지는 경우에 이 메서드를 구현해주어야 합니다.



## Overriding predefined model methods

모델 클래스가 제공하는 기본적인 메서드들을 커스터마이즈 하고자 하는 경우가 있을 수 있습니다. 특히나, save() 나 delete() 메서드를 오버라이드 하고자 하는 경우가 많을 것입니다.

이 메서드들을 얼마든지 오버라이드 해서 동작을 수정할 수 있습니다.

모델 객체를 저장할 때마다, 특정한 동작을 수행하기 위해서 save() 메서드를 오버라이드 할 수 있습니다.

```python
from django.db import models

class Blog(models.Model):

    name = models.CharField(max_length=100)

    tagline = models.TextField()

    def save(self, *args, **kwargs):

        do_something()

        super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.

        do_something_else()
```



아예 저장을 하지 못하게 할 수도 있습니다.

```python
from django.db import models

class Blog(models.Model):

    name = models.CharField(max_length=100)

    tagline = models.TextField()

    def save(self, *args, **kwargs):

        if self.name == "Yoko Ono's blog":

            return # Yoko shall never have her own blog!

        else:

            super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.

```



부모 클래스의 메서드를 호출해주는 것을 잊지마세요. 위 예제 코드에서 super(Blog, self).save(*args, **kwargs) 를 호출함으로써, 실제로 데이터베이스에 객체가 저장되었습니다. 부모 클래스의 메서드를 호출해주지 않는다면, 실제 데이터베이스에는 아무런 동작도 일어나지 않습니다.

오버라이드한 메서드의 인자를 그대로 부모 클래스의 메서드에 넘겨주는 것도 잊지 마세요. "*args, **kwargs" 부분이 그것입니다. Django는 계속적으로 옵션이 추가되고 확장 될것입니다. 새로운 인자가 추가될 수도 있겠지요. 그렇기 때문에 당신이 오버라이드 하는 메서드에 "*args, **kwargs" 형태로 인자를 넘겨 받고 그대로 부모 메서드에 넘겨준다면 당신의 코드는 Django의 변경사항에 관계없이 항상 잘 동작하게 됩니다.

재정의한 모델 메서드는 벌크 조작시에는 호출되지 않습니다.

QuerySet을 이용해서 한번에 삭제할 수도 있습니다. 즉, 조건에 맞는 여러개의 객체를 한번에 삭제할 수 있습니다. 삭제 뿐만 아니라 update도 가능합니다. 이러한 동작을 bulk-operation이라고 합니다. 이 경우에는 오버라이드된 delete() 메서드가 호출되지 않습니다. 그러므로 "pre_delete", "post_delete" 시그널을 이용해야 합니다.

객체의 생성이나 수정이 bulk 로 이루어지는 경우에는 오버라이드된 메서드를 실행할 방법이 없습니다. save() 메서드도 호출되지 않고, pre_save, post_save 시그널도 발생되지 않기 때문입니다.



## Model inheritance

Django 에서의 모델 상속은 파이썬에서의 클래스 상속과 거의 동일한 형태로 이루어 집니다. 단, 모델 상속의 경우에 베이스 클래스는 django.db.models.Model 클래스의 서브 클래스여야 합니다. 

이때, 부모 모델이 자신의 데이터베이스 테이블을 가지도록 할 것인지, 아니면 부모 모델은 실제로 테이블을 생성하지 않고 자식 모델들이 부모 모델에 선언된 공통 필드를 각각 자신의 테이블에 생성할지를 결정해야 합니다.

위와 같은 특징으로 인해 Django에서는 3가지 스타일의 모델 상속이 존재합니다.

먼저 Abstract class가 있습니다. 이 경우 부모 클래스는 실제로 테이블을 만들지 않고 자식 클래스들에 필드 선언이나 메서드 선언만 상속해 줍니다. 이를 상속 받은 자식 모델은 데이터베이스 테이블에 부모 클래스와 자기 자신에 선언된 필드들을 가지게 됩니다.

두번째는 이미 존재하는 모델(아마도 다른 app에서 선언되어 사용중인..)을 상속하는 경우입니다. 이 경우 부모 모델은 자신의 데이터베이스 테이블을 가지며 자식 모델과는 별개로 독립적으로 사용 가능합니다. 이를 Multi-table 상속이라고 부릅니다.

마지막으로, 모델의 필드 선언은 전혀 변경하지 않고 파이썬 레벨의 코드만 수정하고자 하는 경우 사용하는 방법입니다. 이 경우, Proxy 모델을 사용합니다.



## Abstract base classes

추상 클래스 방식은 여러개의 모델 클래스가 공통적인 정보를 가지도록 하고 싶은 경우에 유용합니다. Meta클래스에 abstract=True 옵션을 선언하면 그 모델 클래스는 추상클래스가 됩니다. 이렇게 만들어진 추상클래스는 데이터베이스에 테이블을 생성하지 않습니다. 다만, 다른 모델들이 이 클래스를 상속받을 수 있으며, 부모클래스에 선언된 필드들은 자식 클래스의 테이블에 추가됩니다. 이때 부모 클래스와 자식 클래스가 동일한 이름의 필드를 가지고 있는 경우에는 Django가 예외를 발생시킵니다.

아래는 추상 클래스 방식의 예제입니다.

```python
from django.db import models

class CommonInfo(models.Model):

    name = models.CharField(max_length=100)

    age = models.PositiveIntegerField()

    class Meta:

        abstract = True

class Student(CommonInfo):

    home_group = models.CharField(max_length=5)
```



결과적으로 Student 모델은 3개의 필드(name, age, home_group)를 가지게 됩니다. CommonInfo 모델은 일반적인 모델클래스 처럼 독립적으로 사용 될 수 없습니다. 앞에서 언급했듯이 데이터베이스에 테이블을 만들지도 않고 Manager도 가지지 않습니다. 또한 인스턴스화 할 수도 없습니다. 

추상클래스를 이용한 모델 상속은 데이터베이스에 영향을 주지 않고, 파이썬 레벨에서 공통 필드를 하나의 부모 클래스에서 관리할 수 있는 방법을 제공해줍니다. 앞에서 설명했듯이, 이 경우 자식 모델 당 하나의 테이블만 만들어 진다는 것을 명심하세요.



### Meta inheritance

자식 클래스가 Meta 클래스를 가지지 않는 경우, 자식 클래스는 부모의 메타 클래스를 상속 받습니다. 만약 자식 클래스에서 부모 Meta 클래스를 상속 받으려면 아래와 같이 부모 클래스에 선언된 Meta 클래스로부터 상속 받으면 됩니다. 

```python
from django.db import models

class CommonInfo(models.Model):

    # ...

    class Meta:

        abstract = True

        ordering = ['name']

class Student(CommonInfo):

    # ...

    class Meta(CommonInfo.Meta):

        db_table = 'student_info'
```



이 경우 Django는 내부적으로 한가지 조정을 합니다. 자식클래스의 Meta 클래스를 활성화하기 전에, abstract 옵션을 False로 변경합니다. 즉, 추상 클래스를 상속한다고 해서 자식클래스 또한 추상 클래스가 되지는 않는다는 얘기입니다. 물론 자식클래스 또한 추상 클래스가 되길 원할 수도 있습니다. 이경우에는 자식 클래스의 Meta 클래스에서 abstract=True 옵션을 한번 더 선언해주면 됩니다.

몇몇 Meta 클래스 옵션은 추상 클래스에는 사용할 수 없습니다. 예를들면 추상 클래스는 데이터베이스 테이블을 만들지 않으므로, db_table 옵션은 아무 의미가 없습니다. 물론 자식 클래스들에는 상속됩니다. 그런데 모든 자식 클래스들이 동일한 테이블을 사용하도록 하려는것은 아닐겁니다. 이와같이, 추상 클래스에서 Meta 클래스 옵션을 선언할 때 주의를 기울여야 합니다.



### Be careful with related_name and related_query_name

Person 모델에 ForeignKey 필드가 하나 있다고 칩시다. 이 때, 이 ForeignKey 필드는 Group 모델과 관계를 가집니다. 이 경우 기본적으로 Group 모델에 person_set 이라는 어트리뷰트가 자동으로 생성되며, 이 어트리뷰트는 Group 모델과 관계된 Person 모델들을 엑세스 할 수 있도록 해줍니다. 이때 related_name 옵션을 지정하여, person_set이 아닌 다른 이름을 직접 지정할 수 있습니다. 

그런데, Animal 이라는 모델 또한 Group 모델에 대한 ForeignKey 필드를 가지고 있다면 어떻게 될까요? Group 필드는 person_set 과 animal_set 이라는 역참조 어트리뷰트를 가지게 됩니다. 그런데, Person 모델과 Animal 모델의 ForeignKey 필드에 related_name 옵션을 동일하게 설정했다고 생각해봅시다. 이 경우 각각의 모델에 대한 역참조 어트리뷰트가 중복되기 때문에 에러가 발생됩니다. 물론 이 경우 Person 모델과 Animal 모델 각각의 related_name 값을 다르게 설정해서 문제를 해결할 수 있습니다.

그런데 만약 추상 모델에 관계필드가 선언되어있고, related_name 옵션을 설정했다고 가정해봅시다. 이 추상 모델을 상속 받는 자식들은 동일한 역참조 이름을 가지는 관계필드를 가지게 됩니다.

related_name 값을 설정할때, 그 값으로 '%(app_label)s' 와 '%(class)s' 조합하여 사용하면 이러한 문제를 해결할 수 있습니다.

'%(class)s' 는 자식 클래스의 소문자화된 이름으로 치환됩니다.

'%(app_label)s'는 자식 클래스가 선언된 앱의 이름입니다. 역시 소문자화 된 이름입니다. 

설치된 앱들간에 앱 이름은 중복될 수 없으며, 하나의 앱내에서 모델의 이름은 중복될 수 없습니다. 그러므로 이 두개의 조합으로 유일한 이름이 만들어 지게됩니다.

```python
# common/models.py

from django.db import models

class Base(models.Model):

    m2m = models.ManyToManyField(OtherModel, related_name="%(app_label)s_%(class)s_related")

    class Meta:

        abstract = True

class ChildA(Base):

    pass

class ChildB(Base):

    pass

# rare/models.py:

from common.models import Base

class ChildB(Base):

    pass
```



common/models.py에 선언된 클래스들 부터 살펴봅시다. ChildA.m2m 필드의 역참조 이름은 common_childa_related 가 되며, ChildB.m2m 필드는 common_childb_related 가 됩니다. 즉, OtherModel은 common_childa_related, common_childb_related 두 개의 역참조 어트리뷰트를 가지게 됩니다.

rare/models.py 에 선언된 모델의 경우에 역참조 이름은 rare_childb_related 가 됩니다. 

참고로, 만약 이 경우에 Base 모델 클래스에서 m2m 필드에 related_name 옵션을 지정하지 않으면, 상속받은 자식클래스의 이름을 기준으로 역참조 이름이 결정됩니다. 즉, ChildA 클래스는 childa_set, ChildB 클래스는 childb_set 으로 역참조 이름이 결정됩니다. 

그런데, 위의 예제에서 common 앱과 rare 앱이 동일한 이름의 모델(ChildB)이 Base 모델을 상속받았습니다. 그러므로 역참조 이름이 중복(childb_set)되어 에러가 발생됩니다.



## Multi-table inheritance 

Multi-table 상속은 부모 모델이든 자식 모델이든 모두 각자의 데이터베이스 테이블을 가집니다. 즉, 공통부분의 데이터는 부모모델의 테이블에 저장하고, 자식모델의 데이터는 자식모델의 테이블에 저장되며, 자식 모델은 부모 모델에 대한 링크를 가집니다. 이때, 링크는 내부적으로 OneToOneField를 사용합니다.

아래의 Multi-table 상속의 예제입니다.

```python
from django.db import models

class Place(models.Model):

    name = models.CharField(max_length=50)

    address = models.CharField(max_length=80)

class Restaurant(Place):

    serves_hot_dogs = models.BooleanField(default=False)

    serves_pizza = models.BooleanField(default=False)
```



Place 모델에 선언된 모든 필드는 Restaurant 모델에서도 사용가능합니다. 물론, 각 모델의 필드값들은 서로 다른 테이블에 각각 저장됩니다.

```python
>>> Place.objects.filter(name="Bob's Cafe")

>>> Restaurant.objects.filter(name="Bob's Cafe")
```

만약 Place 모델을 통해서 Restaurant 모델 객체를 쿼리할 수 있습니다. 단 Place 모델에 선언된 필드만 엑세스할 수 있으며, 자동으로 제공되는 restaurant 어트리뷰트를 통해 Restaurant 객체에 접근 할 수도 있습니다. 이때, 어트리뷰트 이름은 자식 클래스의 이름을 소문자화한 이름입니다. 

```python
>>> p = Place.objects.get(id=12)

# If p is a Restaurant object, this will give the child class:

>>> p.restaurant

<Restaurant: ...>
```

하지만, 위의 예제에서 p가 Restaurant 객체가 아니라면(Place 모델을 통해 직접 생성되었다거나, Restaurant 모델이 아닌 다른 자식클래스를 통해 만들어진 경우), restaurant 어트리뷰트에 접근하면 Restaurant.DoesNotExist 예외가 발생됩니다.

```python
>>> p = Place.objects.create(name="Bob's Cafe", address="Bob's Street")

>>> p.restaurant  # Restaurant.DoesNowExist 예외 발생
```



### Meta and multi-table inheritance 

multi-table 상속 관계에서 자식 모델은 부모 모델의 메타클래스를 상속 받아야 할까요? 부모 모델과 자식 모델 모두 자신의 테이블을 가지기 때문에 메타 클래스를 상속받는 경우 여러가지 문제를 발생시킬 수 있습니다. 

이와같은 이유로, multi-table 상속의 경우 기본적으로 자식모델은 부모모델의 Meta 클래스를 상속받지 않습니다. 단, 자식모델에 ordering, get_latest_by 옵션이 지정되지 않은 경우에 한해, 부모 모델의 해당 설정값을 상속받습니다.

만약 부모 모델의 ordering 값을 상속받고 싶지 않다면, 자식모델에 ordering 옵션을 아래와 같이 설정해주면 됩니다.

```python
class ChildModel(ParentModel):

    # ...

    class Meta:

        # Remove parent's ordering effect

        ordering = []
```



### Inheritance and reverse relations 

예를들어 아래와 같이 Supplier 모델이 Place 모델을 상속받으면서, Place 모델에 대한 ManyToManyField도 가지고 있다고 하면,  

```python
class Supplier(Place):

    customers = models.ManyToManyField(Place)

아래와 같이 에러가 발생됩니다.

Reverse query name for 'Supplier.customers' clashes with reverse query

name for 'Supplier.place_ptr'.

HINT: Add or change a related_name argument to the definition for

'Supplier.customers' or 'Supplier.place_ptr'.
```



ManyToManyField 선언에 아래와 같이 related_name 옵션을 추가해주면 이 문제를 해결할 수 있습니다.

`models.ManyToManyField(Place, related_name='provider')`



### Specifying the parent link field

앞에서 언급했듯이 multi-table 상속의 경우 OneToOneField 가 자동으로 생성됩니다. 만약 이 OneToOneField를 직접 지정하고 싶다면, OneToOneField 를 추가하고 필드 옵션에 parent_link=True로 설정해주면 됩니다.



### Proxy models 

multi-table 상속을 사용하는 경우, 각각의 자식 클래스들 마다 테이블이 생성됩니다. 부모 모델이 가지는 데이터 이외에 추가적으로 자식 모델이 가지는 데이터를 저장하기 위한 당연한 동작입니다. 하지만 가끔은 부모 모델의 메서드(파이썬 코드)만 재정의 한다던가 새로 추가하고 싶을 수도 있습니다. 즉, 테이블을 가지는 모델을 상속받되, 자식모델은 테이블을 만들필요가 없는 경우에 해당됩니다.

이런 경우에 proxy 모델 상속을 사용합니다. 이 경우 proxy 모델(자식모델)을 이용해서 모델객체를 만들고, 수정하고 삭제할 수도 있습니다. 물론 이러한 동작은 원본(부모)모델의 테이블 상에서 이루어 집니다. 

proxy 모델이 원본모델과 다른 점은 기본 정렬값과 같은 설정값을 원하는 데로 변경할 수 있다는 점입니다. 물론 원본 모델의 설정값은 변경되지 않습니다.

Proxy 모델 선언은 일반 모델들과 동일합니다. 다만, Meta 클래스에 proxy=True 로 선언해주면 됩니다.

예를들어, Person 모델에 메서드 하나를 추가하고 싶다면 아래와 같이 할 수 있습니다.

```python
from django.db import models

class Person(models.Model):

    first_name = models.CharField(max_length=30)

    last_name = models.CharField(max_length=30)

class MyPerson(Person):

    class Meta:

        proxy = True

    def do_something(self):

        # ...

        pass
```



MyPerson 클래스는 부모 클래스인 Person 클래스의 테이블을 사용합니다. 그렇기 때문에, 아래와 같이 Person 모델로 레코드를 추가하더라도 MyPerson 클래스로 조회가 가능합니다.

```python
>>> p = Person.objects.create(first_name="foobar")

>>> MyPerson.objects.get(first_name="foobar")

<MyPerson: foobar>
```



메서드를 재정의나 추가하는것 외에도, 기본 정렬 옵션을 변경할 수도 있습니다. 

```python
class OrderedPerson(Person):

    class Meta:

        ordering = ["last_name"]

        proxy = True
```



이제, Person 모델을 이용한 쿼리를 정렬되지 않은 상태로, OrderedPerson 모델로 쿼리할때는 last_name 기준으로 정렬되어 리턴됩니다.



### QuerySets still return the model that was requested 

Person 모델로 쿼리하는 경우 리턴되는 모델 객체는 MyPerson이 아니라 Person 객체입니다. 즉, 어떤 모델 클래스로 쿼리했는지에 따라 리턴되는 모델 객체의 타입이 결정됩니다. proxy 모델에서 변경한 사항은 부모 모델에 영향을 끼치지 않는 다는 사실을 기억하시기 바랍니다.



### Base class restrictions

proxy 모델은 하나의 (추상 클래스가 아닌)모델만 상속받을 수 있습니다. proxy 모델은 여러개의 테이블로부터 레코드를 쿼리하는 기능은 제공하지 않기 때문입니다. 추상 모델 클래스의 경우에는 그 추상 모델 클래스가 필드를 가지지 않는다면 몇 개든 상속이 가능합니다. 

Proxy 모델은 자신이 선언하지 않은 Meta 옵션에 한해 부모(추상 클래스가 아닌) 모델로부터 상속받습니다.



### Proxy model managers 

Proxy 모델에 manager를 지정하지 않는 경우, 부모 모델의 manager를 상속받습니다. 직접 지정한 경우, 지정한 manager가 디폴트로 사용되며, 부모클래스의 manager도 사용할 수 있습니다.

아래와 같이 기본 manager를 변경할 수 있습니다.

```python
from django.db import models

class NewManager(models.Manager):

    # ...

    pass

class MyPerson(Person):

    objects = NewManager()

    class Meta:

        proxy = True
```



아래와 같이 부모 클래스의 기본 manager를 상속 받되, 새로운 manager를 추가하려면 아래와 같이 추상클래스에 새로운 manager를 선언하고 그 추상클래스를 상속받으면 됩니다.

```python
# Create an abstract class for the new manager.

class ExtraManagers(models.Model):

    secondary = NewManager()

    class Meta:

        abstract = True

class MyPerson(Person, ExtraManagers):

    class Meta:

        proxy = True
```

위와 같이 사용하는 경우가 흔하지는 않습니다. 다만, 위와 같이 할 수 있다는 것만 알아두세요.



### Differences between proxy inheritance and unmanaged models 

모델 클래스를 하나 선언해서 managed=False 옵션을 주면, 해당 모델은 자동으로 테이블을 생성하지 않게 됩니다. 직접 테이블을 만들어 주어야 합니다. 그런데 여기에 db_table 옵션에 기존에 존재하는 모델의 테이블 이름을 지정하게 되면 어떨까요? proxy 모델과 거의 비슷해 보입니다.

하지만 이 둘은 조금 다른 특징을 가지며, 목적에 맞게 사용해야합니다.

한가지 차이점은 unmanaged 모델의 경우 직접 데이터베이스 테이블을 생성/수정 해주어야 한다는 것입니다. proxy모델의 경우 원본 모델이 변경되더라도 proxy 모델을 수정할 일은 없습니다. 하지만 unmanaged 모델의 경우, 원본 모델에 변경사항이 생길때마다 unmanaged 모델 클래스 또한 동일하게 수정해주어야 합니다. 생각만 해도 피곤한 일입니다.

또다른 차이점은 manager 가 처리되는 방식입니다. proxy 모델의 경우 원본 모델의 디폴트 Manager 를 포함한 모든 Manager를 상속받습니다. unmanaged 모델의 경우 Django 입장에서 원본 모델과 아무런 관련이 없기 때문에 manager는 unmanaged 모델 자체의 선언에 따릅니다. 참고로 multi-table 모델의 경우에는 부모 모델의 manager를 상속받지 않습니다. 

두가지 방식(proxy, unmanaged)을 구현하면서 우리는 하나의 옵션으로 합쳐보려는 시도를 했었습니다만, API가 복잡하고 이해하기 어려워져버려서 그냥 2가지 옵션을 따로 제공하기로 했습니다. 

이 2가지 옵션 중 어떤것을 사용할지를 결정하는 일반적인 규칙은 아래와 같습니다.

1. 만약에 당신이 이미 존재하는 모델이나 데이터베이스 테이블을 참조하되, 원본 테이블의 모든 컬럼이 필요하지 않다면, Meta.managed=False 옵션을 사용하세요. Django에 의해 제어되지 않는 데이터베이스 뷰나 테이블을 사용하는 경우에 유용합니다.
2. 만약에 기존 모델의 파이썬 코드들을 오버라이드 하되, 원본 모델과 동일한 필드를 가지도록 하고 싶다면 Meta.proxy=True 옵션을 사용하세요.



### Multiple inheritance 

파이썬의 상속과 마찬가지로 Django 모델 또한 다중 상속(Multiple Inheritance)이 가능합니다. 단, 이 경우 파이썬의 "name resolution rules" 이 적용됩니다. 예를들어, 부모 클래스들이 각각 Meta 클래스를 가지는 경우 첫번째 부모의 Meta 클래스를 상속받게 되며, 나머지 부모 클래스의 Meta 클래스는 무시됩니다.

일반적으로 여러 부모 모델로부터 상속받는 경우가 별로 없습니다만, 많이 쓰이는 경우는 'mix-in' 클래스 들입니다. 공통적으로 사용되는 필드나 메서드들을 묶어 하나의 클래스로 선언하고 이 클래스를 여러 모델 클래스에 상속하는 경우 입니다. 물론 이 경우, 특정 필드가 어느 부모 클래스에 선언되어있는 것인지 명확하게 간단명료하게 정리되어야 합니다.

Django 1.7 이전에는 id 필드를 가지는 여러 모델로부터 상속을 받더라도 에러를 발생시키지 않았습니다. 하지만 이 경우 데이터 손실이 발생할 수 있습니다. 아래의 예제를 예로들면, "id" 필드가 중복되어 Article.id 필드는 무시됩니다.

```python
class Article(models.Model):

    headline = models.CharField(max_length=50)

    body = models.TextField()

class Book(models.Model):

    title = models.CharField(max_length=50)

class BookReview(Book, Article):

    pass
```



아래의 예제는 이로인한 문제점을 보여주고 있습니다. 아래의 예제에서 review 객체의 id값은 Book 클래스의 id 필드에 의해 결정됩니다. "class BookReview(Book, Article):" 에서 Book을 먼저 선언해주었기 때문입니다. 

결과적으로 review 객체의 id 값은 Book 모델에 의해 자동생성된 ID값인 1이 되며, 이로인해 Article 객체가 새로 추가되는 것이 아니라 id값이 1인 Article 객체를 덮어쓰게 됩니다.

```python
>>> article = Article.objects.create(headline='Some piece of news.')

>>> review = BookReview.objects.create(

...     headline='Review of Little Red Riding Hood.',

...     title='Little Red Riding Hood')

>>>

>>> assert Article.objects.get(pk=article.pk).headline == article.headline

Traceback (most recent call last):

  File "<console>", line 1, in <module>

AssertionError

>>> # the "Some piece of news." headline has been overwritten.

>>> Article.objects.get(pk=article.pk).headline

'Review of Little Red Riding Hood.'
```



이러한 상황을 해결하려면 id 필드가 겹치지 않도록 부모 모델의 id 필드를 명시적으로 선언해주면 됩니다.

```python
class Article(models.Model):

    article_id = models.AutoField(primary_key=True)

    ...

class Book(models.Model):

    book_id = models.AutoField(primary_key=True)

    ...

class BookReview(Book, Article):

    pass
```



또는 아래와 같이 id 필드(AutoField)를 가지는 공용 모델(Piece)을 만들어 id 값이 항상 공용모델을 기준으로 생성되도록 할 수도 있습니다.

```python
class Piece(models.Model):

    pass

class Article(Piece):

    ...

class Book(Piece):

    ...

class BookReview(Book, Article):

    pass
```



###  Field name “hiding” is not permitted 

보통의 파이썬 클래스 상속에서, 자식 클래스는 부모 클래스의 어트리뷰트를 오버라이드 할 수 있습니다. 하지만 Django에서 필드 어트리뷰트의 경우에는 오버라이드가 허용되지 않습니다. 만약 부모 클래스가 author 라는 이름의 필드가 선언되어 있다면, 자식클래스는 같은 이름의 필드를 선언할 수 없습니다.

이러한 제약은 필드 어트리뷰트에만 해당합니다. 일반 파이썬 어트리뷰트는 원한다면 오버라이드 가능합니다. multi-table 상속의 경우에는 어트리뷰트 이름이 다르더라도 데이터베이스 컬럼명을 직접 지정하여, 부모와 자식 테이블 각각의 컬럼명을 동일하게 사용할 수는 있습니다.

참고로 Django는 부모 모델의 필드를 오버라이드 하면 FieldError를 발생시킵니다.