---
title: multiple io, select() and poll()
p: linux/select_poll
date: 2019-08-22 15:27:08
tags: ['linux']
---

## index

1. [**다중 입출력**][i1]
2. [**select**][i2]
3. [**poll**][i3]
4. [**poll vs select**][i4]
5. [**epoll**][i5]
___

## 다중_입출력

[i1]: #다중_입출력

종종 키보드입력(stdin)과 IPC, 그리고 여러 파일 사이에서 일어나는 입출력을 처리하면서, 하나 이상의 파일디스크립터를 블록할 필요가 있다.

다중 입출력은 어플리케이션이 여러개의 파일 디스크립터를 동시에 블록하고,  
그 중 하나라도 블록되지 않고, 읽고 쓸 준비가 되면, 알려주는 기능을 제공한다.  
따라서 다음과 같은 설계방식을 따르는 어플리케이션을 위한 중심점이 된다.

1. 다중 입출력: 파일디스크립터중 하나가 입출력이 가능할때, 알려준다.
2. 준비가 됐나? 준비된 파일 디스크립터가 없다면, 하나 이상의 파일 디스크립터가 준비될 때까지 잠든다.
3. 꺠어나기. 어떤 파일 디스크립터가 준비됐나?
4. 블록하지 않고, 모든 파일 디스크립터가 입출력을 준비하도록 관리한다.
5. 1 로 돌아가서 다시 시작한다.

리눅스는 `select, poll, epoll`인터페이스 라는 세가지를 제공한다.

## select

[i2]: #select

```c
#include <sys/select.h>

int select(int n, fd_set *readfds, fd_set *writefds, fd_set *execptfds, struct timeval *timeout);

FD_CLR(int fd, fd_set *set);
FD_ISSET(int fd, fd_set *set);
FD_SET(int fd, fd_set *set);
FD_ZERO(fd_set *set);
```

`select()`호출은 파일 디스크립터가 입출력을 수행할 준비가 되거나 옵션으로 정해진 시간이 경과할 때 까지만 블록된다.

감시 대상 파일 디스크립터는 세가지 집합으로 나뉘어 각각 다른 이벤트를 기다린다.

* `readfds`: 블록되지 않고 데이터 읽기가 가능한지 (데이터가 없으면 block, read()작업이 가능한지) 감시한다.
* `wrtiefds`: 블록되지 않고 `write()`가 가능한지 감시한다.
* `exceptfds`: 예외가 발생했거나, 대역을 넘어서는 데이터(소켓에만 적용)가 존재하는지 감시.

호출이 성공하면, 각 집합은 요청받은 입출력 유형을 대상으로 입출력이 준비된 파일 디스크립터만 포함되도록 변경된다.  
예를 들어 7, 9두개의 파일디스크립터가 `readfds`에 들어있다고 한다면 호출이 반환될 때, 7이 집합에 남아있을 경우,  
7번은 블록없이 읽기가 가능하다.

첫번째 인자인 n은 파일 디스크립터 집합에서 가장 큰 파일 디스크립터 숫자에 1을 더한 값이다.  
따라서 select()를 호출하려면 파일 디스크립터에서 가장 큰 값이 무엇인지 알아내서 이 값에 1을 더해 첫 인자에 넘겨야 한다.  
`timeout` 인자는 `timeval구조체`를 가르키는 포인터이며 이 구조체는 다음과 같이 정의되어 있다.

```c
#include <sys/time.h>

struct timeval {
    long tv_sec; // 초
    long tv_usec; // 마이크로 초
}
```

이 인자가 NULL이 아니면 select()호출은 입력이 준비된 파일 디스크립터가 없을 경우에도 tv_sec초와 tv_usec 마이크로 초 이후에 반환된다.

고전적인 유닉스 시스템에서는 select()호출이 반환된 다음에 이 구조체가 정의되지 않은 상태로 남아 있으므로, 매번 호출전에  
파일 디스트립터 집합과 더불어 다시 초기화를 해줘야 한다. 실제로 최근 리눅스 버전은 이 인자를 자동으로 수정해서 남아있는 시각으로 값을 설정한다.

