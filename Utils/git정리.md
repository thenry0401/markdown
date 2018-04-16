GIT
===

###Git 저장소 만들기

- 기존 프로젝트를 Git저장소로 만들기:

```
$ git init
```

```
$ git add *.c
$ git add LICENSE
$ git commit -m 'initial project version'
```

- 기존 저장소를 clone하기

```
$ git clone https://github.com/아이디/레퍼짓토리이름.git
```

---

### Git의 기초 - 수정하고 저장소에 저장하기

![](https://git-scm.com/book/en/v2/images/lifecycle.png)

>워킹 디렉토리의 모든 파일은 크게 Tracked(관리대상임)와 Untracked(관리대상이 아님)로 나눈다.

 >Tracked 파일은 또 Unmodified(수정하지 않음)와 Modified(수정함) 그리고 Staged(커밋으로 저장소에 기록할) 상태 중 하나이다

**파일의 상태 확인하기:**

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working directory clean
```

**파일을 새로 추적하기**

`git add` 명령을 실행하여 Git이 파일을 추적하게 한다

- 명령 실행 후 파일의 상태:

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
```

**Modified 상태의 파일을 Stage 하기**

- 이미 tracked상태의 파일을 수정 후 상태확인:

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

- **git add** 명령 실행 후 상태확인:

```
$ git add CONTRIBUTING.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```

---

**파일 무시하기**

Git이 관리할 필요 없는 파일들은 `.gitignore` 파일을 만들고 안에 무시할 패턴들을 적는다

---

**변경사항 커밋하기**

```
$ git commit
```

 위 명령을 입력 하면 아래 화면이 나온다

```
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Your branch is up-to-date with 'origin/master'.
# Changes to be committed:
#	new file:   README
#	modified:   CONTRIBUTING.md
#
~
~
~
".git/COMMIT_EDITMSG" 9L, 283C
```

맨 윗줄에 변경되는 커밋에 대한 내용을 요약하여 적는다

---

**파일 삭제하기**

Git에서 파일을 삭제하려면 `git rm` 명령으로 tracked상태의 파일을 삭제한 후에 커밋을 해야한다

**파일 이름 변경하기**

```
$ git mv file_form file_to
```

---

**커밋 히스토리 조회하기**

`git log` 명령을 입력하면 커밋했던 기록들을 확인할 수 있다

```
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```
