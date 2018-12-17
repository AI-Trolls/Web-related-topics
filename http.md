# http basic
http관련 기본 지식 정리

## 목차
- [Web, Network](#1)
- [HTTP](#2)
- [HTTP Message](#3)
- [HTTP status code](#4)
- [Web Server](#5)
- [HTTP header](#6)
- [HTTPS](#7)
- [Web 공격](#8)

## <a name='1'>Web, Network</a>
- **HTTP(HyperText Transfer Protocol)**
  - protocol? 통신을 시작, 수행, 종료 하는 방법을 정리한 것
- 웹 문서 전송 프로토콜인 HTTP는 발전이 더딤(현재 1.x버전), 곧 차세대 프로토콜 HTTP/2.0이 등장할 것임
- **HTTP**는 **TCP/IP 프로토콜을 기반**으로 동작
- TCP/IP는 **계층으로 구성**됨
  - Application Layer
    - 애플리케이션에서 직접적으로 사용하는 통신의 흐름을 구현
    - HTTP, FTP, DNS 애플리케이션 등이 있음
  - Transport Layer 
    - 애플리케이션 레이어에 접속되어 있는 2대의 엔드포인트(컴퓨터) 사이의 데이터 흐름을 제공
    - TCP, UDP 프로토콜이 있음
  - Network Layer(Internet Layer)
    - 네트워크 상에서 패킷의 이동을 담당(어떻게 전달이 될지, 길은 어떻게 결정하고..)
  - Link Layer ; 하드웨어에 가까운 쪽의 흐름을 담당 (driver, Network Interface Card, ...)
- TCP/IP **통신의 흐름**은 대략적으로 다음과 같음
  - 송신 측은 애플리케이션 레이어에서 밑으로 내려가고
  - 수신 측은 반대로 메시지가 밑에서 올라옴
    1. HTTP의 경우 클라이언트 쪽에서 먼저 HTTP request를 내려보냄
    2. TCP 레이어(transport)에서는 HTTP msg를 조각 내어 조각에 대한 번호와 포트 번호를 붙여 내려보냄
    3. Network 레이어에서는 수신지의 MAC 주소를 추가해서 link 레이어에 내려보냄
    4. 수신측에선 반대의 순서로 메시지를 받음
- **Internet Protocol;IP**는 배송을 담당
  - 네트워크 레이어
  - 패킷 하나하나를 상대쪽으로 전달
  - 이 때, IP addr과 MAC addr이 필요
  - IP 주소는 각 노드에 (일시적or고정적)부여된 주소, MAC 주소는 각 NIC(network interface card)에 할당된 고유 주소
  - IP 주소는 MAC 주소와 맺어지며, 전자는 변경 가능하지만 후자는 그렇지 않다.
  - 실제 통신은 ARP(Address Resolution Protocol)에 의해서 이루어지는데,
    - 보통은 여러 노드를 거쳐서 패킷이 도착하게 되고,
    - 이 때, 어떤 노드를 기준으로 다음 노드를 찾아갈 때 MAC 주소를 사용해 당장의 다음 목적지를 정함
    - ARP는 수신지의 IP 주소를 바탕으로 바로 다음 노드의 MAC 주소를 알려줌
- TCP는 **신뢰성**을 보장해준다.
  - 신뢰성 있는 byte stream 서비스를 제공해줌
  - 데이터를 tcp segment로 나누어 보내고, 잘 도착했는지까지 확인해준다
  - 상대에게 확실하게 데이터를 보내기 위해 사용하는 방법들
    - three way handshaking
    - SYN, ACK
    - 클라이언트(통신을 시작하는놈, 송신측)에서는 맨 처음, 
      1. 'SYN' 플래그로 상대에게 접속을 하면서 동시에 패킷을 보냄
      2. 수신측에서는 'SYN/ACK' 플래그로 송신측에 접속하면서 패킷을 수신했다고 표시
      3. 마지막에 송신측이 'ACK' 플래그를 보내면서 패킷 교환 완료를 전함
- DNS는 application layer의 애플리케이션이고
  - 도메인명을 IP주소로 바꿔주는 서비스를 담당
## <a name='2'>HTTP</a>


## <a name='3'>HTTP Message</a>


## <a name='4'>HTTP status code</a>


## <a name='5'>Web Server</a>


## <a name='6'>HTTP header</a>


## <a name='7'>HTTPS</a>


## <a name='8'>Web 공격</a> 