따라서 Timeout이 5초고 파일 디스트립터가 준비되기 까지 3초가 경과했다면, `tv.tv_sec`은 반환될때 2를 담고 있을 것이다.  
timeout에 설정된 두 값이 모두 0이면, 호출은 즉시 반환되며 호출하는 시점에서 대기중인 이벤트를 알려주지만, 그 다음 이벤트를 기다리지 않는다.

`select`에서 사용하는 fds(집합)은 직접 조작하지 않고 매크로를 사용해서 관리한다.

대다수의 시스템에서는 fds를 비트 배열처럼 간단한 방식으로 구현하고 있다.

### 매크로

* FD_ZERO: `FD_ZERO(&writefds);`
  FD_ZERO는 지정한 집합내 모든 파일 디스크립터를 제거한다. 이 매크로는 항상 select()를 호출하기 전에 사용해야 한다.
* FD_SET: `FD_SET(fd, &writefds);`
  주어진 집합에 파일 디스크립터를 추가한다.
* FD_CLR: `FD_CLR(fd, &writefds);`
  주어진 집합에서 파일디스크립터 하나를 제거한다.
* FD_ISSET: `FD_ISSET(fd, &readfds);`
  주어진 집합에 파일디스크립터가 존재하는지 확인한다. 존재하면 0이 아닌 정수를 반환한다.  
  `select()`호출이 반환된 다음에 파일디스크립터가 입출력이 가능한 상태인지 확인하기 위해 사용된다.

### 예제

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

#define TIMEOUT 5
// 타입 아웃 초
#define BUF_LEN 1024
// 읽기 버퍼 크기

int main(void)
{
    struct timeval tv;
    fd_set readfds;
    int ret;

    // 표준 입력에서 입력을 기다리기 위한 준비
    FD_ZERO(&readfds); // select 호출 전 반드시 사용 해줘야.
    FD_SET(STDIN_FILENO, &readfds); // IN은 데이터가 들어오면 읽을 수 있는 FD

    // select가 5초 동안 기다리도록 timeval구조체 설정
    tv.tv_sec = TIMEOUT;
    tv.tv_usec = 0;

    // 입력 대기 시작
    ret = select(STDIN_FILENO + 1, &readfds, NULL, NULL, &tv);
    //  Readable한 STDIN_FILENO에 대해서 대기, 감시하는 select 선언
    if (ret == -1)
    {
        perror("select");
        return 1;
    }
    else if (!ret)
    {
        printf("%d seconds elapesd.\n", TIMEOUT);
        return 0;
    }
    // 이 아래부터는 select가 0이 아닌 양수를 반환했다는 의미
    // 대기중인 집합 중 하나라도 돌아왔다
    // 그것을 FD_ISSET으로 선별해서, 그것을 READ혹은 WRITE한다.
    // 바로 실행할 수 있으니 빠르게 블록되고 해제된다.
    // 그 전까지는 FD를 감시하며 LOOPING하는 쓰레드가 동작
    // 파일 디스크립터에서 즉시 읽기가 가능하다.

    if (FD_ISSET(STDIN_FILENO, &readfds))
    {
        char buf[BUF_LEN + 1];
        int len;

        // read not block
        len = read(STDIN_FILENO, buf, BUF_LEN);
        if (len == -1)
        {
            perror("READ");
            return 1;
        }
        else if (len)
        {
            buf[len] = '\0';
            printf("read %s\n", buf);
        }
        return 0;
    }
    fprintf(stderr, "This should not happen!\n");
    return 1;
}
```

## POLL

[i3]: #poll

```c
#include <poll.h>

int poll(struct pollfd *fds, nfds_t nfds, int timeout);

