안녕하세요. 오늘은 Private Repository를 clone하는 Docker Image를 만드는 방법에 대해 알아보도록 하겠습니다. Private Repo를 활용하면 회사에서 개발중인 라이브러리를 도커이미지에 포함하는 등의 작업도 할 수 있습니다.

### 예상 독자과 준비물
해당 포스트를 읽는 독자는 Dockerfile을 통해 Docker Image를 만든 경험이 있다고 가정합니다. 또한 본인의 github 계정으로 접근할 수 있는 Private Repository가 필요합니다.
* Dockerfile로 이미지 빌드 경험
* Private Repository

# 제안하는 방식 3가지
해당 포스트에서 제안하는 Private Repository가 포함된 Docker Image를 만드는 방식은 아래 세가지와 같습니다. 각각의 장단점은 구체적인 방법을 소개하면서 말씀드리도록 하겠습니다.
* 미리 clone하여 Dockerfile에서 copy 하기
* Dockerfile에서 github 계정 & 토큰 정보 추가하여 clone하기
* ssh-key를 발급하여 Dockerfile에서 clone하기

## 미리 clone하여 Dockerfile에서 copy 하기
1. 본인의 local에서 먼저 해당 repository를 clone 합니다.
```shell
$ git clone https://github.com/hunhoon21/private-repo.git # 본인이 접근 가능한 repository
```
2. Dockerfile에서 COPY 명령어를 추가합니다. 도커파일 위에선 간단한 라이브러리를 설치했습니다.
```dockerfile
FROM nvidia/cuda:10.2-base-ubuntu18.04
RUN apt-get update && apt-get install -y \
    sudo \
    git \
 && rm -rf /var/lib/apt/lists/*
WORKDIR /app/assets/
COPY private-repo /app/assets/private-repo/
```
> [private-repo-with-COPY] ![copy](../imgs/private-repo-with-copy.gif)

* 장 & 단점
  * 매번 미리 clone을 직접 해야해서 해당 repo의 버전관리가 힘듭니다.

## Dockerfile에서 github 계정 & 토큰 정보 추가하여 clone하기
1. `git clone` 명령어에 본인 계정과 token정보를 추가하면 됩니다.
```dockerfile
FROM nvidia/cuda:10.2-base-ubuntu18.04
RUN apt-get update && apt-get install -y \
    sudo \
    git \
 && rm -rf /var/lib/apt/lists/*
WORKDIR /app/assets/
RUN git clone https://username:password@github.com/username/repo_name.git
```

* 장 & 단점
  * 간단한 방법으로 개인이 빠르게 작업하기에는 좋습니다.
  * 소스코드 레벨인 Dockerfile에 본인의 인증 정보를 담아야 하기 때문에 보안에 매우 취약합니다.

## ssh-key를 발급하여 Dockerfile에서 clone하기
1. ssh key를 생성합니다. (기존 가지고 있던 key가 있다면 스킵해도 괜찮습니다.)
2. github 계정에 해당 ssh key를 등록합니다. 빨간 부분을 클릭하여 등록할 수 있습니다.
> [enroll SSH key on github] ![ssh-key](../imgs/ssh-key.jpg)
3. Dockerfile에서 로컬의 private ssh key를 가져오고, ssh clone을 합니다.
```dockerfile
FROM nvidia/cuda:10.2-base-ubuntu18.04
RUN apt-get update && apt-get install -y \
    sudo \
    git \
  && rm -rf /var/lib/apt/lists/*
ADD my-key /root/.ssh/my-key
RUN \
  touch /root/.ssh/known_hosts && \
  ssh-keyscan github.com >> /root/.ssh/known_hosts && \
  chmod 600 /root/.ssh/my-key && \
  echo "IdentityFile /root/.ssh/my-key" >> /etc/ssh/ssh_config
WORKDIR /app/assets/
RUN git clone git@github.com:hunhoon21/private-repo.git
```
> [private-repo-with-SSH-key] ![SSH-key](../imgs/private-repo-with-ssh-key.gif)

* 장 & 단점
  * 항상 ssh-key를 image에 복사해야합니다. 이는 S3 에 올려두어 자동화도 가능합니다.
  * 소스코드 레벨인 Dockerfile에 본인의 인증 정보를 남기지 않을 수 있습니다.
