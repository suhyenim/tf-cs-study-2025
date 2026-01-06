## [NET-082] OSI 7계층에 대해 설명해 주세요.

<details>
<summary>정답</summary>

네트워크 통신 과정을 7개의 단계로 표준화한 모델

데이터 송신 가정

- L7에서 L5에서 Data를 생성하고 암호화 및 압축을 진행
- L4에서 Port 번호를 추가해 Segment로 변환
- L3에서 IP 주소를 추가해 Packet으로 변환
- L2에서 MAC 주소를 추가해 Frame으로 변환
- L1에서 전기/광신호로 변환해 Bit로 변환해 매체로 송신

</details>

<details>
<summary>개념</summary>

### OSI 7계층을 나눈 이유

각 계층이 독립적인 기능을 수행해 통신 장애 발생 시 원인 파악이 쉬움

### L7 (Application) 
> 사용자 인터페이스 및 네트워크 서비스 제공

프로토콜: HTTP, FTP, SMTP, DNS

### L6 (Presentation)
> 전송하는 데이터 표현 방식 결정 Ex. 데이터 변환, 압축, 암호화와 복호화

프로토콜: GIF, JPEG, ASCII 등

### L5 (Session)
> 응용 프로세스 간의 세션 관리 (연결 유지 및 종료)

TCP/IP 세션을 만들고 없애는 역할


### L4 (Transport)
> 종단 간(End-to-End) 신뢰성 있는 데이터 전송 (TCP, UDP). 데이터 전송에 Port 번호 사용

송신자와 수신자 간 신뢰성 있고 효율적인 데이터 전송을 위해 오류 검출 및 복구, 흐름 제어와 중복 검사 수행

전송 단위: Segment(TCP), Datagram(UDP)

프로토콜: TCP, UDP

### L3 (Network)
> 최적의 경로 설정 및 패킷 전달. IP 주소 활용

전송 단위: Packet

장비: Router, L3 Switch

프로토콜: IP, ICMP

### L2 (Data Link)
> 물리적 매체를 통한 인접 노드 간의 데이터 전송. MAC 주소로 통신

송수신되는 정보의 오류와 흐름을 관리해 정보를 안전하게 전달하고 통신에서의 오류를 찾아 재전송

전송 단위: Frame

장비: Ethernet, Bridge, Switch

프로토콜: MAC

### L1 (Physical)
> 물리적 매체를 통해 비트(0, 1) 신호를 전송 

전송 단위: Bit

장비: 케이블, 허브, 리피터

</details>

## [NET-083] Transport Layer와, Network Layer의 차이에 대해 설명해 주세요.

<details>
<summary>정답</summary>

두 레이어의 가장 큰 차이는 통신의 범위

- Network Layer(L3)는 **호스트와 호스트 사이(컴퓨터 간)** 경로를 찾는 라우팅에 집중하며 IP 주소 사용

- Transport Layer(L4)는 **프로세스와 프로세스 사이(애플리케이션 간)** 신뢰성 있는 데이터 전송에 집중하며 Port 번호 사용

</details>

<details>
<summary>개념</summary>

### Network Layer (L3)

통신범위: Host-to-Host (컴퓨터 간 통신)

식별주소: IP 주소 기반

데이터 단위: Packet

핵심 역할: 최적의 경로 설정 (Routing), Best-Effort(비신뢰성)

### Transport Layer (L4)

통신범위: Process-to-Process (앱 간 통신)

식별주소: Port 주소 기반

데이터 단위: Segment, Datagram

핵심 역할: 신뢰성 보장, 흐름 및 혼잡 제어

</details>

## [NET-084] L3 Switch와 Router의 차이에 대해 설명해 주세요.

<details>
<summary>정답</summary>

두 장비 모두 IP 정보를 바탕으로 패킷 라우팅

BUT 목적과 처리 방식이 다름
- L3 스위치는 내부 LAN 환경에서 하드웨어 방식으로 빠른 스위칭을 수행하는데 최적화
- 라우터는 WAN 연결 및 소프트웨어 방식으로 정교한 경로 제어와 보안 기능에 유리

</details>

<details>
<summary>개념</summary>

### Switch(L2)와 Router(L3)

일반적으로 Switch는 L2 계층 장비

Switch는 네트워크 내에서 데이터 패킷을 전달하고, Router는 서로 다른 네트워크 간 데이터 전달

### L3 Switch

일반적으로 Switch는 L2 계층 장비이지만, Router와 Switch 역할을 모두 할 수 있는 L3 Switch 존재

등장 배경

