# AWS로 배포하기 (uWSGI 관련 설정)


#### WSGI 란?

- EC2 <-> WSGI(HTTP:8000) <-> Web application

- EC2 <-> (runserver:8000) <-> Web application
	
	- **WSGI**는 **EC2**에서 받은 요청을 **Web application** 으로 요청을 전달한다

	- 이전에는 runserver로  **EC2**에서 **Web application**로 바로 요청을 보냈었다

---

#### 웹 서버 관리용 유저 생성

```
sudo adduser nginx
```

#### uWSGI설치

```
(virtualenv 환경 내부에서)
pip install uwsgi
```

#### uWSGI 정상 동작 확인

```
uwsgi --http :8080 --home (virtualenv경로) --chdir (django프로젝트 경로) -w (프로젝트명).wsgi
```

- uwsgi 명령어 설명

	- --http : http로 오는 요청을 받는다
	- :8000 : 8000번 포트로 오는 요청을 받는다
	- --home : uwsgi가 돌아가는 환경, 여기서는 파이썬 홈
	- (virtualenv경로) : 가상환경의 경로, 여기가 파이썬 홈이다
	- --chdir : uwsgi가 실행할 파이썬 어플리케이션을 찾기 쉽게 하기위해서 현재경로를 바꿔주는것이다
	- -w (프로젝트명).wsgi: 어떤 wsgi 설정 파일을 가지고 실행을 할것인지. 모듈을 찾는다

ex) pyenv virtualenv이름이 mysite-env, django프로젝트가 /srv/mysite/django_app, 프로젝트명이 mysite일 경우

```
uwsgi --http :8080 --home ~/.pyenv/versions/mysite-env --chdir /srv/mysite/django_app -w mysite.wsgi
```

실행 후 <ec2도메인>:8080으로 접속하여 요청을 잘 받는지 확인

- SECRET_KEY가 없다고 에러가 뜨는것은 Django의 settings 모듈을 잘못 지정했기 때문이다. wsgi.py 파일에서 설정해준다

```python
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

>>>>>>

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings.deploy")

서버에서 실행할땐 deploy, 로컬에서 실행할땐 debug가 되야한다
```

---


#### wsgi 모듈 작성

- config 폴더 안에 wsgi_modules 파이썬 패키지 작성 

- config > wsgi_modules > debug.py, deploy.py

- 기존 wsgi.py 는 건들지 않는

```
(deploy.py)

import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings.deploy")
application = get_wsgi_application()

--------------------------------------

(debug.py)

import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings.debug")
application = get_wsgi_application()
```

- 서버에서 실행시

```
uwsgi --http :8080 --home (virtualenv경로) --chdir (django프로젝트 경로) -w (프로젝트명).wsgi.deploy
```

- 로컬에서 실행시

```
uwsgi --http :8080 --home (virtualenv경로) --chdir (django프로젝트 경로) -w (프로젝트명).wsgi.deploy
```

#### uwsgi의 긴 명령어를 대체하기 위한 설정파일 작성

- deploy_ec2/.config_secret/uwsgi/debug.ini

- uwsgi 명령어 뒤에 써줬던 설정들을 그대로 써준다

```
[uwsgi]
home = <가상환경 경로>
chdir = <프로젝트 경로>
module = config.wsgi_modules.debug
http = :8000
```

#### wsgi.py와 wsgi_modules 둘다 있는 이유

- uwsgi를 실행할땐 직접 wsgi 모듈을 설정해줄수 있다 runserver 일때는 설정해줄수가 없고 wsgi.py를 무조건 참조한다 따라서 두가지를 같이 작성해준



---

#### uWSGI 사이트 파일 작성

```
sudo mkdir /etc/uwsgi
sudo mkdir /etc/uwsgi/sites
sudo vi /etc/uwsgi/sites/mysite.ini

[uwsgi]
chdir = /srv/mysite/django_app # Django application folder
module = mysite.wsgi:application # Django project name.wsgi
home = /home/ubuntu/.pyenv/versions/mysite # VirtualEnv location

uid = nginx
gid = nginx

socket = /tmp/mysite.sock
chmod-socket = 666
chown-socket = nginx:nginx

enable-threads = true
master = true
pidfile = /tmp/mysite.pid
```

#### uWSGI site파일로 정상 동작 확인

```
uwsgi --http :8080 -i /etc/uwsgi/sites/mysite.ini
```

sudo로 root권한으로 실행

```
sudo /home/ubuntu/.pyenv/versions/mysite/bin/uwsgi --http :8080 -i /etc/uwsgi/sites/mysite.ini
```

#### uWSGI 서비스 설정파일 작성

```
sudo vi /etc/systemd/system/uwsgi.service

[Unit]
Description=uWSGI Emperor service
After=syslog.target

[Service]
ExecPre=/bin/sh -c 'mkdir -p /run/uwsgi; chown nginx:nginx /run/uwsgi'
ExecStart=/home/ubuntu/.pyenv/versions/mysite/bin/uwsgi --uid nginx --gid nginx --master --emperor /etc/uwsgi/sites

Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

#### 리부팅 시 자동으로 실행되도록 설정

```
sudo systemctl enable uwsgi
```