struct pollfd{
    int fd; // 파일 디스크립터
    short events; // 대기 이벤트
    short revents; // 발생이벤트
}
```

비트마스크 기반의 세가지 파일디스크립터 집합을 사용하는 `Select`와 달리,  
`poll`은 fds가 가리키는 단일 pollfd구조체 배열을 nfds개수 만큼 사용한다.

각 구조체의 `events`필드는, 그 파일 디스크립터에서 감시할 이벤트의 비트마스크 이다.

설정 가능한 이벤트는 아래와 같다.

|    name    | desc                        |
| :--------: | :-------------------------- |
|   POLLIN   | 읽을 데이터가 존재                  |
| POLLRDNORM | 일반 데이터를 읽을 수 있다.            |
| POLLRDBAND | 우선권이 있는 데이터를 읽을 수 있다.       |
|  POLLPRI   | 시급히 읽을 데이터가 존재한다.           |
|  POLLOUT   | 쓰기가 블록되지 않을 것이다.            |
| POLLWRNORM | 일반 데이터 쓰기가 블록되지 않을 것이다.     |
| POLLWRBAND | 우선권이 있는 데이터쓰기가 블록되지 않을 것이다. |
|  POLLMSG   | SIGPOLL메세지가 사용가능하다.         |

그리고 `revents`필드에는 아래 이벤트가 추가로 설정될 수 있다.

|   name   | desc                         |
| :------: | :--------------------------- |
|  POLLER  | 주어진 파일 디스크립터에 에러가 있다.        |
| POLLHUP  | 주어진 파일 디스크립터에서 이벤트가 지체되고 있다. |
| POLLNVAL | 주어진 파일 디스크립터가 유효하지 않다.       |

예를 들어 FD에서 읽가와 쓰기를 감시하려면, events를 `POLLIN | POLLOUT`으로 설정한다.  
호출이 반환되면, `pollfd`구조체 배열에서 원하는 파일 디스크립터가 들어있는 항목을 찾아  
`revents`에 해당 플래그가 켜져있는지 확인한다.

### 반환과 에러코드

`poll()`호출이 성공하면, `revents`필드가 0이 아닌 구조체의 개수를 반환한다.  
이벤트가 발생하기 전에 타입아웃이 발생한다면 0을 반환한다.  
에러가 발생하면 -1을 반환하며, errno를 아래중 하나로 설정한다.

|  name  | desc                               |
| :----: | :--------------------------------- |
| EBADF  | 주어진 구조체의 파일디스크립터가 유효하지 않다.         |
| EFAULT | fds를 가리키는 포인터가 프로세스 주소공간을 벗어난다.    |
| EINTR  | 이벤트를 기다리는중 시그널이 발생했다. 다시 호출이 필요하다. |
| EINVAL | nfds인자가 RLIMIT_NOFILE값을 초과했다.      |
| ENOMEM | 요청을 완료하기위한 메모리가 부족하다.              |

### 예시

```c
#include <stdio.h>
#include <unistd.h>
#include <poll.h>
#define TIMEOUT 5

