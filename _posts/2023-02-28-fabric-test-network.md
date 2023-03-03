---
layout: post
title: Hyperledger Fabric (2) - Fabric Test Network
date: 2023-02-28 17:16 +0900
categories: [Blockchain, Hyperledger Fabric]
tags: [blockchain, hyperledger fabric, nodejs]
render_with_liquid: false
---

이전 포스트에서 구축한 개발 환경을 보다 더 세부적으로 살펴보려고 한다. Hyperledger Fabric 폴더의 포스트들은 대부분 [Hyperledger Fabric Tutorials](https://hyperledger-fabric.readthedocs.io/ko/latest/tutorials.html)에서 참고했다.

<br>

## Test Network Component

`fabric-samples/test-network/network.sh`{: .filepath} 파일은 로컬에서 docker 이미지를 사용해 Fabric network를 구축하는 스크립트이다. `./network.sh -h`{: .filepath} 커맨드를 통해 해당 스크립트에 대한 설명을 볼 수 있다.

이전 실행에서 생성됐을 수 있는 컨테이너나 아티팩트를 제거하기 위해 다음 커맨드를 실행한다:

```terminal
$ ./network.sh down
```

{: .nolineno}

그후 네트워크를 띄운다:

```terminal
$ ./network.sh up
```

이 명령어는 두 peer 노드, 하나의 ordering 노드를 생성하고 채널은 생성되지 않는다. 각 노드 그리고 Fabric network와 소통하는 사용자는 네트워크에 참여하기 위해서 organization에 소속되어야 한다. 테스트 네트워크는 Org1과 Org2 두 개의 peer organization을 포함한다. 또한 네트워크의 ordering service를 유지하는 하나의 orderer oragnization을 포함한다. `peer`는 Fabric network의 필수적인 요소이다. peer들은 블록체인 장부를 저장하며 블록체인 원장에 적힌 자산을 관리하는 비즈니스 로직을 포함하는 스마트 컨트랙트를 실행한다.

모든 Fabric network는 `ordering service`도 포함한다. peer들이 거래를 승인하고 블록체인 원장에 블록을 추가할 때 거래의 순서를 정하지 않는다. 분산 네트워크에서 peer들은 거리가 서로 다르며 거래가 체결되었을 때 보이는 원장이 다를 수 있다. 거래 순서에 대한 합의를 하는 것은 오버헤드를 발생시키는 과정이다. ordering service는 peer들이 거래를 승인하고 원장에 입력하는 일에만 집중할 수 있게 한다.

sample network는 orderer organization에 의해 운영되는 단일 노드 **Raft ordering service**를 사용한다. 반면 production network에는 여러 orderer organization이 운영하는 다중 ordering 노드가 있을것이다. 다른 ordering node들은 **Raft consensus algorithm (뗏목 합의 알고리즘)** 을 사용하여 네트워크상 거래 순서를 합의한다.

<br>

## 채널 생성

<br>

앞선 과정으로 peer와 orderer node가 구동되고 있고 이제 Org1과 Org2 사이의 거래를 위해 Fabric 채널을 생성하는 스크립트를 쓸 수 있다. 채널은 특정 네트워크 구성원사이 소통을 위한 private layer이다. 다음 커맨드로 채널을 생성할 수 있으며 `mychannel`이라는 이름으로 자동으로 생성된다. 뒤에 `-c` 플래그를 붙여 채널 이름을 설정할 수 있다. 채널을 생성한 후에 다른 이름으로 채널을 생성함으로써 여러 채널을 열 수 있다:

```terminal
$ ./network.sh createChannel
$ ./network.sh createChannel -c channel1
$ ./network.sh createChannel -c channel2
```

{: .nolineno}

네트워크를 구성하고 채널을 생성하는 것을 한 번에 하려면 `up`과 `createChannel`을 함께 사용하면 된다:

```terminal
$ ./network.sh up createChannel
```

{: .nolineno}

<br>

## 체인코드 실행

<br>

스마트 컨트랙트는 블록체인 원장에 있는 자산을 관리하는 비즈니스 로직을 담고있다. 네트워크 구성원이 실행하는 애플리케이션으로 원장에 자산을 추가, 수정, 거래하는 스마트 컨트랙트를 체결할 수 있다. 애플리케이션은 원장에 있는 데이터를 읽어오기 위해 스마트 컨트랙트 목록을 불러오기도 한다. 거래의 유효성을 확정하기 위해, 스마트 컨트랙트로 체결된 거래는 채널 원장에 등록되기 위해 다수 조직의 서명이 필요하다. 거래에 서명하기 위해 각 조직은 그들의 peer에 스마트 컨트랙트를 불러 실행해야 하며 그 결과로 거래의 결과에 서명하게 된다. 만약 결과가 일관적이고 충분한 수의 서명을 받았다면 거래가 장부에 기록된다. 채널에서 스마트 계약을 실행해야 하는 조직 집합을 정하는 정책을 보증 정책이라 하며 각 체인코드의 정의 부분에 설정된다.

Fabric에서 스마트 컨트랙트는 체인코드로 네트워크에 배포된다. 체인코드는 조직의 peer에 설치되며 그 후 채널에 배포되고, 그 후 거래를 보증하는데 사용되고 블록체인 원장과 연결될 수 있다. 체인코드가 채널에 배포되기 전에, 채널의 구성원들은 체인코드 관리를 인정하는 체인코드 정의에 동의해야 한다. 필요한 수의 조직이 동의하면 체인코드 정의는 채널에 등록되고 사용될 준비가 된다.

`network.sh` 스크립트로 채널 생성을 완료했다면 다음 커맨드로 체인코드를 시작할 수 있다:

```terminal
$ ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

{: .nolineno}

`deployCC` 커맨드는 **asset-transfer (basic)** 체인코드를 `peer0.org1.example.com`과 `peer0.org2.example.com`에 설치하고 채널 명이 명시되어 있다면 해당 채널에, 아니라면 기본 채널명인 `mychannel`에 체인코드를 배포한다. 처음 체인코드를 배포할 때는 스크립트가 체인코드 dependency들을 설치한다. `-l`플래그로 체인코드를 Go, java, typescript, 혹은 javascript로 설치할지를 선택할 수 있다. `fabric-samples`{: .filepath} 디렉토리의 `asset-transfer-basic`{: .filepath} 폴더에서 샘플 체인코드를 볼 수 있다.

## 네트워크 상호작용

테스트 네트워크를 불러온 후에 네트워크와 소통하기 위해 `peer` CLI를 사용할 수 있다. `peer` CLI는 배포된 스마트 컨트랙트를 불러올 수 있고 채널을 업데이트하거나 CLI에서 새로운 스마트 컨트랙트를 설치하고 배포할 수 있다. `text-network`{: .filepath} 에 위치하여 `peer` 바이너리를 CLI 경로에 추가하는 커맨드는 다음과 같다:

```terminal
$ export PATH=${PWD}/../bin:$PATH
```

{: .nolineno}

`fabric-samples`{: .filepath} 폴더의 `core.yaml`{: .filepath} 파일 경로를 가리키는 `FABRIC_CFG_PATH`{: .filepath} 변수도 설정해야 한다:

```terminal
$ export FABRIC_CFG_PATH=$PWD/../config/
```

{: .nolineno}

이제 Org1을 운영할 수 있게 환경변수를 설정한다:

```terminal
# Environment variables for Org1

$ export CORE_PEER_TLS_ENABLED=true
$ export CORE_PEER_LOCALMSPID="Org1MSP"
$ export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
$ export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
$ export CORE_PEER_ADDRESS=localhost:7051
```

{: .nolineno}

`CORE_PEER_TLS_ROOTCERT_FILE`{: .filepath}과 `CORE_PEER_MSPCONFIGPATH`{: .filepath} 환경변수는 `organizations`{: .filepath} 폴더에 있는 Org1 암호자재를 가리킨다.

asset-transfer (basic) 체인코드를 설치하고 시작하는데 `./network.sh deployCC -ccl go`{: .filepath} 명령어를 사용했다면, 다음 명령어로 원장에 초기 자산 목록을 집어넣기 위해 (Go) 체인코드의 `InitLedger`{: .filepath} 함수를 불러올 수 있다:

```terminal
$ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
```

{: .nolineno}

성공했다면 `status:200`{: .filepath}라는 문구를 확인할 수 있다. 이제 CLI에서 원장에 쿼리문을 사용할 수 있으며 다음 커맨드로 채널 원장에 추가된 자산 목록을 불러올 수 있다:

```terminal
$ peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

{: .nolineno}

장부에 있는 자산을 거래하거나 변경할 때 체인코드가 실행되며, 다음 커맨드를 사용한다:

```terminal
$ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```

{: .nolineno}

asset-transfer (basic) 체인코드의 보증 정책에서 Org1과 Org2의 서명이 있어야 거래가 성사될 수 있도록 하고 있기 때문에 명령어에서 `--peerAddresses`{: .filepath} flag를 통해 Org1과 Org2 모두를 타겟으로 체인코드를 실행해야 한다. 네트워크에 TLS가 유효하므로 `--tlsRootCertFiles`{: .filepath} flag로 두 peer 모두 TLS 증명서를 참조해야 한다.

체인코드를 실행한 후 블록체인 원장의 자산이 어떻게 변했는지 확인하기 위해 다른 쿼리문을 사용한다. 이미 Org1 peer에 쿼리문을 사용해 봤으므로 Org2 peer에 실행되는 체인코드를 질의해보겠다. 이전과 마찬가지로 Org2로 작동시키기 위한 환경변수를 설정해준다:

```terminal
# Environment variables for Org2

$ export CORE_PEER_TLS_ENABLED=true
$ export CORE_PEER_LOCALMSPID="Org2MSP"
$ export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
$ export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
$ export CORE_PEER_ADDRESS=localhost:9051
```

이후 Org1에와 마찬가지로 쿼리문을 실행하면 된다.

> 네트워크를 종료하려면 `./network.sh down`{: .filepath} 명령어를 실행하면 된다. 이 명령어는 암호자재, 체인코드 이미지와 컨테이너, 노드, 채널 아티팩트와 이전에 실행되고 있던 docker volume을 지운다.
> {: .prompt-tip}
