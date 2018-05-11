# AWS로 배포하기 (Nginx 관련 설정)


#### 파이참에서 nginx support 플러그인 설치

`Settings > Plugin > nginx Support 설치`


#### Nginx의 역할

왼쪽에서 오른쪽으로 요청을 보낸다

- EC2 -> (http) -> Django

- EC2 -> (http) -> uWSGI -> (WSGI) -> Django

- EC2 -> (http) -> Nginx -> (uWSGI's UnixSocket) -> uWSGI -> (WSGI) -> Django 

	- Nginx와 uWSGI는 소켓통신을 한다
	
	- uWSGI가 받을 수 있는 소켓을 생성한다


#### Nginx 안정화 최신버전 사전세팅 및 설치(로컬과 서버 동일)

```
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx
nginx -v

nginx
# 오류가 뜨면 sudo nginx
```

> nginx를 위해 AWS 보안그룹에 80번 포트 추가

#### 포트 확인하는 법 & 죽이는 법

```
ps -ax | grep <찾고싶은 이름>

kill -9 <포트넘버>
```

#### user 생성 ( 이 이후로 모두 서버에서의 설정)

```
sudo appuser deploy
```

#### Nginx 동작 User 변경 및 서버이름 길이 변경

```
sudo vi /etc/nginx/nginx.conf

user deploy;

http {
	# Prepare EC2 public domain length
	server_names_hash_bucket_size 250;
}
```

#### Nginx 가상서버 설정 파일 작성

```
sudo vi /etc/nginx/sites-available/ec2

server {
    listen 80;
    server_name localhost; (서버에서는 ec2 퍼블릭 도메인으로 작성)
    charset utf-8;
    client_max_body_size 128M;


    location / {
        uwsgi_pass    unix:///tmp/mysite.sock;
        include       uwsgi_params;
    }
}
```

#### uWSGI 사이트 파일 작성(로컬의 프로젝트 폴더)

```
<deploy_ec2/.config_secret/uwsgi/deploy.ini>

[uwsgi]
home = /home/ubuntu/.pyenv/versions/deploy_ec2 #가상환경 경로
chdir = /srv/deploy_ec2/django_app # 프로젝트 폴더 경로
module = mysite.wsgi_modules.deploy

uid = deploy # uid, gid는 uwsgi를 실행할 user id, group id
gid = deploy

socket = /tmp/ec2.sock # 사용할 소켓 파일의 경로 지정
chmod-socket = 666 # 소켓의 소유 권한
chown-socket = deploy:deploy # 소켓의 소유자 uid:gid

enable-threads = true # 성능관련 세팅
master = true # 성능관련 세팅
pidfile = /tmp/ec2.pid # 성능관련 세팅

vacuum = true # uwsgi 실행 종료 후 자동으로 ec2.pid와 ec2.sock 파일 삭제해주는 기능
```

> 작성후 서버에 덮어씌워준다

#### 서버에서 /tmp/ 폴더의 소유자 바꾸기

```
서버에서 /tmp/폴더의 소유자가 root로 되어있으면 접근이 안되기에 생성한 유저인 deploy 유저로 바꿔준다

sudo chown -R deploy:deploy /tmp/
```

#### uWSGI 실행되는지 확인

```
# /srv/deploy_ec2 폴더 내에서
# tmp폴더가 deploy유저권한인데 우리는 ubuntu유저로 실행을 하면 오류가 난다 그래서 deploy유저로써 실행하기 위해 아래와 같이 실행해야한다.

sudo -u deploy /home/ubuntu/.pyenv/versions/deploy_ec2/bin/uwsgi --ini .config_secret/uwsgi/deploy.ini
```

- 정상적으로 실행됐다면 /tmp 폴더안에 `ec2.pid`와 `ec2.sock`파일이 생성된다 

- 이제 소켓파일을 만들어서 Nginx의 요청을 받을 준비까지 된것이다

- 이제 Nginx와 연결을 시킬 차례

#### sites-enabled의 default파일 삭제(필요가 없어서 삭제함)

```
# 서버내에서 실행

sudo rm /etc/nginx/sites-enabled/default

```

#### 새롭게 설정파일 심볼릭 링크 생성 

```
# sites-enabled 폴더 내

sudo ln -s /etc/nginx/sites-available/ec2 . 
```

> ec2 연결에 관한 설정파일 링크를 생성해준것임

#### uWSGI, Nginx재시작

```
sudo systemctl restart uwsgi nginx
```

#### uWSGI 서비스 설정파일 작성

- 서버를 껏다켜도 자동으로 nginx를 실행하도록 한다

```
sudo vi /etc/systemd/system/uwsgi.service

[Unit]
Description=uWSGI Emperor service
After=syslog.target

[Service]
ExecPre=/bin/sh -c 'mkdir -p /run/uwsgi; chown deploy:deploy /run/uwsgi'
ExecStart=/home/ubuntu/.pyenv/versions/deploy_ec2/bin/uwsgi --uid deploy --gid deploy --master --ini /srv/deploy_ec2/.config_secret/uwsgi/deploy.ini

Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

### 오류 발생 시

#### systemctl restart시 오류 발생 시

```
(오류 발생한 서비스에 따라 아래 명령어 실행)
sudo systemctl status uwsgi.service
sudo systemctl status nginx.service
```







ec2.conf, nginx.conf, uwsgi.service 파일을 프로젝트내에서 작성한다












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