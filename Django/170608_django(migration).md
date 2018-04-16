###migration 되돌리기

1. DB 내용도 되돌려야 하기때문에 먼저 DB파일을 삭제한다

1. migrations 폴더에가서 되돌리고 싶은 시점을 찾는다

1. `./manage.py migrate [app이름] [되돌리고 싶은 시점의 migration 파일이름]` 명령어를 입력한다
>migration 파일이름 입력할떄 `.py` 는 뺴고 입력한다

1. `./manage.py showmigrations` 으로 확인하고
 
1. `./manage.py migrate` 한후 데이터베이스에서 테이블들이 사라졌는지 확인한다

1. 필요없게된 migration 파일들을 삭제하고 DB파일도 삭제한다

1. 그러고 나서 makemigrations와 migrate 해줌

---

###같은 이름의 테이블을 또 만들때 migrate 하는법

`./manage.py mygrate --fake`
또는
`./manage.py migrate [app이름] [적용시킬 가장 마지막 migration파일 이름] --fake`
위에꺼는 가장 최신것을 적용, 아래꺼는 직접 지정하여 적용
이렇게만 이름이 겹치는 테이블이지만 migrate를 할수잇게한다
