---
title: 소켓과 스트림을 이해하기 위한 고급 IO함수
p: c/io_functions_advanced
date: 2019-08-27 17:23:28
tags: ['c']
---


## index

1. [**버퍼링 제어하기**][i1]
2. [**스레드 세이프(thread safe)**][i2]
3. [**벡터 입출력**][i3]
___

## 버퍼링_제어하기

[i1]: #버퍼링_제어하기

표준 입출력은 세가지 유형으로 사용자 버퍼링을 구현하고,  
버퍼의 유형과 크기를 다룰 수 있는 인터페이스를 제공한다.  
각각의 버퍼링 타입은 저마다의 목적이 있으니 상황에 맞게 사용해야 한다.

|  type  |                                          desc                                           |                          ie                          |
| :----: | :-------------------------------------------------------------------------------------: | :--------------------------------------------------: |
| 버퍼 미사용 |                       사용자 버퍼를 사용하지않고 바로 커널로 데이터를 보낸다. 딱히 이점이 없다.                        |            **표준에러`stderr`** 외에 거의 사용하지 않음            |
|  행 버퍼  | 행 단위 버퍼링 수행. 개행문자가 나타나면 버퍼 내용을 커널로 보낸다. 화면출력을 위한 스트림일 경우 유용, 화면출력 메세지는 개행문자로 구분되기 때문이다. | **표준 출력`stdin`** 처럼 터미널에 연결된 스트림에서 기본적으로 사용하는 버퍼링 방식 |
| 블록 버퍼  |        고정된 바이트수로 표현되는 블록단위로 버퍼링을 수행. 파일에 적합하여 기본적으로 파일과 관련된 모든 스트림은 블록버퍼를 사용한다.         | **파일 스트림(FILE)**. 표준입출력 에서는 블록버퍼링을 `full`버퍼링이라고 한다.  |

대부분 기본유형의 버퍼링을 사용하는 편이 올바르고 최선이나, 표준입출력은 버퍼링 방식을 제어할 수 있는 인터페이스를 제공한다.

```c
#include <stdio.h>

int setvbuf(FILE *stream, char *buf, int mode, size_t size);
```

1. `stream`의 **버퍼링 유형**을 `mode`로 설정한다.
    * mode
        1. `IONF`: 버퍼 미사용
        2. `IOLBF`: 행 버퍼
        3. `IOFBF`: 블록 버퍼
2. `buf`와 `size`를 무시하는 `IONF`를 제외하고,
3. 나머지는 `size`바이트 크기의 버퍼를 가리키는 `buf`를 주어진 `stream`을 위한 버퍼로 사용한다.
4. 만약 `buf`가 `NULL`이면, **glibc**가 자동적으로 지정된 크기만큼 메모리를 할당한다. (파일 저장을 생각하면 아마 1024이상 4kb, 8kb 정도가 아닐까?)
5. `setvbuf`함수는 스트림을 연다음, 다른 연산(읽기 쓰기 등.)수행하기 전에 호출해야 한다.
6. 성공시 0을 반환, 실패시 그 외의 값을 반환.

제공된 버퍼는 스트림이 닫힐 때까지 반드시 존재해야한다.  
흔히 스트림을 받기 전에 끝나는 스코프 내부의 자동변수로 버퍼를 선언하는 실수를 한다.  
특히 `main()`에서 지역변수로 버퍼를 만든 후, 스트림을 명시적을 닫지 않는 경우.

아래 코드에는 버그가 있다.

```c
#include <stdio.h>

int main(void){
    char buf[BUFSIZ];
    // stdout을 bufsize크게에 맞춰 블록 버퍼로 설정한다.

    setvbuf(stdout, buf, _IOFBF, BUFSIZ);
    printf("ARRR!\n"); // stdout에 버퍼에 들어가고 개행문자를 확인함과 stdout buf에서 fflush되서 화면에 출력된다.

    return 0;
    // buf는 스코프를 벗어나고 해제된다. 하지만 stdout을 닫지 않았다!!
}
```

일반적으로 개발자들은 스트림을 다룰 때 버퍼링에 대해 고민할 필요가 없다.  
표준 에러를 제외하고, 터미널은 행 버퍼링으로 동작하는게 맞고 파일은 블록버퍼링을 사용하는게 맞다. 그게 전부다.

