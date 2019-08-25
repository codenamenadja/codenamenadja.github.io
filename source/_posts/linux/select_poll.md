---
title: multiple io, select() and poll()
p: linux/select_poll
date: 2019-08-22 15:27:08
tags: ['linux']
---

## index

1. [**다중 입출력**][i1]
2. [**select**][i2]
   3. [**pselect**]
3. [**poll**][i3]

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

## POLL_과_SELECT_비교

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