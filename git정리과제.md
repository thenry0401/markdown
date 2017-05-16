# Git 도구 - Stashing 과 Cleaning

**git stash** 

아직 완료하지 않은 일을 커밋하지 않고 나중에 다시 돌아와서 작업을 해야할 때 사용한다
아직 끝내지 않은 수정사항을 스택에 잠시 저장했다가 나중에 다시 적용할 수 있다

Git은 Stash를 적용할 때 Staged 상태였던 파일을 자동으로 다시 Staged 상태로 만들어 주지 않는다. 그래서 git stash apply 명령을 실행할 때 --index 옵션을 주어 Staged 상태까지 적용한다. 그래야 원래 작업하던 상태로 돌아올 수 있다.