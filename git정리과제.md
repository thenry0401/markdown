# Git 도구 - Stashing 과 Cleaning

### git stash

- 아직 완료하지 않은 일을 커밋하지 않고 나중에 다시 돌아와서 작업을 해야할 때 사용한다
아직 끝내지 않은 수정사항을 스택에 잠시 저장했다가 나중에 다시 적용할 수 있다

- Git은 Stash를 적용할 때 Staged 상태였던 파일을 자동으로 다시 Staged 상태로 만들어 주지 않는다. 그래서 git stash apply 명령을 실행할 때 --index 옵션을 주어 Staged 상태까지 적용한다. 그래야 원래 작업하던 상태로 돌아올 수 있다.

---

### 하던 일을 Stash 하기

$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

```
    modified:   index.html
    
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   lib/simplegit.rb
```

이제 브랜치를 변경해 보자. 아직 작업 중인 파일은 커밋할 게 아니라서 모두 Stash 한다. `git stash`나 `git stash save`를 실행하면 스택에 새로운 Stash가 만들어진다.

```
$ git stash
Saved working directory and index state \
  "WIP on master: 049d078 added the index file"
HEAD is now at 049d078 added the index file
(To restore them type "git stash apply")
```

대신 워킹 디렉토리는 깨끗해졌다.

```
$ git status
# On branch master
nothing to commit, working directory clean
```

이제 아무 브랜치나 골라서 쉽게 바꿀 수 있다. 수정하던 것을 스택에 저장했다. 아래와 같이 `git stash list`를 사용하여 저장한 Stash를 확인한다.

```
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
```

Stash 두 개는 원래 있었다. 그래서 현재 총 세 개의 Stash를 사용할 수 있다. 이제 git stash apply`를 사용하여 Stash를 다시 적용할 수 있다. `git stash 명령을 실행하면 Stash를 다시 적용하는 방법도 알려줘서 편리하다. `git stash apply stash@{2}`처럼 Stash 이름을 입력하면 골라서 적용할 수 있다. 이름이 없으면 Git은 가장 최근의 Stash를 적용한다.

```
$ git stash apply
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

 modified:   index.html
 modified:   lib/simplegit.rb

no changes added to commit (use "git add" and/or "git commit -a")
```

Git은 Stash에 저장할 때 수정했던 파일들을 복원해준다. 복원할 때의 워킹 디렉토리는 Stash 할 때의 그 브랜치이고 워킹 디렉토리도 깨끗한 상태였다. 하지만, 꼭 깨끗한 워킹 디렉토리나 Stash 할 때와 같은 브랜치에 적용해야 하는 것은 아니다. 어떤 브랜치에서 Stash 하고 다른 브랜치로 옮기고서 거기에 Stash를 복원할 수 있다. 그리고 꼭 워킹 디렉토리가 깨끗한 상태일 필요도 없다. 워킹 디렉토리에 수정하고 커밋하지 않은 파일들이 있을 때도 Stash를 적용할 수 있다. 만약 충돌이 있으면 알려준다.

Git은 Stash를 적용할 때 Staged 상태였던 파일을 자동으로 다시 Staged 상태로 만들어 주지 않는다. 그래서 git stash apply 명령을 실행할 때 --index 옵션을 주어 Staged 상태까지 적용한다. 그래야 원래 작업하던 상태로 돌아올 수 있다.

```
$ git stash apply --index
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

 modified:   index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

 modified:   lib/simplegit.rb
```

apply 옵션은 단순히 Stash를 적용하는 것뿐이다. Stash는 여전히 스택에 남아 있다. git stash drop 명령을 사용하여 해당 Stash를 제거한다.

```
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
$ git stash drop stash@{0}
Dropped stash@{0} (364e91f3f268f0900bc3ee613f9f733e82aaed43)
```

그리고 `git stash pop`이라는 명령도 있는데 이 명령은 Stash를 적용하고 나서 바로 스택에서 제거해준다.

Stash를 만드는 새로운 방법
Stash를 만드는 방법은 여러 가지다. 주로 사용하는 옵션으로 stash save 명령과 같이 쓰는 `--keep-index`이다. 이 옵션을 이용하면 이미 Staging Area에 들어 있는 파일을 Stash 하지 않는다.

많은 파일을 변경했지만 몇몇 파일만 커밋하고 나머지 파일은 나중에 처리하고 싶을 때 유용하다.

```
$ git status -s
M  index.html
 M lib/simplegit.rb

$ git stash --keep-index
Saved working directory and index state WIP on master: 1b65b17 added the index file
HEAD is now at 1b65b17 added the index file

$ git status -s
M  index.html
```

추적하지 않는 파일과 추적 중인 파일을 같이 Stash 하는 일도 꽤 빈번하다. 기본적으로 git stash`는 추적 중인 파일만 저장한다. 추적 중이지 않은 파일을 같이 저장하려면 Stash 명령을 사용할 때 `--include-untracked`나 `-u 옵션을 붙여준다.

```
$ git status -s
M  index.html
 M lib/simplegit.rb
?? new-file.txt

$ git stash -u
Saved working directory and index state WIP on master: 1b65b17 added the index file
HEAD is now at 1b65b17 added the index file

$ git status -s
$
```

끝으로 --patch 옵션을 붙이면 Git은 수정된 모든 사항을 저장하지 않는다. 대신 대화형 프롬프트가 뜨며 변경된 데이터 중 저장할 것과 저장하지 않을 것을 지정할 수 있다.

```
$ git stash --patch
diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index 66d332e..8bb5674 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -16,6 +16,10 @@ class SimpleGit
         return `#{git_cmd} 2>&1`.chomp
       end
     end
+
+    def show(treeish = 'master')
+      command("git show #{treeish}")
+    end

 end
 test
Stash this hunk [y,n,q,a,d,/,e,?]? y

Saved working directory and index state WIP on master: 1b65b17 added the index file
```

Stash를 적용한 브랜치 만들기
보통 Stash에 저장하면 한동안 그대로 유지한 채로 그 브랜치에서 계속 새로운 일을 한다. 그러면 이제 저장한 Stash를 적용하는 것이 문제가 된다. 수정한 파일에 Stash를 적용하면 충돌이 일어날 수도 있고 그러면 또 충돌을 해결해야 한다. 필요한 것은 Stash 한 것을 쉽게 다시 테스트하는 것이다. git stash branch 명령을 실행하면 Stash 할 당시의 커밋을 Checkout 한 후 새로운 브랜치를 만들고 여기에 적용한다. 이 모든 것이 성공하면 Stash를 삭제한다.

```
$ git stash branch testchanges
M  index.html
M  lib/simplegit.rb
Switched to a new branch 'testchanges'
On branch testchanges
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

 modified:   index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

 modified:   lib/simplegit.rb

