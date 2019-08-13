---
title: basic concept of linux / process and Thread
p: linux/basic_concept_procss_2
date: 2019-08-12 13:07:34
tags: ["linux"]
---

## Index

1. [**Process**][i1]
2. [**Thread**][i2]
3. [**프로세스의 계층 구조**][i3]
4. [**프로세스의 생성**][i4]

## Process

[i1]: #process

* 정의

    활성화 상태로 실행중인, 코드 세그먼트로 코드(실행 컨텍스트)가 이동한 프로그램.  
    데이터, 리소스, 상태, 가상화된 컴퓨터를 포함

1. 커널이 이해하는 실행파일 포멧으로 만들어져 실행가능한 오브젝트 코드로 형태
2. 리눅스에서 가장 일반적인 실행파일 포멧은 `ELF(Executable and Linkable Format)`
3. 실행 파일은 여러 섹션으로 구성
* 메타 데이터
* 코드
* 데이터

4.  위 섹션은 오브젝트 코드가 담긴 바이트 배열(FILE), 선형 메모리 공간에 적재됨
5. 섹션에 담긴 바이트들은 접근 권한이 같고 사용목적이 비슷하며, 동일하게 취급.

가장 중요한 공통 섹션

| 섹션                           | 설명                                                                                            |
| :--------------------------- | :-------------------------------------------------------------------------------------------- |
| 텍스트 섹션                       | 실행가능 코드, 상수, 변수와 같은 읽기전용 데이터. (읽기전용과 실행가능으로 표시                                                |
| 데이터 섹션                       | 정의된 값을 할당한 변수와 같은 초기화된 자료. 일반적으로 읽고 쓰기가 가능한 동적 데이터                                            |
| `BSS`(block storage segment) | 초기화되지 않은 전역 데이터. 함수 콜스텍처럼 곧 해제될 것으로 기대되는 것이 아닌, 프로그램의 실행과 종료에 생성되면 적지 않게 유지될 전역에 할당되고 사용될 데이터 |

* 프로세스는 커널이 중재하고 관리하는 다양한 시스템 리소스와 관련이 있다.
* 프로세스는 일반적으로 시스템 콜을 이용하여 리소스를 요청하고 조작한다.
  * 리소스들(타이머, 대기중인 시그널, 열린 파일, 네트워크 연결, 하드웨어, IPC매커니즘)
* 프로세스 리소스는 자신과 관련한 데이터의 통계정보를 포함하고 있으며, 해당 프로세스의 프로세스 디스크립터 형태로 커널 내부에 저장된다.

> 프로세스는 가상화를 위한 추상 개념이다.
> 선점형 멀티테스킹과 가상메모리를 지원하는 리눅스 커널은 가상화된 프로세서와 가상화된 메모리를 프로세스에 제공한다.

* 프로세스와 커널의 입장에서 본 가상화에 대하여

    커널은 프로세스에 단일 선형 주소공간(Single linear Address space)를 제공하므로, 프로세스 홀로 시스템에 존재하는 모든 메모리를 제어하는 것처럼 보인다.

    커널은 가상 메모리와 페이징 기법(swapping)을 사용해서 프로세스마다 다른 주소공간에서 동작하도록 만들기 때문에, 여러 프로세스가 시스템 상에서 공존.

    최신 프로세서는 운영체제가 독립적인 여러 프로세스 상태를 동시에 관리할 수 있도록 하며, 커널은 이런 하드웨어의 도움을 받아 가상화를 관리한다.

## Thread

[i2]: #thread

* 정의

각 프로세스는 실행 스레드를 하나 이상 포함한다.  
스레드는 프로세스 내부에서 실행하는 활동 단위.  
코드를 실행하고 프로세스 동작 상태를 유지하는 추상개념이다.

> 유닉스의 간결함을 중시하는 철학과 빠른 프로세스 생성시간, 견고한 IPC매커니즘 떄문에,  
> 전통적으로 유닉스 프로그램은 싱글 스레드였고, 스레드 기반으로 옮겨 가려는 요구사항이 적었다.

스레드의 구성요소

* 독자적 지역 변수를 저장하는 스텍
* 프로세서 상태
* 오브젝트 코드의 현재 위치(instruction Pointer)

스케쥴링을 통해 선점한 프로세스의 스레드의 IP를 CPU의 IR로 가져와서 실행됨.

기타 프로세스에 남아있는 대부분의 리소스는 모든 스레드의 공유자원이다.
___

### 리눅스 커널의 스레드 구현

스레드는 단순히 주소공간을 비롯 일부 리소스를 공유하는 일반적인 프로세스일 뿐이다.  
사용자 영역에서 리눅스는 `Pthread`라 하는 POSIX 1003.1c에 따라 쓰레드를 구현한다.  
`glibc`의 일부인 현재 리눅스 쓰레드의 구현체 명은 NPTL(`Nativa POSIX Threading Library`).

___

## 프로세스의_계층_구조

[i3]: #프로세스의_계층_구조

프로세스는 `PID`라는 고유 양수값으로 구분된다. - (첫번쨰 프로세스의 pid는 1)

리눅스에서 프로세스는 프로세스 트리라는 계층 구조를 형성한다.  
일반적으로 프로세스 트리는 `init`프로그램으로 알려진 첫 프로세스가 루트가 된다.

새로운 프로세스는 `fork()` 시스템 콜로 만들어진다.  
이 시스템 콜을 호출하는 기준 프로세스를 복사해서 다른 프로세스를 만든다.

첫번째 프로세스를 제외한 나머지 프로세스는 모두 부모-프로세스가 있고,  
부모-프로세스가 자식-프로세스 보다 먼저 종료 되면, 자식-프로세스를 부모-프로세스로 승격 시킨다.

프로세스가 종료되면 시스템에서 바로 제거하지 않고, 프로세스 일부를 메모리에 유지한다.  
자식-프로세스가 종료될 때 부모-프로세스가 상태를 검사 할 수 있도록 한다.

이를 `"종료된 프로세스를 기다린다."`고 표현한다.

만약 자식-프로세스가 종료되었는데 기다리는 부모-프로세스가 없다면, 이때 좀비-프로세스가 탄생한다.  
일상적으로 첫 init프로세스는 자신이 fork한 모든 프로세스를 기다려서 새로 생성된 프로세스가 종료될 때 좀비가 되는 것을 막는다.

## 프로세스의_생성

[i4]: #프로세스의_생성

프로세스의 생성목적

1. 같은 프로그램의 처리를 여러 프로세스가 나눠서 처리, (웹서버 리퀘스트 분산처리)
2. 전혀 다른 프로그램을 생성, (Bash로 부터 새로운 프로그램 init으로서 생성)

> 1: `fork()` 2: `execve()`

* `fork()`의 양상

1. 자식 프로세스의 메모리 영역을 미리 할당(init에서 부터 발행하는 시점에 메모리)
2. `fork()`함수의 리턴값이 각기 다른 것을 이용하여, 부모와 자식을 다른 코드를 실행하도록 분기. (1개의 스크립트 -> 2개의 실행)
3. 부모 프로세스는 `fork()`로 부터 복귀, 자식 프로세스는 `fork()`로부터 리턴되는 것

```c fork.c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <err.h>

static void child()
{
    printf("I'm child process, pid is %d", getpid());
    exit(EXIT_SUCCESS)
}

static void parent(pid_t pid_c)
{
    printf("I'm parent process, pid is %d, child pid is %d", getpid(), pid_c);
    exit(EXIT_SUCCESS);
}

int main(void)
{
    pid_t ret;
    ret = fork();
    if (ret == -1)
        err(EXIT_FAILURE, "fork() failed");
    if (ret == 0) {
        // child process come here
        child();
    } else {
        // parent process come here returns if created child process (> 1)
        parent(ret)
    }
    // shouldnt come here
    err(EXIT_FAILURE, "shouldn't reach here");
}
```

```bash
$ cc -o fork fork.c
$ ./fork
"i'm parent process, pid is 4193 child pid is 4194"
"i'm child process, pid is 4194"
$
```

* `execve()`의 양상

실행 순서

1. 실행파일을 읽은 다음 프로세스의 메모리 맵에 필요한 정보를 읽어 들입니다.
2. 현재 프로세스의 메모리를 새로운 프로세스의 데이터로 덮어 씁니다.
3. 새로운 프로세스의 첫 번째 명령부터 실행합니다.

기존의 프로세스를 별도의 프로세스로 덮어씌우면서 전환하는 방식.

프로세스를 실행하는데 필요한 디테일

* 실행파일의 inode를 통해 코드를 포함한 HDD영역의 메모리맵 시작주소, 오프셋, 사이즈
* 코드 외의 변수등에서의 데이터 영역에 대한 같은 정보(오프셋, 사이즈, 메모리맵 시작주소),초기 실행필요한 메모리
* 최초 실행할 명력의 메모리 주소(entry point)

프로그램의 실행파일 구조
| 이름                |   값   |
| :---------------- | :---: |
| 코드영역 파일상 오프셋      |  100  |
| 코드 영역 사이즈         |  100  |
| 코드영역 메모리 시작 주소    |  300  |
| 데이터 영역 파일상 오프셋    |  200  |
| 데이터 영역 사이즈        |  200  |
| 데이터 영역 메모리맵 시작 주소 |  400  |
| 엔트리 포인트           |  400  |

```c
// add.c
c = a + b
```

```
# compile to assembly
load m100 r0   # 읽어들여라 / m100 메모리 주소(name a)의 값을 / r0 레지스터에
load m200 r1   # 읽어들여라 / m200 메모리 주소(name b)의 값을 / r1 레지스터에
add r0 r1 r2   # 더해라 / r0 과 r1을 / r2 레지스터에 
store r2 m300  # 저장하라 / r2를 / m300 메모리 주소(name c)에
```

## 실행_포멧_ELF_분석

[i5]: #실행_포멧_elf_분석

리눅스 대표 실행파일인 ELF포멧의 정보는 `readelf`명령어로 해석 가능.  
`/bin/sleep`을 예제로 정보를 가져오면,

```bash
readelf -h /bin/sleep
ELF Header:
#   Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
#   Class:                             ELF64
#   Data:                              2's complement, little endian
#   Version:                           1 (current)
#   OS/ABI:                            UNIX - System V
#   ABI Version:                       0
#   Type:                              DYN (Shared object file)
#   Machine:                           Advanced Micro Devices X86-64
#   Version:                           0x1
#   Entry point address:               0x1b70
#   Start of program headers:          64 (bytes into file)
#   Start of section headers:          33208 (bytes into file)
#   Flags:                             0x0
#   Size of this header:               64 (bytes)
#   Size of program headers:           56 (bytes)
#   Number of program headers:         9
#   Size of section headers:           64 (bytes)
#   Number of section headers:         28
#   Section header string table index: 27
```

`Entry point address:               0x1b70`

1. 실행해야 하는 코드 주소 위치는 `0x1b70`
2. 여기서 코드영역과 데이터 영역의 파일상의 오프셋, 사이즈, 메모리맵 시작주소를 얻으려면

```bash
readelf -S /bin/sleep
There are 28 section headers, starting at offset 0x81b8:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
    #    중략
  [14] .text             PROGBITS         00000000000018d0  000018d0
       0000000000003989  0000000000000000  AX       0     0     16
  [24] .data             PROGBITS         0000000000208000  00008000
       0000000000000080  0000000000000000  WA       0     0     32
  [25] .bss              NOBITS           0000000000208080  00008080
       00000000000001c0  0000000000000000  WA       0     0     32
  # 중략
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```