블록 버퍼링에서 버퍼의 기본크기는 `BUFSIZ`이며 `<stdio.h>`에 정의되어 있다.  
이 값은 일반적인 블록크기의 정수배인 최적의 값이다.  
**(64비트 운영체제인 내 컴퓨터에서는 `8192bytes`)**
___

## 스레드_세이프 :Thread safe

[i2]: #스레드_세이프

pass
___

## 벡터_입출력 :Vector_io

[i3]: #벡터_입출력

> 버퍼세그먼트의 집합을 벡터라 한다.

한번의 시스템콜을 사용해서, 여러개의 버퍼벡터(세그먼트)에 쓰거나, 읽어들일 때 사용.
백터입출력은 선형입출력 메서드에 비해 다음과 같은 장점을 가지고 있다.

1. 자연스러운 코딩 패턴
    * 미리 정의된 구조체가 여러 필드에 걸쳐 데이터가 분리되어 있는 경우,  
    벡터입출력을 사용하면 직관적인 방법으로 조작할 수 있다.
2. 효율
    * 하나의 벡터 입출력 연산은 여러번의 선형 입출력 연산을 대체할 수 있다.
3. 성능
    * 시스템콜의 횟수를 줄일 뿐 아니라, 내부적으로 선현 입출력 구현에 비해 좀 더 최적화된 구현을 제공.

### readv_writev

`read()`, `write()`와 동일하게 동작하지만, 여러 개의 버퍼를 사용한다는 점에서 구분된다.

```c
#include <sys/uio.h>

struct iovec{
    void *iov_base; // 버퍼의 시작 포인터
    size_t iov_len; // 버퍼의 바이트 수
}
```

* `readv()`

파일디스크립터 `fd`에서 데이터를 읽어 count개수만큼 `iov`버퍼에 저장.

```c
#include <sys/uio.h>

ssize_t readv(int fd, const struct iovec *iov, int count);
```

* `writev()`

count개수만큼의 `iov`버퍼에 있는 데이터를 파일디스크립터`fd`에 저장.

```c
#include <sys/uio.h>

ssize_t writev(int fd, const struct iovec *iov, int count);
```

둘 다 성공시 읽거나 쓴 바이트 개수를 반환한다.  
즉, `count * iov_len`과 같아야 하며, 에러가 발생하면 -1을 반환하고 errno를 설정한다.

표준에러에서 추가로 2가지 에러의 상황이 있다.
1. 반환값의 자료형이 `ssize_t`이기 때문에 `SSZIE_MAX`보다 큰 값을 전달했다고 리턴하면 데이터는 전송되지 않고,  
   -1을 반환하며 errno는 EINVAL로 설정한다.  
   (`2**32-1`을 바이트 취급하면 `4gb`)
2. POSIX는 count가 반드시 0보다 크고 `IOV_MAX`와 같거나 작아야한다.  
   리눅스에서  IOV_MAX값은 1024로 정의.  
   만약 count가 0이면 0을 반환하고(0회 실행했으니 데이터전송량 0바이트),  
   IOV_MAX보다 크다면, 데이터는 전송되지 않고 -1을 반환하며 errno는 EINVAL로 설정한다.

버프의 기본사이즈 인 블록사이즈(메모리 접근 최소 단위)가 내 운영체제에서는 8kb 이기 때문에,  
`1024bytes`(를 부분 버퍼로)*`8`(개)이면 데이터가 딱 맞게 **한번의 시스템 콜로** `1kb 8개 버퍼`를 처리하는 것이다. 

### writev_예시

