# pgAdmin 설치법 (Ubuntu 16.04)

### 다운로드할 주소 가져오기

- pgAdmin.org 에서 Download - Python Wheel

	- https://www.pgadmin.org/download/pgadmin-4-python-wheel/

	- pip로 설치하기 위한 방법

- 가장 최신 버전을 클릭해서 .whl 로 끝나는 파일의 주소를 카피한다

```
링크 예:
https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v2.1/pip/pgadmin4-2.1-py2.py3-none-any.whl
```

---

### 설치

- ` /home/thenry `위치에 `pgadmin` 이라는 폴더를 만든다

- 가상환경에서 pip 설치를 할것이기 때문에 가상환경을 설정한다

	- python 버전은 `python versions`로 확인해서 그에 맞게 설정

```
>>> pyenv virtualenv 3.6.0 pgadmin

>>> pyenv local pgadmin

``` 

- wget으로 pgadmin 다운로드 하기

```
wget https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v2.1/pip/pgadmin4-2.1-py2.py3-none-any.whl
```

- pip로 설치하기

```
pip install pgadmin4-2.1-py2.py3-none-any.whl
```

---

### 실행

- 가상환경 내에서 명령어로 실행

```
python ~/.pyenv/versions/pgadmin/lib/python3.6/site-packages/pgadmin4/pgAdmin4.py
```

- 이메일 계정과 비밀번호를 설정 후  `http://127.0.0.1:5050` 로 접속하면 된다

---

### 발생할 수 있는 에러

- `PermissionError: [Errno 13] Permission denied: '/var/lib/pgadmin'` 


```
위와 같은 에러 발생시

>>>sudo mkdir "/var/log/pgadmin"
>>>sudo chmod a+wrx "/var/log/pgadmin"

>>>sudo mkdir "/var/lib/pgadmin"
>>>sudo chmod a+wrx "/var/lib/pgadmin"

위와 같이 해결하였다
```