Dropped refs/stash@{0} (29d385a81d163dfd45a452a2ce816487a6b8b014)
```

이 명령은 브랜치를 새로 만들고 Stash를 복원해주는 매우 편리한 도구다.

워킹 디렉토리 청소하기
작업하고 있던 파일을 Stash 하지 않고 단순히 그 파일들을 치워버리고 싶을 때가 있다. git clean 명령이 그 일을 한다.

보통은 Merge나 외부 도구가 만들어낸 파일을 지우거나 이전 빌드 작업으로 생성된 각종 파일을 지우는 데 필요하다.

이 명령을 사용할 때는 신중해야 한다. 이 명령을 사용하면 워킹 디렉토리 안의 추적하고 있지 않은 모든 파일이 지워지기 때문이다. 명령을 실행하고 나서 후회해도 소용없다. 지워진 파일은 돌아오지 않는다. git stash –all 명령을 이용하면 지우는 건 똑같지만, 먼저 모든 파일을 Stash 하므로 좀 더 안전하다.

워킹 디렉토리의 불필요한 파일들을 전부 지우려면 git clean`을 사용한다. 추적 중이지 않은 모든 정보를 워킹 디렉토리에서 지우고 싶다면 `git clean -f -d 명령을 사용하자. 이 명령은 하위 디렉토리까지 모두 지워버린다. -f 옵션은 강제(force)의 의미이며 "진짜로 그냥 해라"라는 뜻이다.

이 명령을 실행했을 때 어떤 일이 일어날지 미리 보고 싶다면 -n 옵션을 사용한다. -n 옵션은 ‘`가상으로 실행해보고 어떤 파일들이 지워질지 알려달라’'라는 뜻이다.

```
$ git clean -d -n
Would remove test.o
Would remove tmp/
```

git clean 명령은 추적 중이지 않은 파일만 지우는 게 기본 동작이다. .gitignore`에 명시했거나 해서 무시되는 파일은 지우지 않는다. 무시된 파일까지 함께 지우려면 `-x 옵션이 필요하다. 그래서 .o 파일 같은 빌드 파일까지도 지울 수 있다.

```
$ git status -s
 M lib/simplegit.rb
?? build.TMP
?? tmp/

$ git clean -n -d
Would remove build.TMP
Would remove tmp/

$ git clean -n -d -x
Would remove build.TMP
Would remove test.o
Would remove tmp/
```

git clean`이 무슨 짓을 할지 확신이 안들 때는 항상 `-n 옵션을 붙여서 먼저 실행해보자. clean 명령을 대화형으로 실행하려면 -i 옵션을 붙이면 된다.

대화형으로 실행한 clean 명령의 모습은 아래와 같다.

```
$ git clean -x -i
Would remove the following items:
  build.TMP  test.o
*** Commands ***
    1: clean                2: filter by pattern    3: select by numbers    4: ask each             5: quit
    6: help
What now>
```

대화형으로 실행하면 파일마다 지우지 말지 결정하거나 특정 패턴으로 걸러서 지울 수도 있다.

---

# Git 도구 - 검색

### 검색

프로젝트가 크든 작든 함수의 정의나 함수가 호출되는 곳을 검색해야 하는 경우가 많다. 함수의 히스토리를 찾아보기도 한다. Git은 데이터베이스에 저장된 코드나 커밋에서 원하는 부분을 빠르고 쉽게 검색하는 도구가 여러 가지 있으며 앞으로 함께 살펴보자.

###Git Grep

Git의 grep 명령을 이용하면 커밋 트리의 내용이나 워킹 디렉토리의 내용을 문자열이나 정규표현식을 이용해 쉽게 찾을 수 있다. Git 소스를 예로 들어 명령을 어떻게 사용하는지 알아보자.

기본적으로 대상을 지정하지 않으면 워킹 디렉토리의 파일에서 찾는다. 명령을 실행할 때 -n 옵션을 추가하면 찾을 문자열이 위치한 라인 번호도 같이 출력한다.

```
$ git grep -n gmtime_r
compat/gmtime.c:3:#undef gmtime_r
compat/gmtime.c:8:      return git_gmtime_r(timep, &result);
compat/gmtime.c:11:struct tm *git_gmtime_r(const time_t *timep, struct tm *result)
compat/gmtime.c:16:     ret = gmtime_r(timep, result);
compat/mingw.c:606:struct tm *gmtime_r(const time_t *timep, struct tm *result)
compat/mingw.h:162:struct tm *gmtime_r(const time_t *timep, struct tm *result);
date.c:429:             if (gmtime_r(&now, &now_tm))
date.c:492:             if (gmtime_r(&time, tm)) {
git-compat-util.h:721:struct tm *git_gmtime_r(const time_t *, struct tm *);
git-compat-util.h:723:#define gmtime_r git_gmtime_r
```

grep 명령에서 쓸만한 몇 가지 옵션을 좀 더 살펴보자.

예를 들어 위의 결과 대신 어떤 파일에서 몇 개나 찾았는지만 알고 싶다면 `--count`옵션을 이용한다.

```
$ git grep --count gmtime_r
compat/gmtime.c:4
compat/mingw.c:1
compat/mingw.h:1
date.c:2
git-compat-util.h:2
```

매칭되는 라인이 있는 함수나 메서드를 찾고 싶다면 -p 옵션을 준다.

```
$ git grep -p gmtime_r *.c
date.c=static int match_multi_number(unsigned long num, char c, const char *date, char *end, struct tm *tm)
date.c:         if (gmtime_r(&now, &now_tm))
date.c=static int match_digit(const char *date, struct tm *tm, int *offset, int *tm_gmt)
date.c:         if (gmtime_r(&time, tm)) {
```

gmtime_r 함수를 date.c 파일에서 match_multi_number, match_digit 함수에서 호출하고 있다는 걸 확인할 수 있다.

--and 옵션을 이용해서 여러 단어가 한 라인에 동시에 나타나는 줄 찾기 같은 복잡한 조합으로 검색할 수 있다. 예를 들어 “LINK''나 ``BUF_MAX” 둘 중 하나를 포함한 상수 정의를 1.8.0 이전 버전의 Git 소스 코드에서 검색하는 것을 할 수 있다.

--break`와 `--heading 옵션을 붙여 더 읽기 쉬운 형태로 잘라서 출력할 수도 있다.

