---
layout: post
title: The Linux File System
tags:
categories: knowledge
---

## Contents

- The Linux File system
  - /bin
  - /sbin
  - /usr
- Executables

- Recommended Executables
  - Node.js
  - Python

## The Linux File System

일반적인 맥북유저라면 리눅스의 파일시스템을 다룰 일이 없겠지만, 이것저것 하다보면 궁금증이 생기길 마련이다. Mac OS X는 보통 'Linux like' OS 라고 하지만, 정확히 말하자면 xnu kernel 위에 Darwin 이라는 Unix의 버젼이 융합된 것이다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/devops/linuxfilesystem/img1.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Darwin 도 결국 Unix 의 파일시스템을 따르기 때문에, 리눅스 파일시스템을 이해하면 터미널의 작업들과 개발환경에서 마주치는 설정들을 이해하기 쉬워진다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/devops/linuxfilesystem/img2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

리눅스의 파일시스템은 위와 같다. 이 파일시스템은 오픈소스로 관리해야하기 때문에 Linux Foundation 에서 [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.pdf) 라는 문서로 버젼관리를 한다.

일반 맥북에서 리눅스 파일들을 접근하는 방법은 두가지가 있다.

1. Finder 에서 cmd+shift+c 로 컴퓨터 root 로 접근 한 뒤, cmd+shift+. 로 숨겨진 파일들 보기

2. 터미널 cli 환경에서 커맨드로 접근하는 방법

1번은 느리고 GUI 환경이기 때문에 실수로 파일들을 수정하는 대참사가 일어날 수 있기 때문에, 터미널 환경에서 확인을 해보자.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/devops/linuxfilesystem/img3.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

cd / 로 root directory 로 접근 할 수 있고, ls 로 root 에 무슨 파일들이 있는지 볼 수 있다. 본인은 맥북 M1 을 사용하기 때문에, 기존 인텔맥북과도 다른 파일시스템이지만, 본질적인 개념은 동일하다.

##### /bin

bin 폴더는 binaries 를 담고 있는 directory 이며, OS 가 작동하는데 필수적인 executables 들이 들어있다. CLI 는 bash (정확히 말하자면 bash 의 일종인 zsh) 쉘스크립트 언어로 작동하기 때문에 이 폴더에 있는 executables 은 전부 쉘스크립트이며, global 이기 때문에 터미널 어디서든 실행시킬 수 있다. 실제로 제일 많이 쓰는 커맨드인 cd 나 pwd, ls 다 bin directory 안에 있다.

##### /sbin

sbin 폴더는 system binaries 를 담고 있는 directory 이며, super user 가 실행시킬 수 있는 mount, shutdown, reboot 같은 executables 들이 들어있다.

> 인탤맥북에선 root directory 에 lib 이란 폴더가 있는데, M1맥북에선 usr 디렉토리 안에 있다.

##### /usr

usr 폴더는 user binaries 를 담고 있는 directory 이며, 사용자가 실행 시킬 수 있는 exectables 들을 모아둔 directory 들이 담겨있다.

리눅스 파일시스템들은 이것보다 훨씬 방대하지만, 환경설정을 할 때 이 3개의 directory 의 목적만 알면 충분하다.

## Executables

맥북으로 개발환경을 처음 구축 할 때 헷갈리는게 많다. Ruby 나 Python 은 기본적으로 설치되어 있지만, package managing 을 위해 gem, chruby 나 conda 를 사용한다. Homebrew 로 Node 를 설치 할 수 있지만, Node 웹사이트로 가서 .pkg 로 까는 방법이나 conda 같은 package manager 인 nvm 을 깔아서 Node 를 사용 할 수도 있다 ~~homebrew 로 nvm 을 까는 방법도 있다~~.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/devops/linuxfilesystem/img4.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

본질적으로 개발환경이란 건 사용자 입맛에 맞게 사용하도록 극강의 자유도를 위해 존재하지만, 정확히 어떤 executable 를 사용하는지 알고, 해당 executable 이 어디에 존재하는지 알아야만 나중에 환경충돌을 해결 할 수 있다. 그냥 설치하라는대로 막 깔다 보면 directory hell이 펼쳐진다.

```shell
echo $PATH
```

$PATH 라는 환경변수는 kernel 이 executables 가 이런 위치에 있고, 어떤 directory 에 존재하는지 mapping 해준다. 사용자가 설치한 executables (conda, node, nvm 등등) 이 어디 directory 에 있는지 확인 할 수 있는 command 다.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/devops/linuxfilesystem/img5.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

본인의 $PATH 변수이다. 자세히 보면 : 으로 executables 들이 나눠져 있다.

- Node v17.4.0 - /.nvm

- Ruby v3.1.0 - /.gem

- Python - /miniforge3/bin

- Homebrew - /opt/homebrew/bin