- Router 속도 한계 (Hardware vs Software)
    - 과거 서로 다른 네트워크 간 통신 시 반드시 Router를 거쳐야 했음
    - 초기 Router는 모든 패킷 처리를 소프트웨어(CPU) 방식으로 수행해 트래픽 증가 시 CPU가 패킷 하나하나의 경로를 계산하는 속도가 실제 데이터 속도를 따라가지 못해 병목 현상 발생
    - 라우팅 기능을 하드웨어 칩셋 ASIC에 추가해 하드웨어 속도로 라우팅 수행할 수 있도록 함
- VLAN 간 통신 증가
    - 보안과 관리 효율을 위해 네트워크를 여러 VLAN으로 쪼개는 경우가 많아짐
    - L2 Switch는 다른 VLAN으로 패킷을 보낼 능력이 없어 Router를 거치는 과정 필요 => 대역폭 낭비와 지연 시간 발생
    - Switch 내부에서 직접 VLAN 간 라우팅 처리해 효율성 극대화

하드웨어 기반으로 패킷 라우팅

내부 네트워크 LAN 중심

### Router

소프트웨어 기반으로 패킷 라우팅

대부분 LAN과 WAN 간 데이터 전달

정교한 필터링 및 보안 기능 존재

</details>

## [NET-085] 각 Layer는 패킷을 어떻게 명칭하나요? 예를 들어, Transport Layer의 경우 Segment라 부릅니다.

<details>
<summary>정답</summary>

L7에서 L5까지는 Data (또는 Message), L4에서는 Segment(TCP) 또는 Datagram(UDP), L3에서는 Packet, L2에서는 Frame, L1에서는 Bit

</details>

<details>
<summary>개념</summary>

### PDU (Protocol Data Unit)

하위 계층으로 내려갈수록 캡슐화가 진행되며 각 단계에서 packet을 칭하는 말이 다름

- Application, Presentation, Session (L7-L5): Data (Message)
- Transport (L4): Segment(TCP) 또는 Datagram(UDP)
- Network (L3): Packet
- Data Link (L2): Frame
- Physical (L1): Bit

</details>

## [NET-086] 각각의 Header의 Packing Order에 대해 설명해 주세요.

<details>
<summary>정답</summary>

L7 PDU인 Data에 L4 Header(TCP/UDP Header)가 붙어 Segment 생성

Segment에 L3 Header(IP Header)가 붙어 Packet 생성

Packet에 L2 Header(Ethernet Header와 L2 Trailer)가 붙어 Frame 생성

</details>

<details>
<summary>개념</summary>

### 캡슐화 (Encapsulation)

상위 계층의 데이터가 하위 계층으로 내려오면서 각 계층의 제어 정보 (header)가 덧붙여지는 과정

상위 계층의 PDU에 하위 계층의 Header가 붙어 하위 계층의 PDU가 만들어짐

### PDU

Segment = L4 Header + Data

Packet = L3 Header + Segment

Frame = L2 Header + Packet + L2 Trailer

### Header

L4 Header (TCP 헤더와 UDP 헤더)

- 핵심정보: Source/Destination Port (송수신 프로세스 구분)
- TCP의 경우, Sequence Number(순서 보장), Acknowledgment Number(수신 확인), Window Size(흐름 제어) 포함
- UDP의 경우, Port 번호, checksum

L3 Header (IPv4 헤더와 IPv6 헤더)

- 핵심정보: Source/Destination IP Address (최종 목적지 컴퓨터 주소)
- TTL(네트워크 상 무한 루프 방지), Protocol ID(TCP인지 UDP인지)

L2 Header (Ethernet 헤더)

- 핵심정보: Source/Destination MAC Address (물리적 주소)
- EtherType(IPv4 또는 ARP), Trailer(데이터 전송 중 오류 발생 여부)

</details>

## [NET-087] ARP에 대해 설명해 주세요.

<details>
<summary>정답</summary>

ARP란 논리적 주소인 IP 주소를 기반으로 물리적 주소인 MAC 주소를 찾아주는 프로토콜

로컬 네트워크에서 통신하기 위해 최종적으로 상대방의 MAC 주소를 알아야 하므로 IP를 브로드캐스트하여 응답 장비의 MAC 주소 탐색

</details>

<details>
<summary>개념</summary>

### ARP 

동작과정
1. ARP Request (Broadcast)
2. ARP Reply (Unicast)
3. ARP Table 저장

중요성
- L2 (Data Link) 계층과 L3 (Network) 계층 사이의 브릿지 역할

### RARP

ARP와 반대로 MAC 주소를 통해 IP 주소를 탐색하는 프로토콜

</details>


## Reference

https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-OSI-7%EA%B3%84%EC%B8%B5-%EC%A0%95%EB%A6%AC