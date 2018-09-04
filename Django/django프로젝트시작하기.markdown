# Django 프로젝트 시작하기

###환경설정

1. 프로젝트를 위한 폴더 생성

2. pyenv 환경설정
```
pyenv virtualenv <파이썬 버전> <프로젝트 환경이름>
ex) pyenv virtualenv 3.5.3 django_env
```

3.  django 설치
`pip install django~=1.11.0`
> 특정 버전 설치시 '~=' 뒤에 버전 작성 

4. gitignore 생성

	- 프로젝트 폴더 아래에서 `vi .gitignore` 를 통해 파일을 생성한다
	
	- `gitignore.io` 사이트에서 설정에 필요한 것들을 추가후 생성해서 내용을 복사붙여넣는다 

5. git 생성
`git init`

6. 프로젝트를 생성한다
```
django-admin startproject config

```
> 그후 상위 config 폴더 이름을 django_app으로 바꿔준다

7. 앱을 생성한다 

```
django_app 폴더 내에서 다음과 같이 입력

./manage.py startapp <앱 이름>
```

8. pycharm 프로그램을 실행한다

프로젝트 폴더 가장 상단에서 `charm .` 입력

9. pycharm 세팅을 한다

	- django_app 폴더를 **sources root**로 지정한다
		- pycharm 프로그램 왼쪽 화면의 폴더 구조에서 맨위의 django_app을 
		우클릭 - Mark directory as - sources root 
	- Project interpreter 설정한다 
		- pycharm 메뉴바의 file - settings - project페 interpreter 클릭
		- 오른쪽 상단의 톱니바퀴 버튼 - Add Local 클릭
		- /home/thenry/.pyenv/versions/<프로젝트 가상환경 이름>/bin/python 선택
		- 창 밑의 Apply로 적용하고 끝낸다

9.  앱을 생성한 뒤 settings.py 내부의 INSTALLED_APPS에 추가해야 한다

```
django_app/<프로젝트 이름>/settings.py

INSTALLED_APPS = [
	'blog',
]
```

<br><br><br>
#### 생성된 모델을 장고 관리자 페이지에 추가하는 법

- django_app/<앱 이름>/admin.py 에서 다음과 같이 입력

```
admin.site.register(모델 이름)
```

#### 장고 관리자 계정 생성하는 법

- django_app 폴더 내부에서 다음과 같이 입력

```
./manage.py createsuperuser
```

> email은 입력하지 않아도 됨

#### 장고 사이트 언어와 시간대 바꾸는 법

- django_app/<프로젝트 이름>/settings. py 에서 바꿔준다

```
LANGUAGE_CODE = 'Ko-Kr'

TIME_ZONE = 'Asia/Seoul'
```


