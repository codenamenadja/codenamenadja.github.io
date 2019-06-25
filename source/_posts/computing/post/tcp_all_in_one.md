---
title: TCP분석과 이해
p: computing/post/tcp_all_in_one
date: 2019-06-19 18:34:26
tags: ['computing', 'tcp', 'network']
---

TCP의 중요한 성질
1. Connected Oriented: 연결지향
2. Bidirectional Byte stream: 양방향성 바이트스트림(octet-stream)
3. In-order delivery: 순차적인 전송
4. Reliability through ACK: ACK를 통한 안정성
5. Flow Control: 수신자의 receive window에 맞는 바이트 수 안쪽으로 데이터를 전송
6. **Congestion control** : 일방적으로 일대 다 통신을 강요받는 서버가 네트워크 정체를 방지하기 위해 congest해on window를 사용. 데이터양을 제한. TCP vegas등 알고리즘이 있다. Flow Control과 달리 송신자가 단독으로 구현.

____

## 1. 데이터 전송

스텍에는 여러 레이어가 있다.
그렇지만 크게, user영역, device영역으로 나눌 수 있다. 유저 영역과 커널영역의 작업은 CPU가 수행한다.

- 유저영역+커널영역 = HOST
- Device영역 = NIC

> NIC로 인해 IO바운드한 소켓버퍼를 설계할 수 있고 싱글쓰레드의 비동기 처리는 이 장치에 기인한다.

| Boundary | layer        | description                                                         |                                                packet |
| :------- | :----------- | :------------------------------------------------------------------ | ----------------------------------------------------: |
| User     | App(process) | write(fd,buf,len)                                                   |                                                  data |
| Kernel   | File         | 파일descritor을 갱신                                                     |
| Kernel   | Sockets      | 버퍼객체를 소켓버퍼로 copy                                                    | sendsocketbuffer:[BYTE_DATA] (송수신용 버퍼에 맞게 저장된 바이트데이터) |
| Kernel   | TCP          | TCP상태와 checksum에 따라 TCP 계층을 구성                                      |                                   [TCP,B_data]:buffer |
| Kernel   | IP           | IP헤더추가, IP라우팅추가, 체크섬진행                                              |                                       [IP,TCP,B_data] |
| Kernel   | Ethernet     | ethernet 헤더를 붙이고, ARP실행(주소변환)                                       |                                   [Eth,IP,TCP,B_data] |
| Kernel   | Driver       | NIC에게 소켓을 보내라고 전달하는OS의 마지막 레벨                                       |
| Device   | NIC          | 호스트메모리에서 패킷을 받아 fetch한다. **_Interrupt the host when send is done_** |                          [IFG,Pre,Eth,IP,TCP,payload] |

> CheckSum: 중복검사의 한 형태로, 오류정정을 통해, 전자통신이나 기억장치 속에서  
>  송신된 자료의 무결성을 보호하는 단순한 방법이다.

> Linux나 UNIX를 비롯한 POSIX계열 운영체제는 소켓을 FD로 애플리케이션에 그 고유성을 노출한다.

> 파일레이어가 단순한 검사후에, 커널소켓에 연결된 소켓구조체(프로그램레벨 소켓객체)를 사용하여, 소켓함수를 제공한다.

> 커널 소켓은 두개의 버퍼를 가지고 있다. (send Socket Buffer, receice socket buffer) write시스템 콜을 호출하면 유저영역의 데이터가 커널 메모리로 복사된다. 이 다음으로 TCP를 호출한다.

> 소켓과 연결된 TCP Control Block(TCB)가 있다. TCB를 다룬다.

***

TCB에 있는 데이터,
```javascript
TCB: [
connection state(LISTEN, ESTABLISHED, TIME_WAIT등),  
receive window,  
congestion window,  
sequence넘버,  
재전송 타이머 등  
]
```
***

1. TCP상태가, 전송을 허용하면, TCPsegment. 즉, 패킷을 생성한다.

