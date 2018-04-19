# 03. AWS로 배포하기(우분투서버 내 설정)

####python-pip 설치

```
sudo apt-get install python-pip
```

####zsh 설치

```
sudo apt-get install zsh
```

####oh-my-zsh 설치

```
sudo curl -L http://install.ohmyz.sh | sh
```

####Default shell 변경

```
sudo chsh ubuntu -s /usr/bin/zsh
```

####pyenv requirements 설치

```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils
```

#### pyenv 설치

```
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```

#### pyenv 설정 .zshrc에 기록

```
vi ~/.zshrc
export PATH="/home/ubuntu/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```


#### Pillow 라이브러리 설치

[공식문서](https://pillow.readthedocs.io/en/3.4.x/installation.html#basic-installation)

```
sudo apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk
```




