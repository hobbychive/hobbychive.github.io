---
layout: post
title: Hyperledger Fabric (3) - Fabric CA and MSP
date: 2023-03-03 19:52 +0900
categories: [Blockchain, Hyperledger Fabric]
tags: [blockchain, hyperledger fabric, ca, certificate authority, msp]
render_with_liquid: false
---

이 포스트는 폐쇄형 블록체인인 Hyperledger Fabric에서 허락받은 참여자만을 블록체인 네트워크에 참여시키기 위해 사용하는 애플리케이션인 **Fabric CA (Certificate Authority)** 와 CA가 제공하는 정보를 모은 **MSP (Membership Service Provider)** 에 대해 정리한다.

해당 정보들은 HoCheol Nam 블로그의 [CA / MSP](https://hcnam.tistory.com/23) 문서와 [Hyperledger Fabric Reference](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html)를 참고하였다.

<br>

## Certificate Authority (CA)

<br>

Hyperledger fabric CA는 hyperledger fabric 어플리케이션이며 블록체인 네트워크에 참여를 허가하는 정보를 제공한다. 이 정보들이 모여 MSP를 구성하며 채널 접근 권한을 관리하게 된다. PKI(공개키 구조 기반)를 사용한다.

<br>

## Membership Service Provider (MSP)

<br>

MSP에는 **root CA (rCA)** 와 **intermediate CA (iCA)** 로 구성되며 iCA인증서는 rCA 혹은 iCA에 의해 서명되어야 한다.

MSP는 Global과 Local로 구분되며 Global은 Network와 channel, Local은 peer와 orderer 등으로 구분된다.

### Global MSP

Global MSP는 블록체인 네트워크와 채널에 참여한 모든 구성원에 대해 적용된다. **Network MSP**는 네트워크에 참여하는 organization의 MSP를 식별하여 블록체인 네트워크 참여 여부를 결정한다. **Channel MSP**는 특정 채널에 참여하는 조직을 식별한다.

### Local MSP

Local MSP는 MSP를 통해 식별이나 제어 가능한 대상이 탑재된 노드에 한정되며, **peer MSP**는 peer의 파일시스템에 탑재되고 Channel MSP와 비슷한 역할을 수행한다. **orderer MSP**는 OSN에 탑재되어 조직에서의 신뢰 가능한 노드를 식별한다.

```markdown
![Desktop View](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/_images/fabric-ca.png){: w="700" h="400" }
_출처: hyperledger fabric reference_
```

{: .nolineno}

<br>

## 네트워크와 CA 연결

<br>

### using cryptogen tool

`network.sh`{: .filepath} 스크립트는 peer와 ordering 노드를 생성하기 전에 네트워크를 배포하고 운영하는데 필요한 암호자재를 생성한다. 스크립트에서 자동으로 인증서와 키를 생성하는데 필요한 cryptogen 툴을 사용하며 이 툴은 개발과 테스트, 그리고 필요한 암호자재를 빠르게 생성한다. `./network.sh`{: .filepath} 커맨드를 실행할 때 cryptogen 툴이 Org1, Org2, Orderer Org 각각에 인증서와 키를 생성하는 것을 볼 수 있다.

```shell
creating Org1, Org2, and ordering service organization with crypto from 'cryptogen'

/Usr/fabric-samples/test-network/../bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################

##########################################################
############ Create Org1 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output=organizations
org1.example.com
+ res=0
+ set +x
##########################################################
############ Create Org2 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output=organizations
org2.example.com
+ res=0
+ set +x
##########################################################
############ Create Orderer Org Identities ###############
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
+ res=0
+ set +x
```

{: .nolineno}

### using CA

하지만 test network 스크립트는 CA를 사용하여 네트워크를 구축하는 옵션도 제공한다. production network에서 각 organization은 조직에 소속되는 ID를 생성하는 CA를 운영한다. 해당 CA에서 생성한 ID는 모두 같은 신뢰점을 공유한다. cryptogen 툴을 사용하는 것보다 오래 걸리긴 하지만 CA를 사용하여 네트워크를 구축하는 것은 산업에서 네트워크가 배포되는 방법을 보여준다. CA를 배포하는 것은 클라이언트 ID를 Fabric SDK에 등록하고 애플리케이션에 인증서와 private key를 생성할 수 있도록 한다. CA를 사용하여 네트워크를 구축하고 싶다면 먼저 네트워크를 down시킨 후 CA flag를 붙여 네트워크를 불러온다:

```terminal
$ network.sh up -ca
```

{: .nolineno}

커맨드를 실행하면 세 CA를 생성하는 스크립트를 볼 수 있다:

```shell
##########################################################
##### Generate certificates using Fabric CA's ############
##########################################################
Creating network "net_default" with the default driver
Creating ca_org2    ... done
Creating ca_org1    ... done
Creating ca_orderer ... done
```

{: nolineno}

test network는 Fabric CA 클라이언트를 사용하여 각 organization의 CA로 노드와 user ID를 등록한다. 그 후 각 ID에 대한 MSP 폴더를 생성하는 enroll 커맨드를 실행한다. MSP 폴더에는 각 ID에 대한 증명서와 private key가 있으며 CA를 운영하는 organization에서의 해당 ID의 역할과 membership을 수립한다. Org1 admin user에 대한 MSP 폴더를 다음 커맨드를 입력하여 확인해 볼 수 있다:

```terminal
$ tree organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/
```

{: .nolineno}

커맨드 실행 결과로 MSP 폴더 구조와 configuration 파일을 확인할 수 있다:

```shell
organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/
└── msp
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    │   └── localhost-7054-ca-org1.pem
    ├── config.yaml
    ├── keystore
    │   └── 58e81e6f1ee8930df46841bf88c22a08ae53c1332319854608539ee78ed2fd65_sk
    ├── signcerts
    │   └── cert.pem
    └── user
```

{: .nolineno}

admin user에 대한 증명서는 `signcerts`{: .filepath} 폴더에 있고 private key는 `keystore`{: .filepath} 폴더에서 찾을 수 있다.

cryptogen tool 을 사용하는 방법과 Fabric CA를 사용하는 방법 모두 `organizations`{: .filepath} 폴더에 각 organization에 대한 암호 자재를 생성한다. 네트워크를 설정하는데 쓰인 커맨드는 `organizations/fabric-ca`{: .filepath} 디렉토리의 `registerEnroll.sh`{: .filepath} 스크립트에서 확인할 수 있다.
