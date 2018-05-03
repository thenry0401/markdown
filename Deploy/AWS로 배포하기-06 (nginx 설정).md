# AWS로 배포하기 (Nginx 관련 설정)

**우분투는 로컬과 서버에서의 세팅이 똑같다**

#### Nginx의 역할

왼쪽에서 오른쪽으로 요청을 보낸다

- EC2 -> (http) -> Django

- EC2 -> (http) -> uWSGI -> (WSGI) -> Django

- EC2 -> (http) -> Nginx -> (uWSGI's UnixSocket) -> uWSGI -> (WSGI) -> Django

	- Nginx와 uWSGI는 소켓통신을 한다
	
	- uWSGI가 받을 수 있는 소켓을 생성한다


#### Nginx 안정화 최신버전 사전세팅 및 설치

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

#### user 생성

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

#### uWSGI 사이트 파일 작성

```
[uwsgi]
home = /home/ubuntu/.pyenv/versions/deploy_ec2 #가상환경 경로
chdir = /srv/deploy_ec2/django_app # 프로젝트 폴더 경로
module = mysite.wsgi_modules.deploy

uid = deploy # uid, gid는 uwsgi를 실행할 user id, group id
gid = deploy

socket = /tmp/ec2.sock # 사용할 소켓 파일의 경로
chmod-socket = 666 # 소켓의 소유 권한
chown-socket = deploy:deploy # 소켓의 소유자 uid:gid

enable-threads = true # 성능관련 세팅
master = true # 성능관련 세팅
pidfile = /tmp/ec2.pid # 성능관련 세팅
```

> 작성후 서버에 덮어씌워준다

#### 서버에서 /tmp/ 폴더의 소유자 바꾸기

```
서버에서 /tmp/폴더의 소유자가 root로 되어있으면 접근이 안되기에 생성한 유저인 deploy 유저로 바꿔준다

sudo chown -R deploy:deploy /tmp/
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