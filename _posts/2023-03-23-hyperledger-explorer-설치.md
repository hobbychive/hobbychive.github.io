---
layout: post
title: Hyperledger Fabric (4) - Hyperledger Explorer 설치
date: 2023-03-23 15:15 +0900
categories: [Blockchain, Hyperledger Fabric]
tags: [blockchain, hyperledger fabric, hyperledger explorer]
render_with_liquid: false
---

이번 포스트에서는 Hyperledger에서 거래 내역 등을 볼 수 있는 유용한 툴인 Explorer를 설치한다.

<br>

## Hyperledger Explorer

<br>

Hyperledger Explorer는 블록, 트랜잭션 및 관련 데이터, 네트워크 정보, 체인코드와 같은 블록체인 네트워크 구성 요소를 모니터링 할 수 있는 툴이다. 2023.03 기준 Fabric v2.3 이후 버전은 지원되지 않는듯 하며, 현 LTS 버전인 v.2.2.10 환경에 적용해 보도록 한다.

<br>

## Prerequisites

<br>

### Node.js

버전마다 지원되는 Node.js 버전이 다르므로 [hyperledger-labs/blockchain-explorer](https://github.com/hyperledger-labs/blockchain-explorer) 레포지토리의 Release note를 참고하자.

### PostgreSQL

PostgreSQL v9.5 이후 버전을 설치한다:

```terminal
$ sudo apt-get update
$ sudo apt-get install postgresql postgresql-contrib
$ service postgresql restart
$ pg_lsclusters
```

{: .nolineno}

### jq

jq를 설치한다:

```terminal
$ sudo apt-get install jq
```

{: .nolineno}

Hyperledger Fabric이 설치되어 있다 가정하므로 다른 필수 유틸 설치는 생략한다.

### 계정 생성

root 계정을 사용하는것을 피하기 위해 User를 생성해준다:

```terminal
$ sudo adduser explorer
$ sudo usermode -aG sudo explorer
$ su - explorer
```

{: .nolineno}

### Explorer 코드

git repository에서 clone해준다:

```terminal
$ cd
$ git clone https://github.com/hyperledger/blockchain-explorer.git
```

{: .nolineno}

<br>

## 데이터베이스 생성

<br>

Explorer가 필요료 하는 기본 데이터베이스를 생성해준다. createdb.sh 파일을 실행하는 것으로 수행할 수 있다:

```terminal
$ cd blockchain-explorer/app/persistence/fabric/postgreSQL
$ chmod -R 775 db/
$ cd db
$ ./createdb.sh
```

{: .nolineno}

<br>

## Explorer configuration

<br>

### fabric network

**fabric network를 올리고 채널까지 생성한다.** fabric-network의 비공개 key path를 복사해두고, explorer의 튜토리얼 네트워크 설정 파일인 first-network.json에서 비공개 key 파일 path로 대체한다:

```terminal
$ ls ~/fabric-samples/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/priv_sk
# key 확인
$ cd blockchain-explorer/app/platform/fabric/connection-profile
$ vi first-network.json
# 파일 수정
```

{: .nolineno}

### Explorer 설치

설치 script를 실행한다:

```terminal
$ cd blockchain-explorer
$ ./main.sh install
$ ./start.sh
```

{: .nolineno}

그 후 `docker-compose up -d`{: .filepath} 명령어를 실행한다.

이 과정까지 마쳤으면 localhost:8080으로 접속이 가능하며, 기본 id / pw는 `exploreradmin / exploreradminpw` 이다.

![Desktop View](/assets/img/posts/hyperledger_explorer.png){: w="700" h="400" }

`docker-compose down -v`로 종료할 수 있다.

이 포스트는 21시간님의 블로그 포스트 [\[Hyperledger Fabric\] 하이퍼 레저 패브릭 & 익스플로러 설치 방법](https://tmjb.tistory.com/15)과 [hyperledger explorer github repository](https://github.com/hyperledger-labs/blockchain-explorer)를 참고하였다.
