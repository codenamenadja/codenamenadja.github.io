---
title: TCP의 3단계 절차
p: computing/network/3_step_in_tcp
date: 2019-06-19 18:32:33
tags: ['network']
---


# TCP의 3단계

## 1. 연결설정
> 최초 요청을 주는 쪽에서, connect함수 호출과 동시에, three-way handshake가 실행된다!

| requester                    |                  responser |
| :--------------------------- | -------------------------: |
| SYN(SEQ:1000,ACK:-)          |               ...listening |
| waiting...                   | SYN+ACK(SEQ:2000,ACK:1001) |
| ACK(SEQ:1001,ACK:2001)       |                 ...waiting |
| ACK only:lasthandshake       | receive ACK, ready to send |
| SYN:synchronization          |           ACK: acknowledge |
| now we are connected in TCP. | _**three way hand Shake**_ |

> 본격적으로 원하는 페이로드를 요청하고 받기 이전에 이 과정이 완료되야 connected했다고 한다.

> 두번째 단계에서는 데이터 송수신이 시작된다.
***
## 2. 데이터 송수신

- 데이터 전송 단계에서 **flow control**은 첫 번째 단계보다 훨씬 복잡하다.  
예시는 responser가 requester에게 많은 양의 데이터를 전송하는 상황이다.

| RES                        |                             REQ |
| :------------------------- | ------------------------------: |
| packet(SEQ:1301, 100bytes) | listening(registered requester) |
| waitForACK                 |                        ACK:1401 |
| SEQ:1401, 100bytes         |                      ...waiting |
| waitForACK...              |     (didnt receive!) ...waiting |
| timeout(wait enough!)      |                      ...waiting |
| SEQ:1401, 100bytes         |                      ...waiting |
| waitForACK...              |                        ACK:1501 |

> 1. 100바이트를 전송하면서 _SEQ:1301_ 을 전송한다. 1301으로부터 offset 100.  
> 2. ACK패킷이 잘 도착했다면, RES는 1301부터 100떨어진 *SEQ1401*에서 100byte를 전송한다.
> 3. timeout이 발생, 이전 패킷을 다시 보낸다.
> 4. 마지막까지 잘 도착했다면, ACK:1501로 마지막 바이트가 끝난 지점을 보낸다.
> - ACK를 수신한 바이트 수만큼 증가시켜서 돌려보내줘야, 유실없이 잘 받았다는 싸인이다.  
IP(internet Protocol)은 기본적으로 패킷의 유실이 발생할 수 있는 불안정적인 연결을 깔고 가고,  
TCP는 그만큼 정확한 체크를 통해 서로 안전한 연결성을 보장한다. 
***
## 3. 연결 종료
### four-way handshaking
- 연결을 종료하는 과정은 소켓프로그래머에게 가장 중요한 단계에 해당된다.
- 이번에는 연결을 계속 유지되는 것이 기본 TCP소켓 스트림이기 때문에,  
Client가 server에게 연결종료 하겠다고 알리는 과정이라고 가정한다.
***

| Client                 |                 Server |
| :--------------------- | ---------------------: |
| FIN(SEQ:5000,ACK:-)    |  ...waitForClientEvent |
| waitForACK...          | ACK(SEQ:7500,ACK:5001) |
| waitForFIN...          | FIN(SEQ:7501,ACK:5001) |
| ACK(SEQ:5001,ACK:7502) |          ...waitForACK |

***
> 이것은 상호간에 종료 통보와 확인을 교환하는 것이다. 서로 메모리를 아끼는 수단.   
일반적으로 종료 사인을 보내는 사람이 상대방을 향한 소켓을 종료하겠다는 FIN으로 시작하면,  
상대방이 그 FIN에 대해 ACK를 먼저 보내고,  
이후 상대방도 자신도 상대방에게 FIN패킷을 보낸다.  
마지막으로 연결종료를 시도한 사람이 상대방의 FIN에 대한 ACK패킷을 보내면  
통신은 종료되고 서로의 고유한 디스크립터가 붙은 I/O는 캐쉬에서 사라진다.
***

