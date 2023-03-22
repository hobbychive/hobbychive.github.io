---
layout: post
title: Hyperledger Fabric (1-1) - Ubuntu Fabric 개발 환경 세팅하기
date: 2023-03-15 16:07 +0900
categories: [Blockchain, Hyperledger Fabric]
tags: [blockchain, hyperledger fabric, nodejs, ubuntu]
render_with_liquid: false
---

Ubuntu 환경에서 Hyperledger Fabric 개발 환경을 구축한다. 해당 포스트에서는 ssh를 사용하여 ubuntu v22.04.1 서버에 접속하고 Fabric LTS 버전인 v2.2.10을 설치한다.

<br>

## Prerequisites

<br>

### Docker 설치하기

```terminal
$ sudo apt update
$ sudo apt-get -y install docker-compose
```

{: .nolineno }

Docker를 설치한 후 실행하면 된다:

```terminal
$ sudo systemctl start docker
```

{: .nolineno}

### git, curl 설치하기

```terminal
$ sudo apt-get install git
$ sudo apt-get install curl
```

{: .nolineno}

### golang 설치

패브릭 code는 golang으로 구성되어 있으므로 golang을 설치해준다. 설치를 한 후 환경변수까지 설정해준다:

```terminal
curl -O https://dl.google.com/go/go1.18.3.linux-amd64.tar.gz
tar xvf go1.18.3.linux-amd64.tar.gz
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

{: .nolineno}

bash 프로필에 환경변수를 추가해준다:

```terminal
$ nano ~/.bashrc

# 다음을 .bashrc파일 끝에 추가
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

{: .nolineno}

Hyperledger Fabric을 설치할 디렉토리를 새로 생성해준다.

```terminal
$ mkdir fabric-network
```

{: .nolineno}

### node, npm 설치

node v14.13.1, npm v6.4.1로 설치했다. 버전이 높아서 실행이 안되는 경우가 있으므로 이 버전을 다운받는 것을 추천한다.

<br>

## Hyperledger Fabric v2.2.10 설치

<br>

curl을 사용하여 Github에 공개되어있는 hyperledger/fabric 레포지토리의 소스코드를 다운받는다. [v2.2.10](https://github.com/hyperledger/fabric/releases/tag/v2.2.10)에서 직접 다운받는것도 가능하다:

```terminal
# 명령어 형식은 다음과 같다
# curl -sSL https://bit.ly/2ysbOFE | bash -s -- <fabric_version> <fabric-ca_version>
$ curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.10
```

{: .nolineno }

> 버전을 명시해주지 않는다면 릴리즈 최신 버전으로 설치가 되게 된다.
> {: .prompt-info }

바이너리 설치가 완료된 후 clone된 fabric-samples 폴더로 이동한다:

<br>

## 설치 확인

<br>

설치가 제대로 됐는지 확인하기 위해 channel을 생성해본다:

```terminal
$ cd fabric-samples/test-network
$ ./network.sh up createChannel -ca -c mychannel -s couchdb
```

{: .nolineno }

<br>

트러블 슈팅 과정은 [Hyperledger Fabric (1) 포스트](https://hobbychive.github.io/posts/apple-silicon-mac%EC%97%90-fabric-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0/)를 확인하기 바람.
