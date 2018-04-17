#SSH 키 생성하고 Github에 등록하기

1. 컴퓨터의 home 위치에 `.ssh`폴더 생성하기
`mkdir ~/.ssh`

2. `.ssh`폴더로 이동

3.  ssh 키 생성
`ssh-keygen -t rsa -b 4096 -C "본인 이메일"`

```
Enter file in which to save the key (/home/thenry/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:

이 부분들 그냥 내용 없이 엔터로 넘어간다
```

4. `.ssh`폴더에 `id_rsa`파일과 `id_rsa.pub`파일이 생성됐는지 확인

5. Github에서 Settings - SSH and GPG keys 에서 등록해준다

```
id_rsa.pub의 내용을 등록해줘야한다
```
