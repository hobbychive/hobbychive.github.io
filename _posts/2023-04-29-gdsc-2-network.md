---
layout: post
title: GDSC(2) - Network
date: 2023-04-29 15:01 +0900
categories: [GDSC]
tags: [gdsc yonsei, network, osi 7 layers]
render_with_liquid: false
---

Network OSI 7계층에 대한 포스트이다.

## OSI 7 Layers

**OSI(Open Systems Interconnection Reference Model) 7 계층**은 국제 표준화 기구(ISO)에서 네트워크에서 통신이 일어나는 과정을 7단계로 나눈 네트워크 표준 모델을 의미한다.

![Desktop View](/assets/img/posts/osi_7_layers.png){: w="700" h="400" }

데이터를 전송하는 쪽에서는 Application layer부터 Physical layer까지 데이터가 내려보내지며, 각 layer를 지날 때 마다 헤더가 붙는다. 데이터를 수신하는 쪽에서 각 layer를 통과하며 header를 해석하고 올바른 목적지에서 데이터가 수신자에게 전달된다.

현재 산업 표준은 TCP/IP 모델로, 위 7,6,5 layer를 Application layer 하나로 통합한 모델이다.(1,2 layer를 묶어 4 layers로 표현하기도 한다.) 이 포스트에서는 TCP/IP layer 7 부터 layer 1 까지 Top down으로 살펴본다.

<br>

## Application layer (Layer 7)

<br>

응용 프로그램과 통신 프로그램간의 인터페이스를 제공한다. 통신 관계자들끼리 알아볼 수 있는 메시지(데이터)로 변환하고 전송 계층으로 전달하며, 변환 규칙을 프로토콜 이라고 한다.

Application layer의 프로토콜에는 다음과 같은 것들이 있다:

### HTTP (**H**yper**T**ext **T**ransfer **P**rotocol)

웹 서비스에서 웹 서버 및 웹 브라우저간 데이터 전송을 위한 프로토콜이다. 이미지, 비디오, 음성 등 거의 모든 형식의 데이터 전송이 가능하다. 포트 번호는 80번을 사용한다. (포트에 대해서는 Layer 4 에서 다룰 예정) 클라이언트와 서버 간 HTTP 메세지를 주고받으며 통신한다.

### Telnet

네트워크 관리를 할 수 있는 프로토콜이다. 스위치 장비 설정을 통해 운격으로 스위치에 연결된 컴퓨터 ip, Mac address를 관리하기 용이하다.

일반적으로 포트 23번을 사용한다.

### SSH (**S**ecure **SH**ell)

원격 호스트에 접속하기 위해 사용되는 보안 프로토콜이다. 기존 원격 접속은 Telnet을 활용했으나 암호화 제공을 하지 않아 보안에 취약하다는 단점이 있었다.

사용자와 서버는 각각의 **키**를 보유하고 있고, 키를 이용해 연결 상대를 인증한다. 인증이 된 후에 데이터를 주고받게 되므로 보안상 안전하다고 할 수 있다. 키를 생성하는 방식에는 **'대칭키'**와 **'비대칭키(공개키)'**가 있다.

포트는 기본적으로 22번을 사용한다.

### HTTPs (**HTTP s**ecure)

HTTP의 암호화된 버전으로, 클라이언트와 서버 간의 커뮤니케이션을 암호화하기 위하여 `SSL`이나 `TLS`를 사용한다.

포트는 443번을 사용한다.

#### SSL (**S**ecure **S**ockets **L**ayer)

보안 소켓 계층 혹은 디지털 인증서라 불리며, 브라우저와 서버 사이의 암호화된 연결을 수립하기 위한 표준 기술이다.

#### TLS (**T**ransport **L**ayer **S**ecurity)

SSL의 향상된(더 안전한) 버전.

