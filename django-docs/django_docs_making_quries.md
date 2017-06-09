# django - Making queries
---

## Creating objects

모델 클래스는 데이터베이스 테이블을 나타낸다
클래스의 인스턴스는 데이터베이스 테이블의 특정 레코드를 나타낸다

객체를 생성하려면 키워드 인자를 사용하여 모델 클래스에 인스턴스를 생성 후, save ()를 호출하여 객체를 데이터베이스에 저장한다

```python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

---

## Saving changes to objects

`save()` 사용해서 이미 데이터베이스에 있는 객체의 변경사항을 저장할 수 있다
django는 save()를 호출 할 때 까지 데이터베이스에 접근하지 않는다

### Saving ForeignKey and ManyToManyField fields

**<ForeignKey>**

**ForeignKey** 필드를 업데이트하는 것은 보통의 필드를 저장하는 것과 똑같은 방식으로 작동한다. 필드에 올바른 유형의 객체를 할당하기만 하면 된다.

```python
>>> from blog.models import Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

**<ManyToManyField>**

**ManyToManyField** 를 업데이트 할 때는 조금 다르다. 필드에 **add()** 메소드를 사용하여 레코드를 추가한다.

```python
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

한번에 여러가지 레코드를 추가하려면 **add()** 에 여러가지 인자를 넣으면 된다

```python
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

---

## Retrieving objects

데이터베이스에서 객체를 검색하기위해 모델 클래스의 **Manager** 를 통해 **QuerySet** 을 생성한다
QuerySet는 데이터베이스의 객체 컬렉션을 나타낸다. 0 개, 하나 또는 여러 개의 **filter** 를 가질 수 있다. **filter** 는 주어진 매개 변수를 기반으로 쿼리 결과의 범위를 좁 힙니다. SQL 용어에서 **QuerySet** 은 SELECT 문과 같으며 **filter** 는 WHERE 또는 LIMIT와 같은 제한 절이다.

모델의 **Manager** 를 사용해 **QuerySet** 을 가져올 수 있다.
>Manager : django 모델에 데이터베이스 쿼리 작업을 제공하는 인터페이스이다. django 어플리케이션의 모든 모델들은 모두 하나 이상의 manager가 존재한다.

```python
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```

>Manager는 모델 인스턴스가 아닌 모델 클래스로만 접근할 수 있다

### Retrieving all objects

**all()** 메소드를 사용하면 테이블의 모든 객체를 QuerySet으로 가져올 수 있다

```python
>>> all_entries = Entry.objects.all()
```

### Retrieving specific objects with filters

필요한 객체만을 선별해서 QuerySet으로 가져오는 두가지 방법이 있다

**filter(**kwargs)**
주어진 검색 매개변수와 **일치하는** 객체들을 포함하는 QuerySet을 리턴한다

**exclude(**kwargs)**
주어진 검색 매개변수와 **일치하지 않는** 객체들을 포함하는 QuerySet을 리턴한다

#### Chaining filters
#### Filtered QuerySets are unique
#### QuerySets are lazy

### Retrieving a single object with get()
### Other QuerySet methods
### Limiting QuerySets
### Field lookups

### Lookups that span relationships
#### Spanning multi-valued relationships

### Filters can reference fields on the model
### The pk lookup shortcut
### Escaping percent signs and underscores in LIKE statements

### Caching and QuerySets
#### When QuerySets are not cached

## Complex lookups with Q objects
## Comparing objects
## Deleting objects
## Copying model instances
## Updating multiple objects at once

## Related objects
### One-to-many relationships
#### Forward
#### Following relationships “backward”
#### Using a custom reverse manager
#### Additional methods to handle related objects

### Many-to-many relationships
### One-to-one relationships
### How are the backward relationships possible?
### Queries over related objects

## Falling back to raw SQL