int main(void)
{
    struct pollfd fds[2];
    int ret;

    // 표준 입력에 대한 이벤트 감지를 위한 세팅
    fds[0].fd = STDIN_FILENO;
    fds[0].events = POLLIN;

    // 표준 출력에 쓰기가 가능한지 감지하기 위한 준비
    fds[1].fd = STDOUT_FILENO;
    fds[1].events = POLLOUT;

    // 블록 시작
    ret = poll(fds, 2, TIMEOUT * 1000);
    if (ret == -1)
    {
        perror("poll");
        return 1;
    }

    if (!ret)
    {
        printf("%d seconds elapsed.\n", TIMEOUT);
        return 0;
    }

    if (fds[0].revents & POLLIN)
    {
        printf("stdin is readable\n");
    }
    if (fds[1].revents & POLLOUT)
    {
        printf("stdout is writable\n");
    }
    return 0;
}
```

## POLL과_SELECT비교

[i4]: #poll과_select비교

* `poll()`은 가장 높은 파일디스크립터 값에 1을 더해서 인자로 전달할 필요가 없다.
* `poll()`은 파일 디스크립터 숫자가 큰 경우에 좀 더 효율적을 동작한다.  
  `select()`로 값이 900인 단일파일 디스크립터를 감시한다고 할 때,  
  커널은 매번 전달된 파일디스크립터 집합에서 900번째 비트까지 일일이 검사해야한다.
* `select()`의 파일 디스크립터 집합은 크기가 정해져 있으므로, trade-off 발생.  
  집합은 크기가 작으면, `select()`가 감시할 최대 디스크립터 개수를 제약하며, 집합의 크기가 크면 비효율적이다.  
  큰 비트마스크에 대한 연산은 비효율적이며, 파일 디스크립터가 연속적이지 않을경우 특히 심각하다.  
  `poll()`을 사용하면, 딱 맞는 키기의 배열 하나만 사용하면 된다.
* `select()`를 사용하면, 디스크립터 집합을 반환하는 시점에 재구성되므로  
  잇다라 호출하게 되면, 파일디스크립터 집합 재초기화 필요함.  
  `poll()`시스템 콜은 입력과 출력을 분리하므로, 변경없이 배열을 재사용 가능하다.
* `select()`의 timeout반환하게 되면 미정의 상태가 된다.  
  따라서 코드의 이식성을 높이려면, timeout인자를 재초기화 해야한다.  
  `pselect()`를 사용할 경우에는 이런 문제가 없다.

select()시스템 콜에도 몇가지 장점이 있다.

* `select()`는 상대적으로 이식성이 높아 거의 모든 유닉스에서 지원한다.
* `select()`는 타입아웃 값으로 마이크로초까지 지정할 수 있다. `poll()`은 milie-sec단위로 지정할 수 있다.

> `epoll()`은 `poll` `select`보다 훨씬 뛰어난 리눅스 입출력 멀티플렉싱 인터페이스이다. 추후에 정리한다.
___

## epoll

[i5]: #epoll

`poll`과 `select`는 실행할 때 마다, 전체 파일 디스크립터를 요구한다.  그러면 커널은 검사해야할 파일 리스트를 다 살펴본다.p

### epoll_사용법

```c
#include <sys/epoll.h>
int epfd;

int epoll_create1(int flags);

