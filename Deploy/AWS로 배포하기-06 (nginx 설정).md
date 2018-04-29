# AWS로 배포하기 (Nginx 관련 설정)

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