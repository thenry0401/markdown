# 제어문

## if, elif, else(조건문)

if와 else는 조건이 참인지 거짓인지 판단하는 파이썬 선언문(Statement)이며, elif는 else내의 if를 중첩해야 할 때 사용한다.

```
if 조건:
	조건이 참일 경우
else:
	조건이 거짓일 경우
```

```
if 조건1:
	조건1이 참일 경우
else:
	조건1이 거짓일 경우
	
	if 조건2:
		조건1은 거짓이나, 조건2는 참일 경우
	else:
		조건1,2가 모두 거짓일 경우
```

위 코드는 아래와 같이 `elif`로 줄여쓸 수 있다.

```
if 조건1:
	조건1이 참일 경우
elif 조건2:
	조건1은 거짓이나, 조건2가 참일 경우
else:
	조건1,2가 모두 거짓일 경우
```

---

## 조건표현식

- `참일경우 if 조건식 else 거짓일 경우`

## 중첩 조건표현식

```
# 조건이 2개일 경우
조건1이 참일경우 if 조건1 else 조건1은 거짓이나 조건2가 참일경우 if 조건2 else 조건1,2가 모두 거짓일 경우
```
---

## for문 (조건에 따른 순회)

- 기본형태

시퀀스형 데이터를 순회하고자 할 때 사용한다.

```
for 항목 in 순회가능(iterable)객체:
   <항목을 사용한 코드>
```

iterable한 객체에는 문자열, 튜플, 딕셔너리, 셋 등이 있다.

```
>>> champion_list = ['lux', 'ahri', 'ezreal', 'zed']
>>> for champion in champion_list:
...   print(champion)
... 
lux
ahri
ezreal
zed
```

>딕셔너리에서 키나 값을 순회할 때는, iterable한 객체의 위치에 dict.keys()나 dict.values()를 사용한다.
키, 값을 모두 순회할 때에는 dict.items()를 사용한다.

### 중첩

```
for 항목1 in iterable객체1:
  iterable객체1을 순회하며 실행할 코드
  for 항목2 in iterable객체2:
    iterable객체1 내부에서 새로운 iterable객체2를 순회하며 실행할 코드
```

### 중단하기 (break)

데이터를 순회하던 중, 특정 조건에서 순회를 멈추고 반복문을 빠져나갈 때 사용한다.

```
for 항목 in iterable객체:
  (반복문을 중단하고 싶을때)break
```

### 건너뛰기 (continue)

데이터를 순회하던 중, 반복문을 중단하지 않고 다음 반복으로 건너뛸 때 사용한다.

```
for 항목 in iterable객체:
  (현재의 반복을 중간에 그만두고 다음 반복으로 건너뛰고 싶을 때)continue
```

### break확인 (else)

for문으로 데이터를 순회하던 중, break문이 호출되지 않고 반복문이 종료되면 else문이 실행된다.

```
for 항목 in iterable객체:
  pass
else:
  break가 한 번도 호출되지 않았을 경우의 코드
```

### 여러 시퀀스 동시순회 (zip)

```
>>> fruits = ['apple', 'banana', 'melon']
>>> colors = ['red', 'yellow', 'green', 'purple']
>>> for fruit, color in zip(fruits, colors):
...   print('fruit:', fruit, ' color:', color)
... 
fruit: apple  color: red
fruit: banana  color: yellow
fruit: melon  color: green
```

>zip으로 묶은 시퀀스들 중, 가장 짧은 시퀀스가 완료되면 순회가 종료된다.

zip을 사용하면 여러 시퀀스로부터 튜플을 만들 수 있다.

```
>>> list(zip(fruits, colors))
```

>zip으로 반환되는 것은 리스트가 아닌 zip클래스 형태의 iterable객체이기 때문에, 리스트 형태로 사용하려면 list()함수를 사용해준다.

>dict()함수를 사용할 경우 딕셔너리 객체가 만들어지게 된다.

### 숫자 시퀀스 생성 (range)

range()함수는 특정 범위의 숫자 스트림 데이터를 반환한다.

```
range(start, stop, step)
```

zip과 마찬가지로, iterable한 객체를 반환하며, 따라서 for문을 순회할 수 있다.