* Code

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <sys/uio.h>
#include <unistd.h>
int main()
{
    struct iovec iov[3];
    ssize_t nr;
    int fd;
    int i;
    int res;
    char *buf[] = {
        "The term buccaneer comes from the word boucan.\n",
        "A boucan is a wooden frame used for cooking meat.\n",
        "Buccaneer is the West Indies name for a pirate.\n"};

    fd = open("buccaneer.txt", O_WRONLY | O_CREAT | O_TRUNC);
    if (fd == -1)
    {
        perror("open");
        return 1;
    };

    for (i = 0; i < 3; i++)
    {
        iov[i].iov_base = buf[i];
        iov[i].iov_len = strlen(buf[i]) + 1;
        printf("len: %li\n", strlen(buf[i]));
    };

    nr = writev(fd, iov, 3);
    if (nr == -1)
    {
        perror("writev");
        return 1;
    };
    printf("wrote %ld bytes\n", nr);
    res = close(fd);
    if (res == -1)
    {
        perror("close");
        return 1;
    };
    return 0;
}
```

* output

```bash
len: 47
len: 50
len: 48
wrote 148 bytes
```

* strace

```bash
execve("./writev.out", ["./writev.out"], 0x7ffecf1aa940 /* 27 vars */) = 0
brk(NULL)                               = 0x55bf90861000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=90249, ...}) = 0
mmap(NULL, 90249, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fe81dace000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\260\34\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=2030544, ...}) = 0 # fd에 대한 정보 갱신
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fe81dacc000 # 가상 메모리의 데이터 매핑 (메모리를 map하다.)
mmap(NULL, 4131552, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fe81d4cd000
mprotect(0x7fe81d6b4000, 2097152, PROT_NONE) = 0
mmap(0x7fe81d8b4000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e7000) = 0x7fe81d8b4000
mmap(0x7fe81d8ba000, 15072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fe81d8ba000
close(3)                                = 0
arch_prctl(ARCH_SET_FS, 0x7fe81dacd4c0) = 0 # 아키텍쳐의 특수 프로세스나 쓰레드 상태를 설정함(성립시킴) 프로세스에 할당되는 쓰레드에 실행중인 코드블록의 주소를 전달하는?
mprotect(0x7fe81d8b4000, 16384, PROT_READ) = 0
mprotect(0x55bf90670000, 4096, PROT_READ) = 0
mprotect(0x7fe81dae5000, 4096, PROT_READ) = 0 # 메모리 주소에 대한 권한 변경
munmap(0x7fe81dace000, 90249)           = 0 # 메모리 해제
openat(AT_FDCWD, "buccaneer.txt", O_WRONLY|O_CREAT|O_TRUNC, 0100750) = 3
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 1), ...}) = 0
brk(NULL)                               = 0x55bf90861000 # 실행하는 프로세스에 대해 메모리 세그먼트 변경, 추가 등.
brk(0x55bf90882000)                     = 0x55bf90882000
write(1, "len: 47\n", 8)                = 8
write(1, "len: 50\n", 8)                = 8
write(1, "len: 48\n", 8)                = 8
writev(3, [{iov_base="The term buccaneer comes from th"..., iov_len=48}, {iov_base="A boucan is a wooden frame used "..., iov_len=51}, {iov_base="Buccaneer is the West Indies nam"..., iov_len=49}], 3) = 148
write(1, "wrote 148 bytes\n", 16)       = 16
close(3)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```

___

### readv_예시

* code

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <sys/uio.h>
#include <unistd.h>

int main()
{
    char foo[48], bar[51], baz[49];
    struct iovec iov[3];
    ssize_t nr;
    int fd, i;

    fd = open("buccaneer.txt", O_RDONLY);
    if (fd == -1)
    {
        perror("open");
        return 1;
    };

    iov[0].iov_base = foo;
    iov[0].iov_len = sizeof(foo);

    iov[1].iov_base = bar;
    iov[1].iov_len = sizeof(bar);

    iov[2].iov_base = baz;
    iov[2].iov_len = sizeof(baz);

    nr = readv(fd, iov, 3); // 블럭 단위 버퍼로 저장한다. BUF_SIZ(8kb)
    // 파일을 읽는 물리 블록 단위는 4096bytes? 그 단위로 읽으면 재정렬이 필요없어.

    if (nr == -1)
    {
        perror("readv");
        return 1;
    };

    for (i = 0; i < 3; i++)
    {
        printf("%d: %s", i, (char *)iov[i].iov_base);
    };

    if (close(fd) == -1)
    {
        perror("close");
        return 1;
    };

    return 0;
}
```

* output

```bash
0: The term buccaneer comes from the word boucan.
1: A boucan is a wooden frame used for cooking meat.
2: Buccaneer is the West Indies name for a pirate.
```

> 사실 리눅스 커널 내부에 모든 입출력은 벡터 입출력이다.  
> `read()`와 `write()`구현 역시 하나짜리 세그먼트를 가지는 벡터 입출력으로 되어있다.
