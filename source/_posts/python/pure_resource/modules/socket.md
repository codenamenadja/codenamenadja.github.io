---
title: 파이썬 Socket 모듈 정리
p: python/pure_resource/module/socket
date: 2019-06-19 18:47:15
tags: ['python', 'python module', 'socket']
---

1. socket.socket(family,type,proto,fileno)
    - 주어진 주소를 사용하는 소켓을 생성한다.
    - family는 AF_INET, AF_NET6, AF_UNIX, AF_PACKET, AF_RDS(socket.AF_NET)
    - type은 SOCK_STREAM, SOCK_DGRAM, SOCK_RAW
    - protocol number는 보통 0이다, addrFamily가 AF_CAN이거나 하면 다른 값을 가져야 한다.
    - fileno descriptor가 특정되었다면, 파일 디스크립터에 의해서 , 패밀리 타입, 프로토콜번호가 자동으로 잡힌다.

2. serverSocket.listen([backlog]:Optional int)
    -  서버가 연결을 수락하도록 허용한다.
    -  백로그가 특정되어있다면, 새로운 연결을 거절하기전에 시스템이 허락한 받아들여지지 않은 연결 수를 특정하는 것이다.

3. serverSocket.accept()
    - 연결을 받아들인다. 소켓은 주소에 달라붙어있어야하고, 포트 연결에 대해서 수용하고(listen) 있어야 한다.
    - return value는 pair로 (conn, address)이다.
    - conn은 새로운 소켓객체다, 데이터를 보내고 받는데 사용될 수 있다.
    - address는 해당 소켓이 반대쪽 연결의 끝에 어디에 달라붙어 있는지를 얘기한다.
    - 3.5버전 이후 변경점: 시스템콜이 방해를 받아 signal handler이, exception을 raise하지 않는다면, 이 메서드는 시스템 콜에게 InterruptedError를 raise하지 않고, accept를 재시도 한다.
4. socket.close()
    - 소켓이 닫혔다는 것을 표시한다. FD같은 잠재된 시스템 자원 또한 모든 IO 버퍼에 대한 FD를 markfile()로 확인해서 모든 파일 객체가 닫혔다면, 닫혀버린다.
5. socket.markfile(mode='r', buffering=None, *,encoding=None, errors=None, newline=None)
    - 소켓과 연결지어진 파일 객체를 반환한다.
6. file object : 공식문서 용어해설
   - 파일에 기원을 두는 API를 내부 실제 리소스에게 드러내는 객체.
   - read(), write() 등이 그러한 API이다.
   - 이것이 어떻게 생성됐는지에 따라서, file object는 실제 디스크의 파일이나, 다른 타입의 저장소 혹은 communication device에 접근 할 것인지를 고려할 수 있다.
    - 해당 underlying resourced의 목록
        1. statdard input/ouput
        2. in-memory buffers
        3. sockets
        4. pipes, etc..
    - File Objects는 file-like object 혹은 streams라고 불리기도 한다.
    - 그들은 주로 세가지 카테고리의 파일 객체(정석의 생성법은 open메서드를 통해)
      - binary files, buffered binary files, textfiles
7. socket.recv(bufsize[, flags])
    - 소켓에서 데이터를 받아온다. 리턴값은 바이트객체.
    - 한번에 받을 수 있는 데이터 양의 총량은 bufsize에 의해 특정된다.
    > 네트워크와 하드웨어를 생각했을때. 최적의 버프사이즈는 2의 제곱수로 잡는 것이 좋다.
8. socket.connect(address)
    - 해당 주소의 remote socket에 접속한다.
    - 주소의 포멧은 address family에 의해 결정된다.
    - 만약 signal에 의해 connection이 interrupted되면, 메서드는 연결이 끝날때 까지 기다리거나, 
    - raise socket.timeout on timeout 한다. 만약에 signal handler가 exception을 raise하지 않고, 소켓이 블로킹 소켓이거나 timeout을 가졌다면 
    - 논블로킹 소켓에 대해서, 메서드는 InterruptedError exception을 raise한다. 만약 connection 이 signal에 의해 방해 받았을 때.
9. socket.fileno()
    - 소켓의 파일 descriptor을 반환한다. -1을 반환한 다면 실패. select.select()와 유용하게 쓰인다.
10. select.select(rlist,wlist,xlist[,timeout])
    - 이것은 UNIX select()시스템 콜을 향한 딱 맞는 인터페이스이다.
    - 3가지 매개변수는 waitable object의 연속을 전달한다.:
      - 모두 파일 디스크립터를 통해 fileno() 등을 통해 전달받은 숫자가 된다.
      -  1. rlist: wait until ready for reading 
      -  2. wlist: wait ready for writting
      -  3. xlist: wait for an "exceptional condition(by system)"
    -   빈 시퀸스는 허락된다.
    -   timeout 매개변수가 생략되었을때, 함수는 하나라도 FD가 준비되었을 때까지 block된다.
    -   리턴하는 것은 준비된 3가지 리스트 객체를 리턴한다.
    -   시퀸스에 받아들일 수 있는 타입들은,파이썬 fileobjects(sys.stdin, open(), os.popen()), 혹은 소켓 객체가 있다.
    -    3.5버전의 변화: 함수는 이제 신호에 의해서 interrupted될 때, timeout과 함께 재시도 된다. InterruptedError일 때만 그렇고 다른 에러를 Interrupted 핸들러가 발생시키면 재시도 되지 않는다.