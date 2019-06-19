---
title: 소켓프로그래밍 python C nodejs
p: python/pure_resource/socket_programing
date: 2019-06-19 18:27:36
tags: ['socket', 'tcp', 'python', 'c', 'nodejs']
---

> 소켓프로그래밍을 이해할때, 객체 지향적인 언어의 API를 이용해 왔다. nodeJS의 경우와 파이썬의 경우를 먼저 짧게 보여주기로 한다.

____
## 1. nodeJS socket Programing
```javascript
# typescript로 작성했던 서버사이드 TCP소켓 어플리케이션

import * as net from "net"

class TCPServer {
    public app: net.Server;
    public clientSockets: net.Socket[] = [];
    constructor() {
        this.app = net.createServer();
        this.serverConfigs();
    }
    public static bootstrap() {
        return new TCPServer();
    }
    private socketConfigs(clientSocket: net.Socket): void {
        this.clientSockets.push(clientSocket);
        console.log(`server :client connected to server, all  
         users:${this.clientSockets.length}`);
        clientSocket.on("data",(data)=>{
            console.log(`recevied msg:${data.toString()}`);
            if(data.toString().trim().toLowerCase() == "quit"){
                clientSocket.write(">>> disconnect order requested!");
                return clientSocket.end();
            }
            this.clientSockets.forEach((otherSocket)=>{
                if(otherSocket !== clientSocket){
                    otherSocket.write(data);
                }
            });
        });
        clientSocket.on("close",(isError)=>{
            const index =this.clientSockets.indexOf(clientSocket);
            if(index !== -1){
                this.clientSockets.splice(index, 1);
            }
        });
    }
    private serverConfigs() {
        this.app.on("connection", (socket) => {
            this.socketConfigs(socket);
        });
        this.app.on("error", (err) => {
            console.log(`error Message from server : ${err.message}`);
        });
        this.app.on("close", () => {
            console.log("close server");
        });
    }
}

const singleServer = TCPServer.bootstrap();
singleServer.app.listen(8000,()=>{
    console.log("server opende at port 8000");
})
```
____
## 2. python으로 작성된 크롤링 Tcp소켓 어플리케이션
```python


def fetcher(self):
    self.sock = socket.socket()
    self.sock.setblocking(False)
    try:
        self.sock.connect(('xkcd.com',80))
    except BlockingIOError:
        pass
    selector.register(
        self.sock.fileno(),
        EVENT_WRITE,
        self.connected,
    )

def connected(self, key, mask):
    print('connected')
    selector.unregister(key.fd)
    req = 'GET {} HTTP/1.0\r\nHOST: xkcd.com\r\n\r\n'.format(self.url)
    self.sock.send(req.encode('ascii'))

    #register next callback
    selector.register(
        key.fd,
        EVENT_READ,
        self.read_response
    )

```

- 공통점은 추상화가 매우 잘되어 있다는 점이다.  
 그 중에서도 노드JS의 추상화는 정말 말도 안되는 수준이라,  
 우리는 그저 소켓모듈에서 소켓객체를 불러와 callable하거나 한 그것을 생성하고, 전달해주고,  
 한번 들어온 소켓은 캐쉬에 저장되어 다시 주소를 로드할 필요 없이 연결을 보장하는 스트림인 TCP이기 때문에,  
 그것을 ```clientsock.on("data", callback) clientsock.on("exit",callback)``` 로 이벤트를 바인딩 해주어, 클래스 내부에 생성해놓은 static리스트에 넣어놓기만 하면, 알아서 처리해주니,  
 정확한 과정에 대해서 정말 전혀 몰라도, 아무런 문제없이 돌아가는 것만 같다.

 - 반면 파이썬은 C를 그대로 가져온 인터프리터여서 그런지, 네이밍등 모양새가 절차적인 특성을 가지고 있다.  
 그렇다고 해도, 축약되는 부분은 있다. 하지만 이 정도는 박수 받아야 할 정도라는 것은 안다.

> 하지만 정확한 절차에 대해서 어떤 과정이 필요한지, 자세히 알아볼 필요가 있다. C의 코드를 기술한다.
____
## 3. C로 작성된 TCP소켓 어플리케이션
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/types.h>
#include <sys/socket.h>

void error_handling(char *message);

int main(int argc, char **argv)
{
    int serv_sock;
    int clnt_sock;
    struct sockaddr_in serv_addr;
    struct sockaddr_in clnt_addr;
    int clnt_addr_size

    char message[]="Hello World!\n";

    if(argc != 2){
        printf("Usage : %s <port> \n",argv[0]);
        exit(1);
    }

    serv_sock = socket(PF_INET, SOCK_STRAM, 0); /*서버 소켓 생성*/
    if(serv_sock == -1)
        error_handling("socket() error");
    
    memset(&serv_addr, 0, sizeof(serv_addr))
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);
    serv_addr.sin_port=htons(atoi(argv[1]));

    if(bind(serv_sock, (struct sockaddr*) &serv_addr, sizeof(serv_addr)==-1))
    /*소켓에 주소 할당*/
        error_handling("bind() error");
    
    if(listen(serv_sock, 5) == -1) /*연결 요청 대기상태로 진입*/
        error_handling("listen() error");
    
    clnt_addr_size = sizeof(clnt_addr);

    clnt_sock = accept(serv_sock, (struct sockaddr*)&clnt_addr, &clnt_addr_size);
    /*연결요청 수락 함수accept*/

    if(clnt_sock==-1)
        error_handling("accept() error");
    
    write(clnt_sock, message, sizeof(message)); /*데이터 전송*/
    close(clnt_sock); /*연결 종료*/

    return 0;
}

void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

1. 서버소켓을 생성한다.(PF_INET:인터넷프로토콜, SOCK_STREAM:TCP방식, 0)
2. 서버소켓에 주소 할당 35~38 줄에서 주소정보 구조체 변수를 초기화해주고,  
145에서 서버 소켓에 bind
3. 서버가 주소를 가졌으니, 연결오청 대기상태로 들어가야함.  
149에서 listen(서버소켓, 연결대기 시퀸스 길이)
4. 대기 큐에서 대기하고 있는 클라이언트의 연결 요청에 대해서, 수락.  
154에서 accept를 통해 대기큐에 첫번째로 기다리고 있는 클라이언트의 연결요청을 수락하고 있다. 
5. accept 함수 호출 시 리턴된 파일 디스크립터를 가지고 클라이언트와 데이터를 주고받을 차례.  
일반적으로 파일을 가지고 입출력하던 방식과 동일하게 데이터 주고받음.  
write(클라이언트소켓, message, sizeof)는 시스템이 제공해주는 low Level I/O 이다.  
이를 통해 데이터를 전송한다. I/O-bound하다는 것은 여기서 시작된다.  
6. 이제 클라이언트에게 전송할 소켓을 닫아주면서  연결을 종료한다.

***