```
$ git grep --break --heading \
    -n -e '#define' --and \( -e LINK -e BUF_MAX \) v1.8.0
v1.8.0:builtin/index-pack.c
62:#define FLAG_LINK (1u<<20)

v1.8.0:cache.h
73:#define S_IFGITLINK  0160000
74:#define S_ISGITLINK(m)       (((m) & S_IFMT) == S_IFGITLINK)

v1.8.0:environment.c
54:#define OBJECT_CREATION_MODE OBJECT_CREATION_USES_HARDLINKS

v1.8.0:strbuf.c
326:#define STRBUF_MAXLINK (2*PATH_MAX)

v1.8.0:symlinks.c
53:#define FL_SYMLINK  (1 << 2)

v1.8.0:zlib.c
30:/* #define ZLIB_BUF_MAX ((uInt)-1) */
31:#define ZLIB_BUF_MAX ((uInt) 1024 * 1024 * 1024) /* 1GB */
```

git grep 명령은 grep 이나 ack 같은 일반적인 검색 도구보다 몇 가지 좋은 점이 있다. 우선 매우 빠르다. 또한, 워킹 디렉토리만이 아니라 Git 히스토리 내의 어떠한 정보라도 찾아낼 수 있다. 위의 예제에서 이전 버전의 소스에서도 특정 단어를 찾아낸 것을 볼 수 있다.

Git 로그 검색
어떤 변수가 어디에 있는지를 찾아보는 게 아니라, 히스토리에서 언제 추가되거나 변경됐는지 찾아볼 수도 있다. git log 명령을 이용하면 Diff 내용도 검색하여 어떤 커밋에서 찾고자 하는 내용을 추가했는지 찾을 수 있다.

ZLIB_BUF_MAX 라는 상수가 가장 처음 나타난 때를 찾는 문제라면 -S 옵션을 이용해 해당 문자열이 추가된 커밋과 없어진 커밋만 검색할 수 있다.

```
$ git log -SZLIB_BUF_MAX --oneline
e01503b zlib: allow feeding more than 4GB in one go
ef49a7a zlib: zlib can only process 4GB at a time
```

위 두 커밋의 변경 사항을 살펴보면 ef49a7a`에서 `ZLIB_BUF_MAX 상수가 처음 나오고 e01503b 에서는 변경된 것을 알 수 있다.

더 세세한 조건을 걸어 찾고 싶다면 로그를 검색할 때 -G 옵션으로 정규표현식을 써서 검색하면 된다.

라인 로그 검색
진짜 미친 듯이 좋은 로그 검색 도구가 또 있다. 라인 히스토리 검색이다. 비교적 최근에 추가된 기능이어서 잘 알려지진 않았지만, 진짜 좋다. git log`를 쓸 때 `-L 옵션을 붙이면 어떤 함수나 한 라인의 히스토리를 볼 수 있다.

예를 들어, zlib.c 파일에 있는 git_deflate_bound 함수의 모든 변경 사항들을 보길 원한다고 생각해보자. git log -L :git_deflate_bound:zlib.c 라고 명령 실행하면 된다. 이 명령을 실행하면 함수의 시작과 끝을 인식해서 함수에서 일어난 모든 히스토리를 함수가 처음 만들어진 때부터 Patch를 나열하여 보여준다.

```
$ git log -L :git_deflate_bound:zlib.c
commit ef49a7a0126d64359c974b4b3b71d7ad42ee3bca
Author: Junio C Hamano <gitster@pobox.com>
Date:   Fri Jun 10 11:52:15 2011 -0700

    zlib: zlib can only process 4GB at a time

diff --git a/zlib.c b/zlib.c
--- a/zlib.c
+++ b/zlib.c
@@ -85,5 +130,5 @@
-unsigned long git_deflate_bound(z_streamp strm, unsigned long size)
+unsigned long git_deflate_bound(git_zstream *strm, unsigned long size)
 {
-       return deflateBound(strm, size);
+       return deflateBound(&strm->z, size);
 }


commit 225a6f1068f71723a910e8565db4e252b3ca21fa
Author: Junio C Hamano <gitster@pobox.com>
Date:   Fri Jun 10 11:18:17 2011 -0700

    zlib: wrap deflateBound() too

diff --git a/zlib.c b/zlib.c
--- a/zlib.c
+++ b/zlib.c
@@ -81,0 +85,5 @@
+unsigned long git_deflate_bound(z_streamp strm, unsigned long size)
+{
+       return deflateBound(strm, size);
+}
+
```

Git이 함수의 처음과 끝을 인식하지 못할 때는 정규표현식으로 인식하게 할 수도 있다. git log -L '/unsigned long git_deflate_bound/',/^}/:zlib.c 명령으로 위와 같은 결과를 볼 수 있다. 한 라인의 히스토리만 검색할 수도 있고 여러 라인에 걸친 히스토리를 검색할 수도 있다.

---

# Git 도구 - 히스토리 단장하기

### 히스토리 단장하기

Git으로 일하다 보면 어떤 이유로든 커밋 히스토리를 수정해야 할 때가 있다. 결정을 나중으로 미룰 수 있던 것은 Git의 장점이다. Staging Area로 커밋할 파일을 고르는 일을 커밋하는 순간으로 미룰 수 있고 Stash 명령으로 하던 일을 미룰 수 있다. 게다가 이미 커밋해서 결정한 내용을 수정할 수 있다. 그리고 수정할 수 있는 것도 매우 다양하다. 커밋들의 순서도 변경할 수 있고 커밋 메시지와 커밋한 파일도 변경할 수 있다. 여러 개의 커밋을 하나로 합치거나 반대로 커밋 하나를 여러 개로 분리할 수도 있다. 아니면 커밋 전체를 삭제할 수도 있다. 하지만, 이 모든 것은 다른 사람과 코드를 공유하기 전에 해야 한다.

이 절에서는 사람들과 코드를 공유하기 전에 커밋 히스토리를 예쁘게 단장하는 방법에 대해서 설명한다.

마지막 커밋을 수정하기
히스토리를 단장하는 일 중에서는 마지막 커밋을 수정하는 것이 가장 자주 하는 일이다. 기본적으로 두 가지로 나눌 수 있는데 하나는 커밋 메시지를 수정하는 것이고 다른 하나는 파일 목록을 수정하는 것이다.

커밋 메시지를 수정하는 방법은 매우 간단하다.

```
$ git commit --amend
```

이 명령은 자동으로 텍스트 편집기를 실행시켜서 마지막 커밋 메시지를 열어준다. 여기에 메시지를 바꾸고 편집기를 닫으면 편집기는 바뀐 메시지로 마지막 커밋을 수정한다.

커밋하고 난 후 새로 만든 파일이나 수정한 파일을 가장 최근 커밋에 집어넣을 수 있다. 기본적으로 방법은 같다. 파일을 수정하고 git add 명령으로 Staging Area에 넣거나 git rm 명령으로 추적하는 파일 삭제한다. 그리고 git commit --amend 명령으로 커밋하면 된다. 이 명령은 현 Staging Area의 내용을 이용해서 수정한다.