```
>>> for x in range(0, 10):
...   print(x)
... 
0
1
2
3
4
5
6
7
8
9
```

## while문 (반복문)

for문과 유사하나, while뒤의 조건이 참일 경우에 계속해서 반복한다.

```
while 조건:
  조건이 참일경우 실행
  조건이 거짓이 될 경우까지 계속해서 반복
>>> count = 0
>>> while count < 10:
...   print(count)
...   count += 1
... 
0
1
2
3
4
5
6
7
8
9
```

## 컴프리헨션 (Comprehension)

- 함축 또는 내포

iterable한 객체로부터 파이썬의 자료구조를 만드는 방법. 가독성과 사용성에서 이득을 얻을 수 있을 경우 항상 사용해준다.

### 리스트 컴프리헨션

```
[표현식 for 항목 in iterable객체]
```

range와 for문을 사용할 경우

```
>>> numbers = []
>>> for item in range(1, 6):
...   numbers.append(item)
... 
>>> numbers
[1, 2, 3, 4, 5]
```

리스트 컴프리헨션을 사용할 경우

```
>>> [item for item in range(1, 6)]
[1, 2, 3, 4, 5]
```

리스트 컴프리헨션의 중첩

```
for color in colors:
  for fruit in fruits:
```

```
[(color, fruit) for color in colors for fruit in fruits]
```
셋 컴프리헨션

```
{표현식 for 표현식 in iterable객체}
```

제네레이터 컴프리헨션

```
(표현식 for 표현식 in interable객체)
```

>괄호로 되어있지만 튜플을 생성하지 않는다. (튜플은 컴프리헨션이 없다)

>제너레이터 객체는 순회 가능하며, 리스트 형태로 만들 수 있다.

---

# 함수

- 반복적인 작업을 하는 코드를 재사용이 가능하게 정의해 놓은 것.

```
def 함수명(매개변수[parameters]):
	동작
```

### 함수의 정의, 실행

```
# 실행 시 'call func'를 print하는 함수 정의
>>> def func():
...   print('call func')
... 

# 함수 자체는 function객체를 참조하는 변수
>>> func
<function func at 0x10dabf378>

# 함수를 실행시키기 위해 () 사용
>>> func()
call func
```

---

### 리턴값이 있는 함수 정의

```
>>> def return_true():
...   return True
... 
>>> return_true()
True
```

>함수의 결과로 Bool값을 갖는 데이터를 리턴하여 if문이나 while문의 조건으로 사용 가능하다.

---

### 매개변수를 사용하는 함수 정의

```
>>> def print_arguments(something):
...   print(something)
... 
>>> print_arguments('ABC')
ABC
```

### 매개변수(parameter)와 인자(argument)의 차이

- 함수 내부에서 함수에게 전달되어 온 변수는 매개변수라 부르며, 함수를 호출할 때 전달하는 변수는 인자로 부른다.

```
# 함수 정의때는 매개변수
def func(매개변수1, 매개변수2):
  ...
  
# 함수 호출시에는 인자
>>> func(인자1, 인자2)
```

### 리턴값이 없을 경우

- 함수에서 리턴해 주는 값이 없을 경우, 아무것도 없다는 뜻을 가진 `None`객체를 얻는다.

### 위치 인자(Positional arguments)

- 매개변수의 순서대로 인자를 전달하여 사용하는 경우

```
>>> def student(name, age, gender):
...   return {'name': name, 'age': age, 'gender': gender}
... 
>>> student('hanyeong.lee', 30, 'male')
{'name': 'hanyeong.lee', 'age': 30, 'gender': 'male'}
```

### 키워드 인자(Keyword arguments)

- 매개변수의 이름을 지정하여 인자로 전달하여 사용하는 경우

```
>>> student(age=30, name='hanyeong.lee', gender='male')
{'name': 'hanyeong.lee', 'age': 30, 'gender': 'male'}
```

>위치인자와 키워드인자를 동시에 쓴다면, 위치인자가 먼저 와야 한다

### 기본 매개변수값 지정

- 인자가 제공되지 않을 경우, 기본 매개변수로 사용할 값을 지정할 수 있다.