Google에서 웹 전반의 보안을 개선하기 위해 모든 사이트에 HTTPs를 요구했고, SSL 보안 사이트에 대해 높은 검색 순위를 매겼다. 2018년부터는 SSL 인증서가 없는 사이트에 불이익을 주기 시작했다. (안전하지 않음 표시)

### DHCP (**D**ynamic **H**ost **C**onfiguration **P**rotocol)

호스트의 IP주소와 TCP/IP 프로토콜의 기본 설정을 클라이언트에 자동적으로 제공해주는 프로토콜이다. 컴퓨터의 네트워크 부팅 과정에서 동적으로 네임 서버 주소, IP주소, 게이트웨이 주소를 할당해 주기 때문에 IPv4에서 IP 주소가 부족한 문제점을 극복할 수 있다.

다만 해당 과정이 DHCP 서버에 의존하므로 서버가 다운되면 IP 할당이 잘 이루어지지 않는 단점이 있다.

### LOCO

카카오톡에서 사용하는 Application layer protocol 이며, 카카오톡에 트래픽이 많이 몰리며 기존 HTTP 방식의 한계가 드러나 대체재로 개발된 프로토콜이다. (프로젝트명 겁나 빠른 황소) 패킷 사이즈 경량화, 푸시 시스템 구조 최적화, 백엔드 시스템 성능 개선 등의 특징이 있다.

TCP 세그먼트는 40Byte의 플래그와 헤더가 붙게 되는데, 작은 메세지가 전송될 때에도 붙기 때문에 비효율적이다. 이에 대한 이슈를 해결하기 위해 나온 Nagel 알고리즘은 작은 메세지가 충분히 모여 일정 용량 이상이 될 때까지 버퍼에 저장한 후 전송하는 방식인데, 이는 즉각적인 전송이 이루어져야 하는 메신저에는 맞지 않았다. 카카오톡은 앞선 문제들을 LOCO 프로토콜로 해결했다.

<br>

## Transport layer (Layer 4)

<br>

프로세스 사이의 logical communication을 제공한다.
sender에서는 segment단위로 메세지를 잘라서 전송하고, receiver에서는 segment를 합쳐 application layer에 전달하는 역할을 한다. 각 segment를 패킷이라고 한다. 대표적으로 두 가지 프로토콜이 있다.

### TCP (**T**ransmission **C**ontrol **P**rotocol)

sender와 receiver 간 handshake 가 일어난 후 데이터를 전송한다. data loss가 일어나지 않고(reliable), 보낸 순서대로 receiver에서 수신할 수 있다.

연결이 이루어져야 데이터 전송이 일어나기 때문에 다소 느리다.

### UDP (**U**ser **D**atagram **P**rotol)

연결을 고려하지 않고 데이터 전송을 수행한다. 데이터 전송이 빠르지만 손실이 일어날 수 있다(unreliable).

스트리밍 서비스와 같이 데이터 손실이 다소 있더라도 전송 속도가 중요한 transmission에서 사용된다.

### Port

TCP/UDP와 같은 transmission layer protocol은 프로세스를 식별하기 위해 port라는 단위를 사용한다. 운영 체제 통신의 종단점이며, 번호로 구별된다.

포트 번호는 크게 세 종류로 구분된다.

- 0 ~ 1023 : well-known port
- 1024 ~ 49151 : registered port
- 49152 ~ 65535 : dynamic port

### iptables

리눅스 상에서 방화벽을 설정하는 도구이다. 특정 조건을 가지고 있는 패킷에 대해 허용, 차단 등을 지정할 수 있고 특정 조건 등을 통해 다양한 방식의 패킷 필터링과 처리 방식을 지원한다.

<br>

## Network layer (Layer 3)

데이터 패킷에 주소를 붙여 경로를 설정하고(라우팅) 다른 네트워크로부터 패킷을 수신하는 역할을 한다.

### IP (**I**nternet **P**rotocol)