이때 SHA-1 값이 바뀌기 때문에 과거의 커밋을 변경할 때 주의해야 한다. Rebase와 같이 이미 Push 한 커밋은 수정하면 안 된다.

커밋 메시지를 여러 개 수정하기
최근 커밋이 아니라 예전 커밋을 수정하려면 다른 도구가 필요하다. 히스토리 수정하기 위해 만들어진 도구는 없지만 rebase 명령을 이용하여 수정할 수 있다. 현재 작업하는 브랜치에서 각 커밋을 하나하나 수정하는 것이 아니라 어느 시점부터 HEAD까지의 커밋을 한 번에 Rebase 한다. 대화형 Rebase 도구를 사용하면 커밋을 처리할 때마다 잠시 멈춘다. 그러면 각 커밋의 메시지를 수정하거나 파일을 추가하고 변경하는 등의 일을 진행할 수 있다. git rebase 명령에 -i 옵션을 추가하면 대화형 모드로 Rebase 할 수 있다. 어떤 시점부터 HEAD까지 Rebase 할 것인지 인자로 넘기면 된다.

마지막 커밋 메시지 세 개를 모두 수정하거나 그 중 몇 개를 수정하는 시나리오를 살펴보자. `git rebase -i`의 인자로 편집하려는 마지막 커밋의 부모를 `HEAD~2^`나 `HEAD~3`로 해서 넘긴다. 마지막 세 개의 커밋을 수정하는 것이기 때문에 `~3`이 좀 더 기억하기 쉽다. 그렇지만, 실질적으로 가리키게 되는 것은 수정하려는 커밋의 부모인 네 번째 이전 커밋이다.

```
$ git rebase -i HEAD~3
```

이 명령은 Rebase 하는 것이기 때문에 메시지의 수정 여부에 관계없이 HEAD~3..HEAD 범위에 있는 모든 커밋을 수정한다. 다시 강조하지만 이미 중앙서버에 Push 한 커밋은 절대 고치지 말아야 한다. Push 한 커밋을 Rebase 하면 결국 같은 내용을 두 번 Push 하는 것이기 때문에 다른 개발자들이 혼란스러워 할 것이다.

실행하면 Git은 수정하려는 커밋 목록이 첨부된 스크립트를 텍스트 편집기로 열어준다.

```
pick f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file

# Rebase 710f0f8..a5f4a0d onto 710f0f8
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

이 커밋은 모두 log 명령과는 정반대의 순서로 나열된다. log 명령을 실행하면 아래와 같은 결과를 볼 수 있다.

$ git log --pretty=format:"%h %s" HEAD~3..HEAD
a5f4a0d added cat-file
310154e updated README formatting and added blame
f7f3f6d changed my name a bit
위 결과의 역순임을 기억하자. 대화형 Rebase는 스크립트에 적혀 있는 순서대로 `HEAD~3`부터 적용하기 시작하고 위에서 아래로 각각의 커밋을 순서대로 수정한다. 순서대로 적용하는 것이기 때문에 제일 위에 있는 것이 최신이 아니라 가장 오래된 것이다.

특정 커밋에서 실행을 멈추게 하려면 스크립트를 수정해야 한다. `pick`이라는 단어를 'edit’로 수정하면 그 커밋에서 멈춘다. 가장 오래된 커밋 메시지를 수정하려면 아래와 같이 편집한다.

```
edit f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```

저장하고 편집기를 종료하면 Git은 목록에 있는 커밋 중에서 가장 오래된 커밋으로 이동하고, 아래와 같은 메시지를 보여주고, 명령 프롬프트를 보여준다.

```
$ git rebase -i HEAD~3
Stopped at f7f3f6d... changed my name a bit
You can amend the commit now, with

       git commit --amend

Once you’re satisfied with your changes, run

       git rebase --continue
```


명령 프롬프트가 나타날 때 Git은 Rebase 과정에서 현재 정확히 뭘 해야 하는지 메시지로 알려준다. 아래와 같은 명령을 실행하고

```
$ git commit --amend
```


커밋 메시지를 수정하고 텍스트 편집기를 종료하고 나서 아래 명령을 실행한다.

```
$ git rebase --continue
```


이렇게 나머지 두 개의 커밋에 적용하면 끝이다. 다른 것도 pick을 edit로 수정해서 이 작업을 몇 번이든 반복할 수 있다. 매번 Git이 멈출 때마다 커밋을 정정할 수 있고 완료할 때까지 계속 할 수 있다.

커밋 순서 바꾸기
대화형 Rebase 도구로 커밋 전체를 삭제하거나 순서를 조정할 수 있다. “added cat-file” 커밋을 삭제하고 다른 두 커밋의 순서를 변경하려면 아래와 같은 Rebase 스크립트를

```
pick f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```


아래와 같이 수정한다.

```
pick 310154e updated README formatting and added blame
pick f7f3f6d changed my name a bit
```

수정한 내용을 저장하고 편집기를 종료하면 Git은 브랜치를 이 커밋의 부모로 이동시키고서 `310154e`와 `f7f3f6d`를 순서대로 적용한다. 명령이 끝나고 나면 커밋 순서가 변경됐고 “added cat-file” 커밋이 제거된 것을 확인할 수 있다.

커밋 합치기
대화형 Rebase 명령을 이용하여 여러 개의 커밋을 꾹꾹 눌러서 커밋 하나로 만들어 버릴 수 있다. Rebase 스크립트에 자동으로 포함된 도움말에 설명이 있다.

```
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```


‘`pick’이나 ‘`edit'말고 ``squash’'를 입력하면 Git은 해당 커밋과 바로 이전 커밋을 합칠 것이고 커밋 메시지도 Merge 한다. 그래서 3개의 커밋을 모두 합치려면 스크립트를 아래와 같이 수정한다.

```
pick f7f3f6d changed my name a bit
squash 310154e updated README formatting and added blame
squash a5f4a0d added cat-file
```

저장하고 나서 편집기를 종료하면 Git은 3개의 커밋 메시지를 Merge 할 수 있도록 에디터를 바로 실행해준다.

```
# This is a combination of 3 commits.
# The first commit's message is:
changed my name a bit

# This is the 2nd commit message:

updated README formatting and added blame

# This is the 3rd commit message:

added cat-file
```

이 메시지를 저장하면 3개의 커밋이 모두 합쳐진 커밋 한 개만 남는다.

커밋 분리하기
커밋을 분리한다는 것은 기존의 커밋을 해제하고(혹은 되돌려 놓고) Stage를 여러 개로 분리하고 나서 그것을 원하는 횟수만큼 다시 커밋하는 것이다. 예로 들었던 커밋 세 개 중에서 가운데 것을 분리해보자. 이 커밋의 ‘`updated README formatting and added blame’을 ‘`updated README formatting'과 ``added blame’'으로 분리하는 것이다. rebase -i 스크립트에서 해당 커밋을 "edit"로 변경한다.

```
pick f7f3f6d changed my name a bit
edit 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```

저장하고 나서 명령 프롬프트로 넘어간 다음에 그 커밋을 해제하고 그 내용을 다시 두 개로 나눠서 커밋하면 된다. 저장하고 편집기를 종료하면 Git은 제일 오래된 커밋의 부모로 이동하고서 f7f3f6d`과 `310154e`을 처리하고 콘솔 프롬프트를 보여준다. 여기서 커밋을 해제하는 `git reset HEAD^ 라는 명령으로 커밋을 해제한다. 그러면 수정했던 파일은 Unstaged 상태가 된다. 그다음에 파일을 Stage 한 후 커밋하는 일을 원하는 만큼 반복하고 나서 `git rebase --continue`라는 명령을 실행하면 남은 Rebase 작업이 끝난다.

```
$ git reset HEAD^
$ git add README
$ git commit -m 'updated README formatting'
$ git add lib/simplegit.rb
$ git commit -m 'added blame'
$ git rebase --continue
```

나머지 a5f4a0d 커밋도 처리되면 히스토리는 아래와 같다.

```
$ git log -4 --pretty=format:"%h %s"
1c002dd added cat-file
9b29157 added blame
35cfb2b updated README formatting
f3cc40e changed my name a bit
```

다시 강조하지만 Rebase 하면 목록에 있는 모든 커밋의 SHA-1 값은 변경된다. 절대로 이미 서버에 Push 한 커밋을 수정하면 안 된다.

filter-branch는 포크레인
수정해야 하는 커밋이 너무 많아서 Rebase 스크립트로 수정하기 어려울 것 같으면 다른 방법을 사용하는 것이 좋다. 모든 커밋의 이메일 주소를 변경하거나 어떤 파일을 삭제하는 경우를 살펴보자. `filter-branch`라는 명령으로 수정할 수 있는데 Rebase가 삽이라면 이 명령은 포크레인이라고 할 수 있다. `filter-branch`도 역시 수정하려는 커밋이 이미 공개돼서 다른 사람과 함께 공유하는 중이라면 사용하지 말아야 한다. 하지만, 잘 쓰면 꽤 유용하다. `filter-branch`가 유용한 경우를 예로 들어 설명하기 때문에 여기에서 대충 어떤 경우에 유용할지 배울 수 있다.

모든 커밋에서 파일을 제거하기
갑자기 누군가 생각 없이 git add . 같은 명령을 실행해서 공룡 똥 덩어리를 커밋했거나 실수로 암호가 포함된 파일을 커밋해서 이런 파일을 다시 삭제해야 하는 상황을 살펴보자. 이런 상황은 생각보다 자주 발생한다. filter-branch`는 히스토리 전체에서 필요한 것만 골라내는 데 사용하는 도구다. `filter-branch`의 `--tree-filter`라는 옵션을 사용하면 히스토리에서 `passwords.txt 파일을 아예 제거할 수 있다.

```
$ git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
Rewrite 6b9b3cf04e7c5686a9cb838c3f36a8cb6a0fc2bd (21/21)
Ref 'refs/heads/master' was rewritten
```

--tree-filter 옵션은 프로젝트를 Checkout 한 후에 각 커밋에 명시한 명령을 실행시키고 그 결과를 다시 커밋한다. 이 경우에는 각 스냅샷에 passwords.txt 파일이 있으면 그 파일을 삭제한다. 실수로 편집기의 백업파일을 커밋했으면 `git filter-branch --tree-filter rm -f *~ HEAD`라고 실행해서 삭제할 수 있다.

이 명령은 모든 파일과 커밋을 정리하고 브랜치 포인터를 다시 복원해준다. 이런 작업은 테스팅 브랜치에서 실험하고 나서 master 브랜치에 적용하는 게 좋다. filter-branch 명령에 --all 옵션을 추가하면 모든 브랜치에 적용할 수 있다.

하위 디렉토리를 루트 디렉토리로 만들기
다른 VCS에서 코드를 임포트하면 그 VCS만을 위한 디렉토리가 있을 수 있다. SVN에서 코드를 임포트하면 trunk, tags, branch 디렉토리가 포함된다. 모든 커밋에 대해 trunk 디렉토리를 프로젝트 루트 디렉토리로 만들 때도 filter-branch 명령이 유용하다.

```
$ git filter-branch --subdirectory-filter trunk HEAD
Rewrite 856f0bf61e41a27326cdae8f09fe708d679f596f (12/12)
Ref 'refs/heads/master' was rewritten
```

이제 trunk 디렉토리를 루트 디렉토리로 만들었다. Git은 입력한 디렉토리와 관련이 없는 커밋을 자동으로 삭제한다.

모든 커밋의 이메일 주소를 수정하기
프로젝트를 오픈소스로 공개할 때 아마도 회사 이메일 주소로 커밋된 것을 개인 이메일 주소로 변경해야 한다. 아니면 아예 git config`로 이름과 이메일 주소를 설정하는 것을 잊었을 수도 있다. 자신의 이메일 주소만 변경하도록 조심해야 한다. `filter-branch 명령의 --commit-filter 옵션을 사용하여 해당 커밋만 골라서 이메일 주소를 수정할 수 있다.

```
$ git filter-branch --commit-filter '
        if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
        then
                GIT_AUTHOR_NAME="Scott Chacon";
                GIT_AUTHOR_EMAIL="schacon@example.com";
                git commit-tree "$@";
        else
                git commit-tree "$@";
        fi' HEAD