epfd = epoll_create1(0);
if(epfd<0){
    perror("epoll_create1);
}
```

`epoll_create1()`에서 반환하는 fd는 폴링이 끝난뒤 반드시 `close()`해줘야 한다.  에러 발생시,

* EINVAL: 잘못된 flags인자
* EMFILE: 사용자가 열 수 있는 최대 파일 초과
* ENFILE: 시스템에서 열 수 있는 최대파일을 초과
* ENOMEM: 작업을 수행하기 위한 메모리 부족

-1 을 반환하고 errno를 위 중 하나로 설정한다.
___

### epoll_create에서 epoll_create1으로

`epoll_create()`는 구식 메서드며, epoll_create1()으로 대체됐다.  
epoll_create()는 아무런 인자를 받지 않으며 그대신 size인자를 받는데, 이는 사용되지 않음.

size는 감시할 파일 디스크립터 개수에 대한 힌트로 사용되는데,  **최신 커널에서는 동적으로 요청된 자료구조로 크기를 정하며, 이 인자는 단지 0보다 크기만 하면 된다.**

만약에 size값이 0이거나 0보다 작다면 `EINVAL`을 반환한다.
___

### epoll_제어

`epoll_ctl()`시스템 콜은 주어진 epoll컨텍스트에 파일 디스크립터를 추가하거나 삭제할 때 사용한다.

```c
#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

// sys/epoll.h에 정의된 epoll_event 구조체.
struct epoll_event{
    __u32 events; // events
    union {
        void *ptr;
        int fd;
        __u32 u32;
        __u64 u64;
    }
    data;
}
```

`epoll_ctl()`호출이 성공하면 해당 epoll인스턴스는 `epfd`파일 디스크립터와 연결된다.  
op인자는 fd가 가리키는 파일에 대한 작업을 명시한다.  

* op의 값
  * EPOLL_CTL_ADD: edfd와 연관된 epoll인스턴스에 fd와 연관된 파일을 감시하도록 추가하며, 각 이벤트는 event인자로 정의한다.
  * EPOLL_CTL_DEL: edfd와 연관된 epoll인스턴스에 fd를 감시하지 않도록 삭제한다.
  * EPOLL_CTL_MOD: 기존에 감시하고 있는 fd에 대한 이벤트를 event에 명시된 내용으로 갱신한다

event인자는 그 작업동작에 대한 설명을 담고 있다.

* epoll_event.events의 값(OR연산으로 여러 이벤트를 묶을 수 있음)
  * EPOLLERR: 해당 파일에서 발생하는 에러 상황. 이 이벤트는 따로 지정하지 않아도 항상 감시한다.
  * EPOLLET: 파일을 감시할 때, `edge-trigger`를 사용한다. 기본 동작은 레벨트리거 방식.
  * EPOLLHUP: 파일에서 발생하는 행업을 감시한다. 이 이벤트도 따로 지정하지 않아도 항상 감시한다.
  * EPOLLIN: 파일 읽기가 지연되지 않고 바로 가능한지 감시.
  * EPOLLONESHOT: 이벤트 발생 후 파일을 한번 읽고 나면 더 이상 감시하지 않는다. 이를 다시 활성화 하려면, `EPOLL_CTL_MOD`를 통해서 새로운 이벤트 값을 설정해야 한다.
  * EPOLLOUT: 파일 쓰기가 지연되지 않고 바로 가능한지 감시.
  * EPOLLPRI: 즉 시 읽어야한 OOB(TCP에서 말하는 OUT OF BAND데이터로, 전송순으로 받는 데이터의 순서를 무시하고 보내는 메세지. 잘 사용 안됨)데이터가 있는지 감시.

epoll_event구조체의 data필드는 사용자 데이터를 위한 필드다.  
이 필드에 담긴 내용은 이벤트 발생 후 사용자에게 반환될 때 함께 반환된다.  
주로 `event.data.fd`를 fd로 채워서 이벤트가 발생 했을 때,  
어떤 파일 디스크립터를 들여다 봐야 하는지 확인하는 용도로 사용한다.

`epoll_ctl()`호출이 성공하면 0을 반환, 실패시 -1반환.  
errno를 다음 중 한 가지로 설정.

* EBADF: epdf가 유효한 epoll인스턴스가 아니거나, fd가 유효한 파일 디스크립터가 아니다.
* EEXIST: op가 EPOLL_CTL_ADD인데, fd가 이미 epfd와 연결되어 있다.
* EINVAL: epfd가 epoll인스턴스가 아니거나, epfd가 fd와 같다. 또는 잘못된 op값이 사용.
* ENOENT: op가 EPOLL_CTL_MOD, 혹은 EPOLL_CTL_DEL인데, fd가 epfd와 연결되지 않았다.
* ENOMEM: 해당 요청을 처리하기에는 메모리가 부족하다.
* EPERM: fd가 epoll을 지원하지 않는다.

```c
#include <sys/epoll.h>

FILE *stream;
int fd, epfd;
struct epoll_event event;
int ret;

stream = fopen("sample", "r");
if (!stream){
    perror("fopen");
    return 1;
}

fd = fileno(stream);
if (fd == -1){
    perror("fileno");
    return 1;
}

epfd = epoll_create1(0);
if (epfd<0){
    perror("epoll_create1");
}

event.data.fd = fd;
event.events = EPOLLIN | EPOLLOUT;

// 이벤트 epfd에 추가
ret = epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &event);
// 기존 이벤트 변경
ret = epoll_ctl(epfd, EPOLL_CTL_MOD, fd, &event);
// 기존 이벤트 삭제
ret = epoll_ctl(epfd, EPOLL_CTL_DEL, fd, &event);

