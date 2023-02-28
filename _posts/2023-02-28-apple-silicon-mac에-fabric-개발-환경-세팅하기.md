---
layout: post
title: Hyperledger Fabric (1) - Apple Silicon Mac에 Fabric 개발 환경 세팅하기
date: 2023-02-28 15:04 +0900
categories: [Blockchain, Hyperledger Fabric]
tags: [blockchain, hyperledger fabric, nodejs, apple silicon]
render_with_liquid: false
---

Hyperledger Fabric을 활용한 개발 프로젝트를 위해 M1 Max 맥북에 개발 환경을 세팅하는데, Fabric LTS 버전인 2.2.X 버전, 최신 릴리즈 버전인 2.4.8이 모두 Apple Silicon arm64 아키텍쳐에서 잘 설치되지 않는 어려움이 있었다. 여러 문서를 찾아보며 이를 해결한 방법을 포스트한다.

<br>

## Prerequisites

<br>

Mac 환경을 기준으로 설명하므로, Git 설치는 따로 설명하지 않겠다. 우선 Docker를 설치해야 한다. 여기에서는 Homebrew를 이용해 설치한다.

- Docker 설치하기

```shell
$ brew install --cask docker
```

{: .nolineno }

Docker를 설치한 후 실행하면 된다.

<br>

## Hyperledger Fabric v2.5.0-beta 설치

<br>

Github에 공개되어있는 hyperledger/fabric 레포지토리에서 [v2.5.0-beta](https://github.com/hyperledger/fabric/releases/tag/v2.5.0-beta)의 소스코드를 다운받는다. v2.5.0-beta에서 멀티 아키텍쳐를 지원하므로, Apple Silicon chip 에서도 Hyperledger 환경을 구축할 수 있게 된다. 터미널에서 Fabric binaries를 설치할 폴더로 이동하여 다운로드 받은 소스코드의 scripts/bootstrap.sh를 실행해야 한다. 이 때 fabric 과 fabric-ca 버전을 명시해주어야 하며, 실행 코드는 다음과 같다:

```terminal
$ [다운받은폴더경로]/scripts/bootstrap.sh | bash -s -- 2.5.0-beta 1.5.6-beta3
```

{: .nolineno }

> 버전을 명시해주지 않는다면 릴리즈 최신 버전인 2.4.8로 설치가 되게 된다.
> {: .prompt-info }

바이너리 설치가 완료된 후 clone된 fabric-samples 폴더로 이동한다:

```terminal
cd fabric-samples
```

{: .nolineno }

docker에서 fabric-nodeenv와 fabric-javaenv를 pull 해주는데, 이 때 플랫폼을 amd64로 명시해주어야 자동으로 인식되는 arm64와 충돌하지 않는다:

```terminal
docker pull --platform amd64 hyperledger/fabric-nodeenv:2.5
docker pull --platform amd64 hyperledger/fabric-javaenv:2.5
```

{: .nolineno }

<br>

## 테스트 네트워크 구축

<br>

테스트 네트워크를 구축하기 위해 test-network 폴더로 이동한다:

```terminal
cd test-network
```

{: .nolineno }

network.sh를 실행시켜 네트워크를 시작하고 채널을 생성한다:

```terminal
./network.sh up createChannel -c mychannel -ca
```

{: .nolineno }

체인코드를 배포한다. 언어를 선택할 수 있고, 여기서는 node.js로 하기 위해 javascript를 선택했다:

```terminal
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript -ccl javascript
```

{: .nolineno }

환경변수를 설정해주고, Invoke한다:

```terminal
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
# Org1 설정
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

{: .nolineno }

```terminal
peer chaincode invoke \
  -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  --tls \
  --cafile $ORDERER_CA\
  -C mychannel \
  -n basic \
  --peerAddresses localhost:7051 \
  --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
  --peerAddresses localhost:9051 \
  --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
  -c '{"function":"InitLedger","Args":[]}'
```

{: .nolineno }

result: status:200 이 터미널에 뜨면 된 것이고, 배포한 체인코드를 실행하여 블록체인 정보를 읽어올 수 있다:

```terminal
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

{: .nolineno }

정보가 확인되면 성공한 것으로, 네트워크를 종료한다.

```terminal
./network.sh down
```

{: .nolineno }

이 포스트는 rivernine.log의 [Hyperledger Fabric v2.2 네트워크 구축하기](https://velog.io/@rivernine/Hyperledger-Fabric-v2.2-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0) 포스트와 hyperledger/fabric 레포의 [Issue #3702](https://github.com/hyperledger/fabric/issues/3702)를 참고하였다.