```
이메일 주소를 새 주소로 변경했다. 모든 커밋은 부모의 SHA-1 값을 가지고 있기 때문에 조건에 만족하는 커밋의 SHA-1값만 바뀌는 것이 아니라 모든 커밋의 SHA-1 값이 바뀐다.

---

#  Reset 명확히 알고 가기

###Reset 명확히 알고 가기

Git의 다른 특별한 도구를 더 살펴보기 보기 전에 `reset`과 `checkout`에 대해 이야기를 해보자. 이 두 명령은 Git을 처음 사용하는 사람을 가장 헷갈리게 하는 부분이다. 제대로 이해하고 사용할 수 없을 것으로 보일 정도로 많은 기능을 지녔다. 이해하기 쉽게 간단한 비유를 들어 설명해보자.

세 개의 트리
Git을 서로 다른 세 트리를 관리하는 컨텐츠 관리자로 생각하면 `reset`과 `checkout`을 좀 더 쉽게 이해할 수 있다. 여기서 “트리” 란 실제로는 “파일의 묶음” 이다. 자료구조의 트리가 아니다 (세 트리 중 Index는 트리도 아니지만, 이해를 쉽게 하려고 일단 트리라고 한다).

Git은 일반적으로 세 가지 트리를 관리하는 시스템이다.

트리	역할
HEAD

마지막 커밋 스냅샷, 다음 커밋의 부모 커밋

Index

다음에 커밋할 스냅샷

워킹 디렉토리

샌드박스

HEAD
HEAD는 현재 브랜치를 가리키는 포인터이며, 브랜치는 브랜치에 담긴 커밋 중 가장 마지막 커밋을 가리킨다. 지금의 HEAD가 가리키는 커밋은 바로 다음 커밋의 부모가 된다. 단순하게 생각하면 HEAD는 마지막 커밋의 스냅샷이다.

HEAD가 가리키는 스냅샷을 살펴보기는 쉽다. 아래는 HEAD 스냅샷의 디렉토리 리스팅과 각 파일의 SHA-1 체크섬을 보여주는 예제다.

```
$ git cat-file -p HEAD
tree cfda3bf379e4f8dba8717dee55aab78aef7f4daf
author Scott Chacon  1301511835 -0700
committer Scott Chacon  1301511835 -0700

initial commit

$ git ls-tree -r HEAD
100644 blob a906cb2a4a904a152...   README
100644 blob 8f94139338f9404f2...   Rakefile
040000 tree 99f1a6d12cb4b6f19...   lib
```

cat-file`와 `ls-tree 명령은 일상적으로는 잘 사용하지 않는 저수준 명령이다. 이런 저수준 명령을 “plumbing” 명령이라고 한다. Git이 실제로 무슨 일을 하는지 볼 때 유용하다.

Index
Index는 바로 다음에 커밋할 것들이다. 이미 앞에서 우리는 이런 개념을 ‘`Staging Area’라고 배운 바 있다. ``Staging Area'는 사용자가 git commit 명령을 실행했을 때 Git이 처리할 것들이 있는 곳이다.

먼저 Index는 워킹 디렉토리에서 마지막으로 Checkout 한 브랜치의 파일 목록과 파일 내용으로 채워진다. 이후 파일 변경작업을 하고 변경한 내용으로 Index를 업데이트 할 수 있다. 이렇게 업데이트 하고 git commit 명령을 실행하면 Index는 새 커밋으로 변환된다.

```
$ git ls-files -s
100644 a906cb2a4a904a152e80877d4088654daad0c859 0	README
100644 8f94139338f9404f26296befa88755fc2598c289 0	Rakefile
100644 47c6340d6459e05787f644c2447d2595f5d3a54b 0	lib/simplegit.rb
```

또 다른 저수준 ls-files 명령은 훨씬 더 장막 뒤에 가려져 있는 명령으로 이를 실행하면 현재 Index가 어떤 상태인지를 확인할 수 있다.

Index는 엄밀히 말해 트리구조는 아니다. 사실 Index는 평평한 구조로 구현되어 있다. 여기에서는 쉽게 이해할 수 있도록 그냥 트리라고 설명한다.

워킹 디렉토리
마지막으로 워킹 디렉토리를 살펴보자. 위의 두 트리는 파일과 그 내용을 효율적인 형태로 .git 디렉토리에 저장한다. 하지만, 사람이 알아보기 어렵다. 워킹 디렉토리는 실제 파일로 존재한다. 바로 눈에 보이기 때문에 사용자가 편집하기 수월하다. 워킹 디렉토리는 샌드박스로 생각하자. 커밋하기 전에는 Index(Staging Area)에 올려놓고 얼마든지 변경할 수 있다.

```
$ tree
.
├── README
├── Rakefile
└── lib
    └── simplegit.rb

1 directory, 3 files
```

워크플로
Git의 주목적은 프로젝트의 스냅샷을 지속적으로 저장하는 것이다. 이 트리 세 개를 사용해 더 나은 상태로 관리한다.

