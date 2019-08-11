---
title: 커널의 이해와 시스템 콜 이해
p: linux/understanding_kernel
date: 2019-08-11 17:35:49
tags: ["linux", "os"]
---

## Index

___

1. [**개요**][i1]
2. [**사용자 모드로 구현되는 기능**][i2]
___

## 개요

[i1]: #개요

### OS의 구성, 커널,사용자 모드

| 타입    |                  정의                  |
| :---- | :----------------------------------: |
| 커널모드  | OS의 인터페이스로 내부 장치에 대한 Wrapper manager |
| 사용자모드 |       OS의 인터페이스를 사용하는 외부 프로세스        |

### 커널프로세스의 고유한 역할

| title        |                            설명                            |
| :----------- | :------------------------------------------------------: |
| 디바이스 드라이버 관리 |  외부 프로세스가 디바이스에 대한 동시접근을 제한, 종류가 같은 디바이스는 동일 인터페이스로 조작   |
| 프로세스 관리 시스템  | 프로세스의 상태를 관리하고, 그에 따른 상태를 조정 및 관리 (OnProcessing, , kill) |
| 프로세스 스케줄링    |                  프로세스당 CPU 리소스 사용 시간 부여                  |
| 메모리 관리 시스템   |                프로세스당 메모리(물리메모리, 가상메모리) 부여                |

프로세스의 동작과 OS의 역할이라 함은, 다양한 데이터가 메모리를 중심으로

    1. CPU의 레지스터
    2. L1,L2 Cache
    3. 메모리
    4. 저장장치

와 같은 저장장치의 계층 구조를 통해 프로세스를 빠르고 안정적으로 동작시킨다.
저장장치에 보관된 데이터는 디바이스 드라이버에 요청하여 접근 할 수 있지만,
좀 더 간단히 접근하기 위해, **파일 시스템**이라하는 프로그램을 통해 접근한다.

    1. 프로세스(사용자 모드)
    2. 파일시스템(커널모드)
    3. 저장장치 드라이버
    4. 저장장치

**시스템 또한 최우선의 관리자로서 작동하려면 저장장치로 부터 OS를 읽어야 한다.**

> 정확하게는 OS를 읽어들이기 전에 **BIOS**, **UEFI**라 하는 하드웨어 임베디드 소프트웨어가 작동하여
>
> 1. 하드웨어의 초기와 처리
> 2. 동작할 OS를 선택하는 부트로더
>
> 가 동작합니다.
___

## 사용자_모드로_구현되는_기능

[i2]: #사용자_모드로_구현되는_기능

- 다룰 내용
    1. 시스템 콜
    2. OS가 제공하는 라이브러리
    3. OS가 제공하는 프로그램

### 시스템 콜의 종류

    1. 프로세스 생성, 삭제
    2. 메모리 확보, 해제
    3. 프로세스간 통신(IPC)
    4. 네트워크
    5. 파일시스템 다루기
    6. 파일 다루기(디바이스 접근)

### CPU의 모드 변경

1. 프로세스가 사용자 모드로 사용되는 와중에 커널에 처리를 요청하고자 시스템 콜을
호출하면 **CPU인터럽트 발생**
2. CPU가 사용자 모드에서 커널모드로 변경
3. 요청내용 처리
4. 시스템 콜 처리 종료
5. 저장해놨던 CODE segment에 대한 정보를 이어가, IR에 담음
6. 사용자 모드로 전환 프로세스 동작 **이어감**

### 시스템 콜 호출의 동작 순서

    ```c hello.c
    #include <stdio.h>

    int main(void)
    {
        puts("hello world");
        return 0;
    }
    ```

    ```bash bash
    strace -o hello.log ./hello
    hello world
    ```

    ```bash bash
    cat hello.log
    execve("/hello", ["./hello"], [/* 77 vars */]) = 0
    brk(NULL)                               = 0x5619fe7aa000
    access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
    access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or 
    directory)
    ...
    close(3)                                = 0
    write(1, "hello world\n", 12)           = 12
    exit_group(0)                           = ?
    ```
___
    ```python hello.py
    print("hello world")
    ```

    ```bash bash
    strace -o hello.py.log python3 ./hello.py
    ```

    ```bash bash
    cat hello.py.log
    execve("/home/junehan/miniconda3/bin/python3", ["python3", "hello.py"], 0x7fff882f0768 /* 77 vars */) = 0
    brk(NULL)                               = 0x5619fe7aa000
    access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
    readlink("/proc/self/exe", "/home/junehan/miniconda3/bin/pyt"..., 4096) = 38
    access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or 
    directory)
    ...
    close(3)                                = 0
    write(1, "hello world\n", 12)           = 12 # write system call
    rt_sigaction(SIGINT, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x7f8b453db890}, {sa_handler=0x5619fd4904d0, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x7f8b453db890}, 8) = 0
    sigaltstack(NULL, {ss_sp=0x5619fe7ca720, ss_flags=0, ss_size=8192}) = 0
    sigaltstack({ss_sp=NULL, ss_flags=SS_DISABLE, ss_size=0}, NULL) = 0
    munmap(0x7f8b4564c000, 262144)          = 0
    brk(0x5619fe842000)                     = 0x5619fe842000
    exit_group(0)                           = ?
    # 705개의 시스템 콜 호출
    ```
