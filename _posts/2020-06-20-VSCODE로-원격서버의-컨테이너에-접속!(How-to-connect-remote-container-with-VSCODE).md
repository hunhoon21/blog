안녕하세요. 오늘은 원격서버에서 킨 도커 컨테이너에 로컬에서 킨 VSCODE로 접속해보는 방법에 대해 알아보도록 하겠습니다. 해당 글은 [여기](https://florian-kriegel.de/blog/?p=234) 를 참고했음을 알립니다. 여러분의 삽질에 도움이 되기를 바랍니다!

### 선수조건

위 작업을 위해서는 먼저 로컬의 VSCODE에 extension 두가지가 필요합니다.

* remote
* Docker

### Local VSCODE로 Remote Container에 접속하는 과정

위 작업은 세가지로 이루어졌습니다.

1. Local과 Remote 서버의 Container 연결
2. Local의 환경변수에 $DOCKER_HOST 추가
3. Local VSCODE의 settings.json에 "docker.host" 추가

#### 1. Local과 Remote 서버의 Container 연결

```shell
$ ssh -NL localhost:7769:/var/run/docker.sock hooncheol.shin@182.@@@.@@@.@@@
```

* 내 로컬에서 사용하지 않는 포트와 원격 서버의 running container 정보를 연결시켜 주는 작업입니다.
* 제 경우에는 로컬의 `7769` 라는 포트 번호를 `182.@@@.@@@.@@@` 원격 서버의 ruuning docker 소켓(`/var/run/docker.sock`) 를 연결시켜 주었습니다.

#### 2. Local의 환경변수에 $DOCKER_HOST 추가

```shell
$ DOCKER_HOST='tcp://localhost:7769'
```

* 로컬(저의 경우는 맥)내 $DOCKER_HOST 라는 환경변수에 내가 사용한 포트번호를 알려줍니다.
* VSCODE에서 저 포트를 타고 원격의 소켓에 접근할 수 있게됩니다.

#### 3. Local VSCODE의 settings.json에 "docker.host" 추가

```json
# in settings.json

{
	...,
	"docker.host": "tcp://localhost:7769"
}
```

* VSCODE에도 `"docker.host"` 정보를 알려줍니다.

### Local VSCODE에서 Remote Container에 attach!

이후에는 vscode docker extension에서 원격의 container에 attach 할 수 있습니다.

>  ![remote_container](../imgs/remote_container.png)