1. 이후 IP레이어를 거쳐 다음 HOP등에 대한 라우팅이나 헤더추가가 끝나면, NIC로 결국 내려가게 되는데,

1. NIC는 패킷전송 요청을 OS레벨 드라이버에게서 받고, 메인 메모리에 있는 패킷을 **자신의 메모리로 복사하고** 네트워크 선으로 전송한다.

1. 놀라운 일이다. NIC는 독립적인 메모리를 가진 디바이스이다.

____

## 2. NIC

> 매우 중요하다, 비동기 싱글쓰레드가 가능한건 순전히 이 부분의 힘이다.
> 
- NIC가 패킷을 전송할 때 NIC는 호스트CPU에 interrupt를 발생시킨다. 모든 인터럽트에는 인터럽트 번호가 있으며,
- 운영체제는 이 번호를 이용하여 이 인터럽트를 처리할 수 있는 적합한 드라이버를 찾는다.
- 드라이버는 인터럽트 핸들러를 드라이버가 가동되었을 때 **운영체제에** 등록해둔다.
- 운영체제가 핸들러를 호출하고, 핸들러는 전송된 패킷을 운영체제에 반환한다.

____

## 3. 패킷 수신

* NIC
  1. 드라이버가 미리 할당해 놓은 메모리버퍼가 없으면 NIC가 패킷을 버릴 수 있다!  
  2. 패킷을 호스트 메모리로 전송한 후, NIC가 호스트 운영체제에 인터럽트를 보낸다.

* 드라이버
  1. 드라이버가 상위 레이어로 패킷을 전달하려면 운영체제가 이해할 수 있도록 운영체제 구조체를 포장해야한다.
  - LINUX:sk_buff
  - BSC:mbuf
  - MS:NET_BUFFER_LIST

* Ethernet
  1. 상위프로토콜인 네트워크 프로토콜을 찾는다. 이때 Ethernet헤더의 ethertype을 검사한다.  
  2. IPv4의 ethertype은 0x0800이다. 헤더를 제거하고, IP레이어로 패킷을 전달한다.

* IP레이어
  1. IP헤더 체크섬을 확인한다. : 논리적으로 여기서 IP 라우팅을 해서 패킷을 로컬장비가 처리해야하는지 아니면 다른장비로 전달해야 하는지 판단한다. 
  2. IP헤더의 proto값을 보고 찾는다. TCP proto값은 6이다. 헤더를 제거하고, TCP레이어로 패킷을 전달한다.

* TCP
  1. TCP체크섬을 확인하는데, 요즘 네트워크 스텍에는 checksum offload기술이 적용되어 있기 떄문에, 커널이 아니라 드라이버 NIC가 이 과정을 처리해준다.
  2. TCPcontrol block을 찾는다. 이때 패킷의 <소스 IP, 소스 port, 타킷 IP, 타깃 port>를 식별자로 사용한다.
  3. 연결을 찾으면, 페이로드를 receive socket buffer에 추가한다.
  4. TCP상태에 따라 새로운 TCP패킷(ACK등)을 전송할 수 있다.

>TCP가 Receivesocketbuffer로 잔달할때, 이 크기가 결국 receive window이다. 최신 네트워크 스텍은 윈도우 크기를 자동 조절하는 기능을 가지고 있다.

1. 마지막으로 애플리케이션이 read시스템 콜을 사용하면 버퍼에 있는 메모리를 유저공간의 메모리로 복사하고, 복사한 메모리는 제거한다.
2. 제거된 공간덕에 버퍼에 공간이 늘었기 때문에 윈도우 크기를 증가시킨다. 

소켓레이어에 sendbuffer[snd.UNA:아직 ACK를 받지 못한 데이터시퀸스 시작 넘버, snd.NXT:다음에 보낼 시퀸스 시작 넘버]
- 상대방의 ACK와 합의된 window크기에 따라 맞춰서 보내준다.

소켓레이어에 receivebuffer:[nxt:다음번에 받을 상대방의 시퀸스 넘버]