등 본인이 제일 많이 사용하는 executables 들의 위치이다. 그렇지만 같은 package 가 다른 directory 에 중복으로 설치 된 경우는 어떨까? 터미널에서 명령어를 치면 어떤 directory 의 어떤 executable 이 실행되는지 확인하는 습관을 기르는건 좋다.

예시를 들어보자.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/devops/linuxfilesystem/img6.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

본인은 개발환경으로 python, node, ruby 를 제일 많이 사용한다. where 명령어로 해당 executables 의 directory 들을 확인 할 수 있고, which 명령어로 현재 shell 에서 사용하고 있는 executable 의 directory 를 확인 할 수 있다.

**Python**은 두개의 경로가 있다. 첫 경로는 miniforge 로 사용한 ARM compatible Python 이고, 두번째 경로는 built in python 이다. which python 으로 확인해본 결과 현재 shell 은 miniforge python 을 사용하고 있다.

**Node.js** 도 두개의 경로가 있다. 첫 경로는 nvm package manager 로 설치한 Node.js 이고, 두번째 경로는 Homebrew 로 설치한 Node.js 이다. which node 으로 확인해보면 현재 shell 은 nvm 의 node 를 사용하고 있다.

**Ruby** 도 두개의 경로가 있다. 첫 경로는 homebrew 로 설치한 ruby 이고, 두번째 경로는 built in ruby 다. which ruby 로 확인해보면 현재 shell 은 homebrew 로 설치한 /.rubies 의 ruby 를 사용하고 있다.

어떤 executable 를 사용하는지에 따라 version control 과 virtual environment 를 설정 할 수있는지의 유무가 갈리기 때문에, 되도록이면 본인이 설치한 executables 를 잘 기억해야 한다.

## Recommended Executables

ARM 아키텍쳐를 사용하는 M1 맥북을 기준으로 본인이 선호하는 경로를 소개하겠다.

##### Node.js

Node 같은 경우 설치 할 수 있는 방법이 다양하다. [Node 공식홈페이지](https://nodejs.org/en/)에서 .pkg 를 설치하는 방법, Homebrew 를 통한 [brew install node 방법](https://formulae.brew.sh/formula/node#default) (npm 도 같이 딸려온다), 그리고 Node.js 의 package manager (conda 와 같은) [nvm](https://github.com/nvm-sh/nvm) 을 직접 설치하는 방법이 있다.

nvm 을 직접 설치하는걸 추천하는데, Node.js 를 위해 특화된 version control 이 가능하고, conda 와 같이 nvm use 명령어를 통해 이전 버젼을 쉽게 사용할 수 있기 때문이다.

nvm 을 설치하는 법은 간단하다.

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

를 터미널에서 실행시키면 home directory 에 숨겨진 파일 .nvm 이 생기면서 nvm 명령어가 활성화 된다.

```shell
command -v nvm
```

위 커맨드를 실행시켰을 때 nvm 이라고 뜨면 설치가 완료된다.

##### Python

Mac OS X 환경에서 파이썬을 설치 할 수 있는 방법은 많지만, ARM 아키텍쳐 M1 맥에서는 더욱 세분화 된다. 보통 conda 를 사용해서 가상환경을 다루지만, conda 는 인텔맥북 특화이기 때문에 모든 스레드가 rosetta 를 통해 변환된 상태로 작업이 이루어진다. 이를 위한 대안으로 conda-forge (이하 miniforge) 가 존재한다. Miniforge 는 ARM 아키텍쳐 특화이며, miniforge 로 M1 칩의 최대 장점인 GPU 가속화 패키지인 metal 을 사용할 수 있다. 하지만 Miniforge 에서 사용하는 라이브러리들이 ARM 환경에서 아직 업데이트가 안된 경우가 종종 있어, 이전에 개발된 source code 를 사용하려면 requirements 를 하나씩 다 뜯어봐야할 수도 있다 (geopandas 가 아직도 지원이 안된다).

분명 conda 와 miniforge 는 둘만의 장단점이 있다. conda 는 모든 라이브러리들이 정상적으로 작동하지만 느리다. miniforge 는 M1 칩의 최대장점인 속도를 파이썬 환경으로 가져올 수 있게하지만, ARM 미지원 라이브러리는 사용 불가능하다. 그렇기 때문에 보통 사용자들은 둘중 하나를 선택하지만, 둘다 parallel 로 까는 방법이 존재한다.

[Mac M1 Monterey Installing Miniforge and Anaconda/Miniconda Side-by-Side](https://www.youtube.com/watch?v=w2qlou7n7MA)

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/posts/devops/linuxfilesystem/img7.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

굉장히 쉬운 튜토리얼인데, 최종 결과물로 위와 같이 기본 shell 을 miniforge 로 설정하고 conda executable 을 활성화 시키고

```bash
source start_miniconda.sh
```

을 실행시키면 shell 이 miniconda 로 바뀐다. miniforge, miniconda 둘다 conda executable 을 활성화 시키고 평소대로 conda create 로 각 아키텍쳐에 맞는 가상환경 또한 설정할 수 있게 된다.