인터넷 네트워크에서의 데이터 통신 규약이다. IP 통신에 필요한 주소를 네트워크에 연결된 호스트에 부여하여 각 장치를 식별한다. IPv4, IPv6 두 가지 체계가 있다.

### routing

라우팅은 네트워크에서 경로를 선택하는 프로세스로, 미리 정해진 규칙을 사용하여 최상의 경로를 선택하는 프로세스이다. 이를 수행하는 장치가 라우터이며 소스에서 대상으로 이동하는 데이터의 경로를 결정하고, 데이터를 전달하며, 로드 밸런싱을 통해 데이터 손실로 인한 오류를 줄인다.

### ICMP (**I**nternet **C**ontrol **M**essage **P**rotocol)

IP 패킷을 처리할 때 발생되는 문제를 알려주는 프로토콜이다. IP에는 통신에서의 에러에 대한 처리 방법이 명시되어 있지 않으므로, ICMP를 통해 도착지 호스트가 없거나, 포트가 닫혀있는 등의 에러 상황이 발생할 때 sender에게 에러 정보를 알려줄 수 있다.

에러 메세지에는 크게 두 가지가 있다:

- Destination Unreachable
- Time Exceeded

윈도우의 ping 이 ICMP 프로토콜을 이용한 대표적인 예라고 할 수 있다.

<br>

## Link layer (Layer 2)

<br>

하나의 노드(스위치)에서 인접한 노드까지 연결한 링크로 데이터그램을 옮기는 layer이다. network layer에서 받은 데이터그램을 frame 단위로 쪼개어 전송한다. 데이터 운송 수단에 대한 프로토콜이므로 실제 기기가 존재해야 한다.

### NIC (**N**etwork **I**nterface **C**ard)

LAN에 연결 지점을 제공하기 위해 컴퓨터에 설치하는 어댑터이다. 사용 규격에 따라 종류를 나눌 수 있고, 현재 Ethernet 방식 NIC가 가장 많이 쓰인다.

버스 인터페이스에 따라 종류를 나눌 수도 있는데, PCIe를 지원하는 마더보드에 부착할 수 있는 방식이 가장 많다고 할 수 있다.

### Ethernet

LAN 환경에서 절대 다수를 차지하는 네트워크 구성 방식이다. 유선 방식이며, 케이블의 종류가 발전함에 따라 속도가 증가해왔다.

<br>

## Physical layer (Layer 1)

윗 계층에서 전달된 프레임을 실제 cable을 통해 전송한다.

### LAN (**L**ocal **A**rea **N**etwork)

지역 네트워크이다. 라우터 / 스위치를 통해 연결되어있으며, 학교, 회사, 집 등에서 장비를 서로 연결한 것이라 볼 수 있다.

### WAN (**W**ide **A**rea **N**etwork)

LAN과 LAN 사이를 연결한 네트워크이다.

![Desktop View](/assets/img/posts/lan_wan.png){: w="500" h="600" }

### UTP / STP

랜선의 종류이며, 케이블 속 구리선의 실드 구조에 대한 명칭이다. TP 는 **Twisted Pair**를 의미하며, 두 가닥의 케이블이 꼬여있다는 의미이다.

#### UTP (**U**nshielded **TP**)

두 선이 한 쌍으로 꼬여있는 케이블이다.

![Desktop View](/assets/img/posts/utp.png){: w="500" h="200" }

#### STP (**S**hielded **TP**)

알루미늄 호일 등으로 실드가 되어 있는 케이블이다. 소재에 따라 FTP, S-STP등으로 나뉜다.

![Desktop View](/assets/img/posts/stp.png){: w="500" h="200" }

<br>

## Wireshark

<br>

네트워크 패킷을 캡처하고 분석하는 도구이다.

네트워크 구간 사이로 돌아다니는 패킷을 수신하여 저장하며, `PCAP(Packet CAPture)` 파일 포맷으로 저장된다.
