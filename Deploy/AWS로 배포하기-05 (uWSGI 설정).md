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


## Nginx 관련 설정

#### Nginx 안정화 최신버전 사전세팅 및 설치

```
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx
nginx -v
```

#### Nginx 동작 User 변경

```
sudo vi /etc/nginx/nginx.conf

user nginx;
```

#### Nginx 가상서버 설정 파일 작성

```
sudo vi /etc/nginx/sites-available/mysite

server {
    listen 80;
    server_name localhost;
    charset utf-8;
    client_max_body_size 128M;


    location / {
        uwsgi_pass    unix:///tmp/mysite.sock;
        include       uwsgi_params;
    }
}
```

#### 설정파일 심볼릭 링크 생성

```
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/mysite
```

#### sites-enabled의 default파일 삭제

```
sudo rm /etc/nginx/sites-enabled/default
```

> nginx.conf파일에 어떤 폴더에 있는 설정을 가져와서 실행할 지 적혀있음


#### uWSGI, Nginx재시작

```
sudo systemctl restart uwsgi nginx
```

-

### 오류 발생 시

#### systemctl restart시 오류 발생 시

```
(오류 발생한 서비스에 따라 아래 명령어 실행)
sudo systemctl status uwsgi.service
sudo systemctl status nginx.service
```

#### 502 Bad Gateway

**Nginx log파일 확인**  
```
cat /var/log/nginx/error.log
```

#### Nginx log파일에서 sock파일 접근 불가시

**socket파일 권한 소유자 확인**

```
cd /tmp
ls -al
-rw-r--r--  1 nginx  nginx     6 Nov  8 06:58 mysite.pid
srw-rw-rw-  1 nginx  nginx     0 Nov  8 06:58 mysite.sock

nginx가 소유자가 아닐 경우, 
sudo rm mysite.pid mysite.sock으로 삭제 후 서비스 재시작
```

## 서버 설정

### Media

> 본인의 media폴더 위치를 확인

`sudo vi /etc/nginx/sites-available/app`

```
location /media/ {
	alias /srv/app/media/;
}
```

**이미지 업로드시 Permission Denied발생 할 경우**  
`sudo chmod 777 media`로 `media`폴더의 권한을 변경, `media`폴더 하위 폴더 모두 삭제 후 다시 실행


### Static

> 본인의 static_root폴더 위치를 확인

`sudo vi /etc/nginx/sites-available/app`

```
location /static/ {
	alias /srv/app/static_root/;
}
```

### 데이터베이스

> macOS에서는 `sudo -u postgres` 입력 불필요

**유저생성**  
`sudo -u postgres createuser -s -P <username>`

**유저삭제**  
`sudo -u postgres dropuser <username>`

**삭제**  
`sudo -u postgres dropdb <db name>`

**생성**  
`sudo -u postgres createdb <db name> owner=<owner name>`

-

#### AWS Secutiriy Groups 80 Port추가

Security Groups -> Inbound -> Edit -> HTTP


## Cloudflare

#### ALLOWED_HOSTS 추가



## SSL

> [Nginx에 SSL적용](https://haandol.wordpress.com/2014/03/12/nginx-ssl-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0startssl-com%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC/)  
> [Cloudflare에 Custom SSL적용](https://support.cloudflare.com/hc/en-us/articles/200170466-How-do-I-upload-a-custom-SSL-certificate-Business-or-Enterprise-only-)


## S3 연결

### boto3를 이용해서 S3 Bucket생성

`boto3`

```python
pip install boto3
```

```
>>> import boto3
>>> session = boto3.Session(profile_name='fc-4th')
>>> client = session.client('s3')
>>> client.create_bucket(Bucket='fc-4th-test', CreateBucketConfiguration={'LocationConstraint': 'ap-northeast-2'})
{'Location': 'http://fc-4th-test.s3.amazonaws.com/', 'ResponseMetadata': {'RequestId': '0B679A5EBF1FCF19', 'HostId': 'qLvVYj0n74HZKnA46m+ILCabPs71dt0GEqNFllrRguaBSbvQ2MpQ4aWhOT/PjHFzVo8nE1/H4cw=', 'HTTPStatusCode': 200, 'RetryAttempts': 0, 'HTTPHeaders': {'content-length': '0', 'location': 'http://fc-4th-test.s3.amazonaws.com/', 'date': 'Thu, 09 Mar 2017 01:41:17 GMT', 'x-amz-id-2': 'qLvVYj0n74HZKnA46m+ILCabPs71dt0GEqNFllrRguaBSbvQ2MpQ4aWhOT/PjHFzVo8nE1/H4cw=', 'x-amz-request-id': '0B679A5EBF1FCF19', 'server': 'AmazonS3'}}}
>>> 
```