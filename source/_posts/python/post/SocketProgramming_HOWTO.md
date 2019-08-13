---
title: SocketProgramming_HOW 번역
p: python/post/SocketProgramming_HOWTO
date: 2019-06-19 18:48:54
tags: ['python', 'python post']
---

## Author: Gordon McMillan

출처 : [https://docs.python.org/3.7/howto/sockets.html#socket-howto](https://docs.python.org/3.7/howto/sockets.html#socket-howto)

***

## 1. Sockets

INET(i.e IPv4)소켓에 대해서만 이야기하겠다. 하지만 그들이 99%의 실사용 소켓이다. 그리고 STREAM TCP소켓에 대해서만 이야기 할 것이다.

니가 만약 무얼하고 있는지 알고 있다면, 그 무엇보다 스트림 소켓에서 더욱 좋은 퍼포먼스를 가지게 될 것이다. 

소켓이 무엇인지에 대한 미스터리를 정리하려 할 것이다. 블로킹 소켓과 NON-blocking소켓에 대해서도 조금 힌트를 준다. 그렇지만, 일단 blocking소켓에 대해서 이야기하는 것으로 시작하겠다. 너는 그들이 어떻게 넌블러킹 이전에 해왔는지 알 필요가 있다.

이들을 이해하는데 생기는 문제는, socket이라는게 미묘하게 다른 다수의 것을 뜻할 수 있다는 것이다.
그러니 첫째로, client socket, server socket사이에 구분을 해보자.(서버 소켓이 더 switch board operator같다.)
클라이언트 어플리케이션 예를 들어 너의 브라우저는, 클라이언트 소켓만을 사용한다. 웹서버는 서버소켓과 클아이언트 소켓을 동시에 사용한다.

***

## 2. History

IPC(Inter Process Communication)의 다양한 form에서, 소켓은 가장 유명하다. 어떤 플랫폼에서도, 그들은 더욱 빠르고, cross-platform커뮤니케이션이다. 소켓은 유일하게 지금 게임이 되는 존재다.

***

## 3. Creating a Socket

네가 링크를 클릭했을떄, 너의 브라우저는 이러한 것들을 한다.


```python
# create an INet, STREAMing socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# now connect to web server on port 80
s.connect(("www.python.org",80))
```

connect가 끝났을 때, 소켓 s는 페이지에 대한 리퀘스트를 보내기 위해서 사용될 수 있다.
동일한 소켓이 읽고 응답할 것이다. 그리고 소멸된다. 클라이언트소켓은 기본적으로 한번의 교환을 위해서 사용된다.

웹서버에서는 조금더 복잡하다. 첫쨰로 웹서버는 'server socket'을 생성한다.

```python
# create INET, STREAMing socket
serversocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
# bind the socket to public host, and a well-known port
serversocet.bind((socekt.gethostname(),80))
serversocket.listen(5)
```

몇 가지 알아야 할 사실: 우리는 socket.gethostname()을 사용하여 소켓이 외부 publicIP에 보일 수 있도록 했다.
local안에서 설정했다면, 동일 기계안에서만 보이게 될것이다.

두번째 알아야할 사실: well-known port는 public상의 효율성을 위한 약속이다. 네가 만약, 놀고싶으면 4자리 이상의 포트를 사용해라.

마지막: listen의 argument는 소켓라이브러리로 하여금, 우리가 queue를 5개 connect리퀘스트를 올리길 원한다는 것이다.(5 is normal max). 이후 외부 connect는 거절된다. 다른 코드가 적절히 구성되었다면, 그것은 충분할 것이다.

이제 우리가 서버소켓이 80번 포트에 있으니, 우리는 웹서버의 메인루프에 접근할 수 있다.

```python
while True:
    # accept connection from outside
    (clientsocket, address) = server.socket.accept()
    # noe do something with clientsocket
    # in this case, we'll pretent this is threded server
    ct = client_thread(clientsocket)
    ct.run()
```

사실 3가지 일반적인 방법이, 이것의 루프를 가능하게 하는 방법으로 있다.
1. 쓰레드를 붙여서, 클라이언트 소켓을 제어하는 것.
2. 앱을 재구성해서, nonblocking socket을 사용하게 하는것.
3. select를 활용하여 서버 소켓과 어떤 활성화된 클라이언트 소켓 사이를 다양화 시킨다.
   
알아야 하는 중요한 것은 이것이다. 이것에 서버소켓이 하는 모든 것이다.
이것은 어떤 데이터를 받지도 않고 데이터를 보내지도 않는다. 이것은 그저 클라이언트 소켓을 생성한다.
각 클라이언트 소켓은 어떤 클라이언트의 소켓이 host와 그의 bound된 port에 connect()함으로써 생성된다. 해당 클라이언트 소켓을 빠르게 우리가 생성한다면, 우리는 더 많은 연결을 기다리도록 돌아갈 수 있다.(listen(5))
2명의 클라이언트는 자유롭게 소통할 수 있다.
> 그들은 dynamically allocated port를 사용한다. 어떤 대화가 끝났을때 재사용 되는 포트.

***

## 4. IPC(interProcessCommunication)

네가 만약 2개의 프로세스 사이에 빠른 IPC를 원한다면, pipes나 shared memory를 생각해 봐야한다.
네가 만약 AF_INET소켓을 사용하기로 결정했다면, 서버소켓을 localhost에 bind해라.
대부분의 플랫폼에서, 이것은 네트워크의 몇개의 레이어로 하여금 shortcut을 사용하도록 해서, 더 빠르고 덜 요란하게 성립하게 해준다.

> See also: 멀티프로세싱은 crosee-platform IPC를 higher-level API로 통합시킨다.

***

## 5. Using Socket

첫번째로 기억해야 할 것은 웹 브라우저의 클라이언트소켓과 웹 서버의 클라이언트소켓은 개별적인 괴물들이다.
그 말은, 이것은 peer to peer 대화 라는 것이다. 다른식으로 얘기하자면, 데자이너로서 너는 대화를 위한 에티켓으로 무엇을 할지를 정해야 할 것이다.
보통 소켓을 연결시키는 것은, 대화를 시작하는 것이다. 리퀘스트를 보냄으로써, 혹은 조직된 행동에 동의하기로 서명함으로써.
> 조직된 행동이란 소켓의 룰이 아니다.

이제 소통을 위한 2개의 동사 세트가 있다. 너는 send, recv를 쓸 수 있다. 아니면 너는 , 너의 클라이언트 소켓을 파일같은 괴수로 만들어서 읽고 쓸 수 있다. 두번째 방법은, 자바가 그의 소켓을 대표하는 방법이다. 넌 너에게 소켓에 대해서 flush를 사용할 필요가 있다는 것을 경고하기만 하겠다. 그것은 bufferd 파일이다. 그리고 쉬운 착각이 뭔가 **write**하기 위해서, 그리고 나서 응답에 대해서 **reply**한다는 것이다. flush가 없다면, 계속 응답에 대해서 기다려야 한다는 것이다. 왜냐하면 리퀘스트는 아직 너의 출력 버퍼에 적혀진 채로 있을 수 있다.

이제 우리는 가장 주요한 소켓의 가장 큰 걸림돌에 도착했다.
- 네트워크 버퍼에 대한 ```send``` 와 ```recv``` 명령이다.

그들은 네가 건네준 모든 바이트를 다루지 않는다. 왜냐하면, 그들의 주된 목적은 네트워크 버퍼를 다루는 것이다. 일반적으로, 그들은 관련된 네티워크 버퍼가 차거나(send), 비었을 때(recv), return한다.
그리고 나서, 그들은 너에게 얼마나 많은 버퍼를 다뤘는지 알려준다.
너의 메세지가 완전히 다뤄지기전까지 그들을 다시 부르는 것은 너의 소관이다.

recv가 0bytes를 반환한다면, 그것은 다른 사이드의 연결이 닫혔거나, 반만 닫았다는 얘기이다. 그러면 너는 해당 연결에 대해서 더 이상의 데이터를 받을 수 없다.
너는 상대방이 반만 닫았기 때문에, 데이터를 잘 보낼 수 있을 수도 있다.

```HTTP``` 같은 프로토콜은 소켓을 하나의 전송에 대해서만 사용한다. 클라이언트가 요철을 보내면, 응답을 읽는다. 그리고 나면 소켓은 정지된다. 이것은 클라이언트가 응답의 끝을 0바이트를 받는 것을 통해서 알 수 있다는 것이다.

그러나 소켓을 추가적인 전송을 위해서 너의 소켓을 재사용 하기로 계획하고 있다면, 너는 소켓에는 EOT(END OF Transfer)가 없다는 것을 알아야 한다.

다시 강조한다. 소켓의 send, recv가 0 바이트를 다룬 루에 반환한다면, 연결은 이미 끊어졌다. 만약 연결이 끊어지지 않았다면? 너는 평생 recv 기다려야 할 수도 있다.
왜냐하면 소켓이 더 이상 읽어야 할게 없다고 알려주지 않을 것이기 때문이다.
네가 그것에 대해서 조금 더 생각한다면, 소켓에 대한 기초구현적인 진실을 깨닫게 될 것이다.
- 메세지는 고정된 길이(yuck)거나, 범위가 제한(shrug) 되어야 한다. 또는, 그들이 얼마나 긴지(훨씬 낫다), 혹은 연결 종료를 소통하면서 말이다. 선택은 너의 몫이다. 하지만 어떤 것은 다른 것보다 낫다.

네가 연결을 종료하고 싶지 않다고 생각했을때. 가장 쉬운 방법은 고정된 길이의 메세지이다.

```python
class MySocket:
"""
클래스만을 설명한다 - 깔끔함을 위한 코드이지 효율성이 아니다.
"""

    def __init__(self, sock=None):
        if sock is None:
            self.sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        else:
            self.sock = sock

    def connect(self, host, port):
        self.sock.connect((host, port))

    def mysend(self, msg):
        totalsent = 0
        while totalsent < MSGLEN:
            sent = self.sock.send(msg[totalsent:])
            if sent == 0:
                raise RuntimeError("socket connection broken")
            totalsent = totalsent +sent

    def myreceive(self):
    chunks = []
    bytes_recd = 0
    while bytes_recd < MSGLEN:
        chunk = self.sock.recv(min(MSGLEN - bytes_recd, 2048))
        if chunk == b'':
            raise RuntimeError("socket connection broken")
        chunks.append(chunk)
        bytes_recd = bytes_recd + len(chunk)
    retutn b''.join(chunks)
```

여기서 코드를 전달하는 것은 어떤 메세지 포멧에서도 거의 다 유용하다. 파이썬에서, 네가 문자열을 보내면, 너는 len()을 통해서, 그것의 길이를 정할 수 있다. 첫번째 단어를 가져옴 으로써, 너는 길이를 알아볼 수 있다. 너는 연속된 청크사이즈를 얻게 될 것이다.(4096 ~ 8192가 네트워크 버퍼사이즈로 적당하다.)그리고 네가 구획 문자로 무엇을 받았는지, 확인하면서 말이다.

한가지 네가 주의할 점은: 너의 일반적인 프로토콜이 다수의 메세지가 메세지를 뒤로 위치하도록 보내진다면(어떤 응답 없이),
그리고 네가 recv를 전달해서 청크사이즈의 꾸러미를 수용한다면,
너는 추가로 오는 메세지의 읽는 것을 시작하는 것을 종료해버릴지 모른다.
너는 그것을 다른 곳에 놓고, 그게 필요해질 때까지 저장할 수 있어야 한다.

메세지를 그것의 길이를 앞에 두도록 하는 것은 더욱 복잡해진다. 너느 모든 5개의 문자를 한번의 recv로 받지 못할 수도 있다.
제대로 활용하기 위해서, 너는 그것에서 멀어질 것이다.

하지만, 커다란 네티워크 전송해서, 너의 코드는 매우 빠르게 부서질 것이다.
네가 두 개의 recv loop를 사용하지 않는 다면, 
1. 첫번째는 길이를 정하고.
2. 두번째는 데이터를 받아오기 위해서.

불행하게도, 이것은 또한 네가 send할때, 항상 메세지를 한번에 보내서 너의 버퍼에서 삭제하는 것은 아니라는 것에서 밝혀진다. 이걸 읽고나서도, 너는 결국 이것에 물릴 것이다!

공간에서 있어서, 문자열을 만드는 것은, 이러한 증진은 읽는 이들을 위한 훈련으로 남아 있다.

## 6. Binary Data

소켓을 통해서 바이너리 데이터를 보내는 것은, 딱 맞는 일이다. 문제는, 모든 기계가 같은 포멧의 바이너리 데이터를 받는건 아니라는 것이다. 에를 들어 Motorola 칩은, 16비트를 사용해 1을 표현하고, 두개의 16진수 바이트로 00 01으로 한다. 인텔에서는, 바이트가 거꾸로 된다.1은 01 00과 같다. 소켓 라이브러리는, 16비트 32비트 정수를 전환하는 것으로 ntohl, htonl, ntohs, htoms를 호출한다. n은 네트워크를 뜻하고 h는 호스트를 뜻한다.s는 short, l은 long을 뜻한다. 네트워크의 주문자가 호스트인 경우, 그것은 아무것도 안한다. 하지만 기계가 바이트가 거꾸로 됐다면, 이것은 바이트를 적절하게 바꾼다.

32비트 체제인 요즘엔, 아스키가 바이너리 데이터를 표현하는게, 그냥 바이너리를 표현하는 것보다 통 작다. 왜냐하면, 놀랍게도, 모든 길이가 0이나 1을 가지고 있기 때문이다. "0"이라는 문자는 2바이트를 잡고, 바이너리는 4바이트를 잡는다. 하지만 그것은 고정된 길이의 메세지와는 맞지 않는다.

## 7. Disconnecting

딱 잘라 말해서, 너는 소켓을 close하기 전에 shutdown해버릴 것이다.
shutdown은 소켓에서 경고다.
네가 어떤 argument를 전달했으냐에 따라서, 더 이상 보내진 않겠지만, 듣기는 하겠습니다. 또는 난 듣지 않고 있어요. 라고 말할 수 도 있다.
대부분 소켓 라이브러리는, 이러한 shutdown()을 close와 같이 취급하여 에티켓을 무시해버린다.
그래서 대부분의 경우에, 별도의 shutdown은 필요하지 않다.

shutdown을 효과적으로 사용하는 방법은 HTTP같은 교환법이다.
클라이언트가 리퀘스트를 보내면 이후 shutdown(1)을 수행한다. 이것은 서버에게 클라이언트가 전송을 중지한다. 그러나 아직 받을 수 있다.
라고 말해준다. 서버는 EOF를 0바이트를 받아들임으로써 감지할 수 있다.
이것은 리퀘스트가 끝났다고 감지하는 것이다.
서버가 응답을 보낸다. 만약 send가 성공적으로 완수되면, 클아이언트는 아직 듣고 있었던 것이다.

파이썬은 멀치감치 자동 shutdown을 수행한다.
그리고 소켓이 garbage collected됐을때 그것을 자신에게 공식화한다.
이것은 필요할 경우에 자동으로 close처리 할 것이다.
하지만, 이것에 의존하는 것은 매우 나쁜 습관이다.
만약 너의 소켓이close하지 않고 사라졌다면, 소켓은 다른 끝에서 계속 살아 있을 수 있다.
네가 그저 느린 것이라고 판단하고 말이다.
그러니 제발 일이 끝나면 close를 명시해라. 내부적인 장치가 항상 중요하다.

## 8. When Sockets Die

아마도 blocking소켓을 사용하는 것에서 가장 나쁜것은, 다른 쪽이 close없이 조용해져 버렸을때 일어나는 일이다.
너의 소켓은 아마 걸려 있을것이다. TCP는 연결지향적인 프로토콜이라, 그래서 오래 기다린다. 연결을 소멸시키기 전까지 오래 기다린다.
만약 네가 쓰레드를 사용한다면, 모든 쓰레드는 필수적으로 죽어버릴 것이다.
네가 거기에 대해서 할 수 있는 것이 거의 없다.

쓰레드를 죽이려고 하지마라:

- 이유 중 하나는, 쓰레드는 프로세스보다 훨씬 효율적인 이유는, 그들이 자동으로 자원을 재활용 함으로써, 오버헤드를 피하고 있기 때문이다.
- 다르게 말해서, 네가 쓰레드를 죽이려고 하면, 너의 모든 프로세스는 망치게 될 가능성이 높다.

## 9. Non-blocking Sockets

네가 진행을 이해하고 있다면, 소켓을 사용하는 데 필요한 기계적인 것들을 거의 다 알고 있는 것이다. 너는 아직 동일한 call을 사용할 것이다, 전과 다를바 없이.
이것은 그런 것이다. 네가 제대로 하면, 너의 앱은 거의 할 것을 다 한것이다.

파이썬에서는 ```socket.setblocking(0)```을 사용하여 non-blocking하게 한다. C에서 이것은 더욱 복잡하다.

> 한가지 말하자면, 너는 BSD의 0_NONBLOCK과 거의 구별할 수 없는 POSIX스타일의 0_NDELAY중에 고르게 될것이다.(TCP_NODELAY)와는 완전히 다르다. 그러나 동일한 개념이다.

> 소켓을 만든 후, 그리고 사용하기 전에 너는 그것을 한다.

주요 기계적인 차이는 send, recv, connect 그리고 accept는 아무것도 하지않고 return할 수 있다는 것이다. 너는 물론 몇가지 선택권이 있다.
너는 return 코드와 error코드를 체크해서, 일반적으로 너를 미치게 할 수 있다.
믿지 못해겠다면, 한번 쯤 해보는 것도 좋은 생각이다.
너의 앱이 커지면, 버그가 넘치고, CPU를 빨아먹는다.
그러니 그냥 생각 없이 이것을 따라라.

- USE ```select```

C에서는, select는 매우 복잡하다. Python에서는 그냥 케익 한조각일 뿐이다. 그러나 C에 매우 가깝기 때문에, python에서 Select를 이해하면, C에서 조금 순탄해 질 수 있다:

```c
read_to_read, ready_to_write, in_error = \
    select.select(
        potential_readers
        ,potential_writers
        ,potential_errs
        timeout      
    )
```

너는 select에 3가지 리스트를 전달한다.
1. 니가 읽으려고 시도하고 싶은 모든 소켓
2. 니가 쓰려고 시도하고 싶은 모든 소켓
3. 마지막(보통 비어있다) 네가 원하는 것에 에러를 체크해줄 것들.

너는 소켓이 하나의 리스트 이상에 전달 될 수 있다는 것을 알아야 한다.
select에 대한 호출은 blocking이다.
그러나 timeout을 줄 수 있다.
이것은 일반적으로  민감한 사안이다.

> 적절히 긴 timeout을 주어라, (예를 들면 1분), 네가 만약 다른 선택을 할 마땅한 이유가 없다면.