![](https://git-scm.com/book/en/v2/images/reset-workflow.png)

이 과정을 시각화해보자. 파일이 하나 있는 디렉토리로 이동한다. 이걸 파일의 v1이라고 하고 파란색으로 표시한다. git init 명령을 실행하면 Git 저장소가 생기고 HEAD는 아직 없는 브랜치를 가리킨다(`master`는 아직 없다).

![](https://git-scm.com/book/en/v2/images/reset-ex1.png)
이 시점에서는 워킹 디렉토리 트리에만 데이터가 있다.

이제 파일을 커밋해보자. git add 명령으로 워킹 디렉토리의 내용을 Index로 복사한다.

![](https://git-scm.com/book/en/v2/images/reset-ex2.png)
그리고 git commit 명령을 실행한다. 그러면 Index의 내용을 스냅샷으로 영구히 저장하고 그 스냅샷을 가리키는 커밋 객체를 만든다. 그리고는 `master`가 그 커밋 객체를 가리키도록 한다.

![](https://git-scm.com/book/en/v2/images/reset-ex3.png)

이때 git status 명령을 실행하면 아무런 변경 사항이 없다고 나온다. 세 트리 모두가 같기 때문이다.

다시 파일 내용을 바꾸고 커밋해보자. 위에서 했던 것과 과정은 비슷하다. 먼저 워킹 디렉토리의 파일을 고친다. 이를 이 파일의 v2라고 하자. 이건 빨간색으로 표시한다.

![](https://git-scm.com/book/en/v2/images/reset-ex4.png)

git status 명령을 바로 실행하면 “Changes not staged for commit,” 아래에 빨간색으로 된 파일을 볼 수 있다. Index와 워킹 디렉토리가 다른 내용을 담고 있기 때문에 그렇다. git add 명령을 실행해서 변경 사항을 Index에 올려주자.

![](https://git-scm.com/book/en/v2/images/reset-ex5.png)

이 시점에서 git status 명령을 실행하면 “Changes to be committed” 아래에 파일 이름이 녹색으로 변한다. Index와 HEAD의 다른 파일들이 여기에 표시된다. 즉 다음 커밋할 것과 지금 마지막 커밋이 다르다는 말이다. 마지막으로 git commit 명령을 실행해 커밋한다.

![](https://git-scm.com/book/en/v2/images/reset-ex6.png)

이제 git status 명령을 실행하면 아무것도 출력하지 않는다. 세 개의 트리의 내용이 다시 같아졌기 때문이다.

브랜치를 바꾸거나 Clone 명령도 내부에서는 비슷한 절차를 밟는다. 브랜치를 Checkout 하면, HEAD가 새로운 브랜치를 가리키도록 바뀌고, 새로운 커밋의 스냅샷을 Index에 놓는다. 그리고 Index의 내용을 워킹 디렉토리로 복사한다.

Reset 의 역할
위의 트리 세 개를 이해하면 reset 명령이 어떻게 동작하는지 쉽게 알 수 있다.

예로 들어 file.txt 파일 하나를 수정하고 커밋한다. 이것을 세 번 반복한다. 그러면 히스토리는 아래와 같이 된다.

![](https://git-scm.com/book/en/v2/images/reset-start.png)

자 이제 reset 명령이 정확히 어떤 일을 하는지 낱낱이 파헤쳐보자. reset 명령은 이 세 트리를 간단하고 예측 가능한 방법으로 조작한다. 트리를 조작하는 동작은 세 단계 이하로 이루어진다.

1 단계: HEAD 이동
reset 명령이 하는 첫 번째 일은 HEAD 브랜치를 이동시킨다. checkout 명령처럼 HEAD가 가리키는 브랜치를 바꾸지는 않는다. HEAD는 계속 현재 브랜치를 가리키고 있고, 현재 브랜치가 가리키는 커밋을 바꾼다. HEAD가 master 브랜치를 가리키고 있다면(즉 master 브랜치를 Checkout 하고 작업하고 있다면) git reset 9e5e6a4 명령은 master 브랜치가 `9e5e6a4`를 가리키게 한다.

![](https://git-scm.com/book/en/v2/images/reset-soft.png)

reset 명령에 커밋을 넘기고 실행하면 언제나 이런 작업을 수행한다. reset --soft 옵션을 사용하면 딱 여기까지 진행하고 동작을 멈춘다.

이제 위의 다이어그램을 보고 어떤 일이 일어난 것인지 생각해보자. reset 명령은 가장 최근의 git commit 명령을 되돌린다. git commit 명령을 실행하면 Git은 새로운 커밋을 생성하고 HEAD가 가리키는 브랜치가 새로운 커밋을 가리키도록 업데이트한다. reset 명령 뒤에 HEAD~(HEAD의 부모 커밋)를 주면 Index나 워킹 디렉토리는 그대로 놔두고 브랜치가 가리키는 커밋만 이전으로 되돌린다. Index를 업데이트한 다음에 git commit 명령를 실행하면 git commit --amend 명령의 결과와 같아진다(마지막 커밋을 수정하기를 참조).

2 단계: Index 업데이트 (--mixed)
여기서 git status 명령을 실행하면 Index와 reset 명령으로 이동시킨 HEAD의 다른 점이 녹색으로 출력된다.

reset 명령은 여기서 한 발짝 더 나아가 Index를 현재 HEAD가 가리키는 스냅샷으로 업데이트할 수 있다.

![](https://git-scm.com/book/en/v2/images/reset-mixed.png)

--mixed 옵션을 주고 실행하면 reset 명령은 여기까지 하고 멈춘다. reset 명령을 실행할 때 아무 옵션도 주지 않으면 기본적으로 --mixed 옵션으로 동작한다(예제와 같이 git reset HEAD~ 처럼 명령을 실행하는 경우).

위의 다이어그램을 보고 어떤 일이 일어날지 한 번 더 생각해보자. 가리키는 대상을 가장 최근의 커밋`으로 되돌리는 것은 같다. 그러고 나서 _Staging Area_를 비우기까지 한다. `git commit 명령도 되돌리고 git add 명령까지 되돌리는 것이다.

3 단계: 워킹 디렉토리 업데이트 (--hard)
reset 명령은 세 번째로 워킹 디렉토리까지 업데이트한다. --hard 옵션을 사용하면 reset 명령은 이 단계까지 수행한다.

![](https://git-scm.com/book/en/v2/images/reset-hard.png)

이 과정은 어떻게 동작하는지 가늠해보자. reset 명령을 통해 git add`와 `git commit 명령으로 생성한 마지막 커밋을 되돌린다. 그리고 워킹 디렉토리의 내용까지도 되돌린다.

이 --hard 옵션은 매우 매우 중요하다. reset 명령을 위험하게 만드는 유일한 옵션이다. Git에는 데이터를 실제로 삭제하는 방법이 별로 없다. 이 삭제하는 방법은 그 중 하나다. reset 명령을 어떻게 사용하더라도 간단히 결과를 되돌릴 수 있다. 하지만 --hard 옵션은 되돌리는 것이 불가능하다. 이 옵션을 사용하면 워킹 디렉토리의 파일까지 강제로 덮어쓴다. 이 예제는 파일의 v3버전을 아직 Git이 커밋으로 보관하고 있기 때문에 `reflog`를 이용해서 다시 복원할 수 있다. 만약 커밋한 적 없다면 Git이 덮어쓴 데이터는 복원할 수 없다.

복습
reset 명령은 정해진 순서대로 세 개의 트리를 덮어써 나가다가 옵션에 따라 지정한 곳에서 멈춘다.

HEAD가 가리키는 브랜치를 옮긴다. (--soft 옵션이 붙으면 여기까지)

Index를 HEAD가 가리키는 상태로 만든다. (--hard 옵션이 붙지 않았으면 여기까지)

워킹 디렉토리를 Index의 상태로 만든다.

경로를 주고 Reset 하기
지금까지 reset 명령을 실행하는 기본 형태와 사용 방법을 살펴봤다. reset 명령을 실행할 때 경로를 지정하면 1단계를 건너뛰고 정해진 경로의 파일에만 나머지 reset 단계를 적용한다. 이는 당연한 이야기다. HEAD는 포인터인데 경로에 따라 파일별로 기준이 되는 커밋을 부분적으로 적용하는 건 불가능하다. 하지만, Index나 워킹 디렉토리는 일부분만 갱신할 수 있다. 따라서 2, 3단계는 가능하다.

예를 들어 git reset file.txt 명령을 실행한다고 가정하자. 이 형식은(커밋의 해시 값이나 브랜치도 표기하지 않고 `--soft`나 `--hard`도 표기하지 않은) `git reset --mixed HEAD file.txt`를 짧게 쓴 것이다.

HEAD의 브랜치를 옮긴다. (건너뜀)

Index를 HEAD가 가리키는 상태로 만든다. (여기서 멈춤)

본질적으로는 file.txt 파일을 HEAD에서 Index로 복사하는 것뿐이다.

![](https://git-scm.com/book/en/v2/images/reset-path1.png)

이 명령은 해당 파일을 Unstaged 상태로 만든다. 이 명령의 다이어그램과 git add 명령을 비교해보면 정확히 반대인 것을 알 수 있다.

![](https://git-scm.com/book/en/v2/images/reset-path2.png)

이것이 git status 명령에서 이 명령을 보여주는 이유다. 이 명령으로 파일을 Unstaged 상태로 만들 수 있다. (더 자세한 내용은 파일 상태를 Unstage로 변경하기를 참고한다.)

특정 커밋을 명시하면 Git은 “`HEAD에서 파일을 가져오는” 것이 아니라 그 커밋에서 파일을 가져온다. git reset eb43bf file.txt 명령과 같이 실행한다.

![](https://git-scm.com/book/en/v2/images/reset-path3.png)

이 명령을 실행한 것과 같은 결과를 만들려면 워킹 디렉토리의 파일을 v1으로 되돌리고 git add 명령으로 Index를 v1으로 만들고 나서 다시 워킹 디렉토리를 v3로 되돌려야 한다(결과만 같다는 얘기다). 이 상태에서 git commit 명령을 실행하면 v1으로 되돌린 파일 내용을 기록한다. 워킹 디렉토리를 사용하지 않았다.

git add 명령처럼 reset 명령도 Hunk 단위로 사용할 수 있다. --patch 옵션을 사용해서 Staging Area에서 Hunk 단위로 Unstaged 상태로 만들 수 있다. 이렇게 선택적으로 Unstaged 상태로 만들거나 내리거나 이전 버전으로 복원시킬 수 있다.

합치기(Squash)
여러 커밋을 커밋 하나로 합치는 재밌는 도구를 알아보자.

“oops.''나 ``WIP”, “forgot this file” 같은 깃털같이 가벼운 커밋들이 있다고 해보자. 이럴 때는 reset 명령으로 커밋들을 하나로 합쳐서 남들에게 똑똑한 척할 수 있다. (커밋 합치기를 하는 명령어가 따로 있지만, 여기서는 reset 명령을 쓰는 것이 더 간단할 때도 있다는 것을 보여준다.)

이런 프로젝트가 있다고 생각해보자. 첫 번째 커밋은 파일 하나를 추가했고, 두 번째 커밋은 기존 파일을 수정하고 새로운 파일도 추가했다. 세 번째 커밋은 첫 번째 파일을 다시 수정했다. 두 번째 커밋은 아직 작업 중인 커밋으로 이 커밋을 세 번째 커밋과 합치고 싶은 상황이다.

![](https://git-scm.com/book/en/v2/images/reset-squash-r1.png)

git reset --soft HEAD~2 명령을 실행하여 HEAD 포인터를 이전 커밋으로 되돌릴 수 있다(히스토리에서 그대로 유지할 처음 커밋 말이다).

![](https://git-scm.com/book/en/v2/images/reset-squash-r2.png)

이 상황에서 git commit 명령을 실행한다.

![](https://git-scm.com/book/en/v2/images/reset-squash-r3.png)

이제 사람들에게 공개할만한 히스토리가 만들어졌다. file-a.txt 파일이 있는 v1 커밋이 하나 그대로 있고, 두 번째 커밋에는 v3버전의 file-a.txt 파일과 새로 추가된 file-b.txt 파일이 있다. v2 버전은 더는 히스토리에 없다.

Checkout
아마도 checkout 명령과 reset 명령에 어떤 차이가 있는지 궁금할 것이다. reset 명령과 마찬가지로 checkout 명령도 위의 세 트리를 조작한다. checkout 명령도 파일 경로를 쓰느냐 안 쓰느냐에 따라 동작이 다르다.

경로 없음
git checkout [branch] 명령은 git reset --hard [branch] 명령과 비슷하게 [branch] 스냅샷을 기준으로 세 트리를 조작한다. 하지만, 두 가지 사항이 다르다.

첫 번째로 reset --hard 명령과는 달리 checkout 명령은 워킹 디렉토리를 안전하게 다룬다. 저장하지 않은 것이 있는지 확인해서 날려버리지 않는다는 것을 보장한다. 사실 보기보다 좀 더 똑똑하게 동작한다. 워킹 디렉토리에서 Merge 작업을 한번 시도해보고 변경하지 않은 파일만 업데이트한다. 반면 reset --hard 명령은 확인하지 않고 단순히 모든 것을 바꿔버린다.

두 번째 중요한 차이점은 HEAD를 업데이트 하는가이다. reset 명령은 HEAD가 가리키는 브랜치를 움직이지만(브랜치 Refs를 업데이트하지만), checkout 명령은 HEAD 자체를 다른 브랜치로 옮긴다.

예를 들어 각각 다른 커밋을 가리키는 master`와 `develop 브랜치가 있고 현재 워킹 디렉토리는 develop 브랜치라고 가정해보자(즉 HEAD는 develop 브랜치를 가리킨다). git reset master 명령을 실행하면 develop 브랜치는 master 브랜치가 가리키는 커밋과 같은 커밋을 가리키게 된다. 반면 git checkout master 명령을 실행하면 develop 브랜치가 가리키는 커밋은 바뀌지 않고 HEAD가 master 브랜치를 가리키도록 업데이트된다. 이제 HEAD는 master 브랜치를 가리키게 된다.

그래서 위 두 경우 모두 HEAD는 결과적으로 A 커밋을 가리키게 되지만 방식은 완전히 다르다. reset 명령은 HEAD가 가리키는 브랜치의 포인터를 옮겼고 checkout 명령은 HEAD 자체를 옮겼다.

![](https://git-scm.com/book/en/v2/images/reset-checkout.png)

경로 있음
checkout 명령을 실행할 때 파일 경로를 줄 수도 있다. reset 명령과 비슷하게 HEAD는 움직이지 않는다. 동작은 git reset [branch] file 명령과 비슷하다. Index의 내용이 해당 커밋 버전으로 변경될 뿐만 아니라 워킹 디렉토리의 파일도 해당 커밋 버전으로 변경된다. 완전히 git reset --hard [branch] file 명령의 동작이랑 같다. 워킹 디렉토리가 안전하지도 않고 HEAD도 움직이지 않는다.

git reset`이나 `git add 명령처럼 checkout 명령도 --patch 옵션을 사용해서 Hunk 단위로 되돌릴 수 있다.

요약
reset 명령이 좀 더 쉬워졌을 거라고 생각한다. 아직 checkout 명령과 정확하게 무엇이 다른지 혼란스럽거나 정확한 사용법을 다 익히지 못했을 수도 있지만 괜찮다.

아래에 어떤 명령이 어떤 트리에 영향을 주는지에 대한 요약표를 준비했다. 명령이 HEAD가 가리키는 브랜치를 움직인다면 “HEAD” 열에 ‘`REF’라고 적혀 있고 HEAD 자체가 움직인다면 ``HEAD'라고 적혀 있다. WD Safe? 열을 꼭 보자. 여기에 NO라고 적혀 있다면 워킹 디렉토리에 저장하지 않은 내용이 안전하지 않기 때문에 해당 명령을 실행하기 전에 한 번쯤 더 생각해보아야 한다.