if (ret){
    perror("epoll_ctl");
    close(epfd);
    return 1;
}

close(epfd);
return 0;
```

___

### epoll로_이벤트_대기하기

```c
#include <sys/epoll.h>
#define MAX_EVENTS 64
/*
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
*/

struct epoll_event *events; // event를 포인터를 담을 수 있는 공간으로
int nr_events, i, epfd;

#include <stdlib.h>
events = malloc(sizeof(struct epoll_event)* MAX_EVENTS);
if (!events){
    perror("malloc");
    return 1;
}

nr_events = epoll_wait(epfd, events, MAX_EVENTS, -1);
if (nr_events < 0){
    perror("epoll_wait");
    free(events);
    return 1;
}

for (i = 0; i < nr_events; i++){
    printf("event=%ld on fd=%d\n", events[i].events, events[i].data.fd);
    /*
    이제 events[i].data.fd에 대한 events[i].events를
    블록하지 않고 처리할 수 있다.
    */
}

free(events);

```

1. `epoll_wait()`를 호출
2. timeout밀리 초 동안 epoll인스턴스인 epfd에 등록한 파일의 이벤트를 기다린다.
3. 성공시 events에는 해당 이벤트, 파일이 일기나 쓰기가 가능한 상태인지 나타내는  
   `epoll_event`구조체에 대한 포인터가 기록.  
   최대 maxevents만큼의 이벤트가 기록된다.  
   발생한 이벤트 개수를 반환하며 에러시 -1, errno를 다음중 하나로 기록한다.
4. `epoll_wait()`가 결과를 반환하면, epoll_event구조체의 evens필드에는 발생 이벤트가 기록된다.  
   data필드에는 사용자가 `epoll_ctl()`을 호출하기 전에 설정한 값이 담겨있다.

* EBADF: epfd가 유효한 fd가 아니다.
* EFAULT: events가 가리키는 메모리에 대한 쓰기 권한이 없다.
* EINTR: 시스템 콜이 완료되거나 타임아웃 전에 시그널이 발생해서 동작을 멈추었다.
* EINVAL: epfd가 유효한 epoll인스턴스가 아니거나 maxevents값이 0이하이다.

timeout이 0이면 즉히 반환 반환값은 0이다.  
timeout이 -1이면 이벤트가 발생 할 때까지 해당 호출은 반환되지 않는다.(블로킹 상태)
___

### 에지트리거와_레벨트리거

`epoll_ctl()`로 전달되는 event인자의 events필드를 EPOLLET로 설정하면,  
fd에 대한이벤트 모니터가 레벨 트리거가 아닌 엣지 트리거로 동작한다.

유닉스 파이프로 통신하는 입출력에 대한 사례는 아래와 같다.

1. 출력하는 측에서 파이프에 1kb만큼 데이터를 쓴다.
2. 입력 받는 쪽에서는 파이프에 대해서 `epoll_wait()`를 수행하고
3. 파이프에 데이터가 들어와서 읽을 수 있는 상태가 되기를 기다린다.

* 레벨 트리거로 이벤트를 모니터링하면, 2의 `epoll_wait()`호출은 즉시 반환하며,  
파이프가 읽을 준비가 되었음을 알려준다.
* 엣지 트리거로 모니터링하면 1단계가 완료될 때까지, 호출은 반환되지 않는다.

기본 동작방식은 **레벨트리거**방식이다. **`poll()`**과 **`select()`**가 동작하는 방식이 레벨트리거 방식인데,  
대부분 개발자들이 생각하는 방식 이기도 하다.

**엣지 트리거**방식은 일반적으로 **논블로킹 입출력**을 활용하는 접근방식을 **요구**하며, `EAGAIN`을 주의깊게 확인해야 한다.
