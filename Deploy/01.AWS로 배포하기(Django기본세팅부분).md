# 01. AWS로 간단하게 배포하기(Django 기본 세팅 부분) 

(https://github.com/thenry0401/deploy_ec2_practice)

- 폴더 하나를 만들어주고 가상환경을 만든다

```
>>> take deploy_ec2

>>> pyenv virtualenv 3.6.0 deploy_ec2

>>> pyenv local deploy_ec2

>>> pip install django
```

- **.gitignore** 파일을 생성한다

- django 프로젝트 생성한다

```
>>> django-admin startproject deploy_ec2
```

- deploy_ec2 폴더를 django_app로 바꾼다 (**deploy_ec2/django_app**)

- django_app 폴더 아래의 deploy_ec2 폴더를 config로 바꾼다 (**deploy_ec2/django_app/config**)


- pycharm으로 열고 **django_app**폴더를 source_root로 설정해주고 interpreter도 설정해준다

	- interpreter 주소 예시	
	`home/thenry/.pyenv/versions/deploy_ec2/bin/python`
	
- **git init** 해주고 첫 커밋을 한다

- **django_app** 폴더안에서 `./manage.py runserver` 로 서버가 잘 열리는지 확인한다

---

- 배포하기 전에 중요한 정보들이 올라가는 걸 방지하기 위한 체크리스트를 확인한다

	- `https://docs.djangoproject.com/en/2.0/howto/deployment/checklist/`
	
- 그중에서 먼저 SECRET_KEY를 숨기기 위해서 **deploy_ec2** 폴더 바로 아래에 **.config_secret** 폴더를 생성한다 (**.gitignore** 에 추가해준다)

	- 위 폴더 아래에 `settings_common.json` 파일을 생성한다
	(배포환경과 로컬환경 두개 모두에서 사용할 비밀값들 저장)
	
	- 내용 작성
	
```
{
  "django": {
    "secret_key": "시크릿 키 내용"
  }
}
```

---
	
- allowed_host 도 숨기기 위해 두개의 파일을 생성한다

	- `settings_deploy.json` (배포)
	- `settings_debug.json` (로컬)
	

- settings 파일을 분할하기 위해 config 폴더 아래에 settings 파이썬 패키지를 생성한다

- settings 패키지 아래에 base.py 를 생성후 settings 파일의 모든 내용을 옮겨주고 settings 파일은 지워준다

- **base.py** 파일안에 .config_secret 폴더의 경로를 지정해준다

```
ROOT_DIR = os.path.dirname(BASE_DIR)

# .config_secret 및 하위 파일 경로
CONFIG_SECRET_DIR = os.path.join(ROOT_DIR, '.config_secret')
CONFIG_SECRET_COMMON_FILE = os.path.join(CONFIG_SECRET_DIR, 'settings_common.json')

```

- config_secret 변수에 CONFIG_SECRET_COMMON_FILE 경로의 파일 읽은 값을 할당한다 (**base.py** 내 작성)

```
# f = open(CONFIG_SECRET_COMMON_FILE)
# config_secret_string = f.read()
# config_secret = json.loads(config_secret_string)
# f.close()
config_secret_common = json.loads(open(CONFIG_SECRET_COMMON_FILE).read())

주석과 밑에 코드가 같은 의미이다
``` 

- secret_key 값을 가져온다 **base.py** 내 작성)

```
SECRET_KEY = config_secret_common['django']['secret_key']
```

- 런서버를 한번 돌려서 확인한다

```
>>> ./manage.py runserver --settings=config.settings.base

settings 파일이 경로가 다르기 때문에 직접 지정해준것임
```

- settings 패키지 아래에 **debug.py** 를 생성한다
	
	- **base.py** 에서 allowed_hosts와 debug 부분을 삭제하고 **debug.py**에 설정해준다

```
debug.py

from .base import *

config_secret_debug = json.loads(open(CONFIG_SECRET_DEBUG_FILE).read())

DEBUG =True

ALLOWED_HOSTS = config_secret_debug['django']['allowed_hosts']
```

> 이제 base로는 런서버가 안되고 `./manage.py runserver --settings=config.settings.debug` 로 런서버해야한다

---

README.md 파일 작성한다

---