```
>>> def student(name, age, gender, cls='WPS'):
...   return {'name': name, 'age': age, 'gender': gender, 'class': cls}
... 
>>> student('hanyeong.lee', 30, 'male')
{'name': 'hanyeong.lee', 'age': 30, 'gender': 'male', 'class': 'WPS'}
기본 매개변수값의 정의 시점

기본 매개변수값은 함수가 실행될 때 마다 계산되지 않고, 함수가 정의되는 시점에 계산되어 계속해서 사용된다.
>>> def return_list(value, result=[]):
...   result.append(value)
...   return result
... 
>>> return_list('apple')
['apple']
>>> return_list('banana')
['apple', 'banana']
```

- 함수가 실행되는 시점에 기본 매개변수값을 계산하기 위해, 아래와 같이 바꿔준다.

```
>>> def return_list(value, result=None):
...   if result is None:
...     result = []
...   result.append(value)
...   return result
... 
>>> return_list('apple')
['apple']
>>> return_list('banana')
['banana']
```

### 위치인자 묶음

함수에 위치인자로 주어진 변수들의 묶음은 `*매개변수명`으로 사용할 수 있다.
관용적으로 `*args`를 사용한다.

```
def print_args(*args):
  print(args)
```

### 키워드인자 묶음

- 함수에 키워드인자로 주어진 변수들의 묶음은 `**매개변수명`으로 사용할 수 있다. 관용적으로 `**kwargs`를 사용한다.

```
def print_kwargs(**kwargs):
  print(kwargs)
```

---

### docstring


- 함수를 정의한 문서 역할을 한다.
- 함수 정의 후, 몸체의 시작부분에 문자열로 작성하며, 여러줄로도 작성 가능하다.

```
>>> def print_args(*args):
...   'Print positional arguments'
...   print(args)
... 
>>> help(print_args)
```

### 함수를 인자로 전달

- 파이썬에서는 함수 역시 다른 객체와 동등하게 취급되므로, 함수에서 인자로 함수를 전달, 실행, 리턴하는 형태로 프로그래밍이 가능하다.

### 내부 함수

- 함수 안에서 또 다른 함수를 정의해 사용할 수 있다.

### 스코프(영역)

- 파이썬에서는 코드 작성 시, 각 함수마다 독립적인 스코프(영역)을 가진다.
- 메인 프로그램(현재 동작하는 프로그램의 최상위 위치)의 영역은 전역 영역(Global Scope)라고 하며, 전역 스코프 내부에서 독립적인 영역을 갖고 있는 경우에는 지역 영역(Local Scope)라고 부른다.

아래 코드의 경우, `show_global_champion`함수 내부의 영역은 별개의 로컬 스코프를 가지며, `champion`변수는 전역 영역의 것을 가져와 출력하는것을 볼 수 있다.

```
champion = 'Lux'

def show_global_champion():
    print('show_global_champion : {}'.format(champion))

show_global_champion()
print('print champion : {}'.format(champion))
```

위 코드를 아래와 같이 변경 후, 실행 해 본다.

```
champion = 'Lux'

def show_global_champion():
    print('show_global_champion : {}'.format(champion))

def change_global_champion():
    print('before change_global_champion : {}'.format(champion))
    champion = 'Ahri'
    print('after change_global_champion : {}'.format(champion))

show_global_champion()
change_global_champion()
```

>`change_global_champion`함수에서 오류가 발생한다.

>첫 번째 코드에서는 `champion`변수가 함수의 로컬 스코프에 존재하지 않기 때문에 글로블 스코프에서 해당 변수를 찾아 출력하였으나, 이번 코드에서는 내부에 또다른 `champion`변수가 존재하기 때문에 할당하기 전인 변수를 사용한 것으로 판단하여 프로그램에서 오류를 발생시킨다.

>이름이 같은 두 변수가 다른 객체임을 내장함수 `id`를 사용해 확인해보자.

>또한, 각 영역에 해당하는 데이터들은 `locals()`함수를 사용해 확인 할 수 있으며, 전역 영역의 데이터들은 `globals()`함수를 사용한다.