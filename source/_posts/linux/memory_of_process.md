---
title: basic concept of linux / about memory
p: linux/memory_of_process
date: 2019-08-16 10:09:52
tags: ["linux"]
---

## Index

1. BIOS, UEFI등(메인보드 임베디드 소프트웨어)으로 인해 OS가 메모리 레벨(커널메모리)로 이동. (프로세스지만 일반프로세와는 다른취급)
2. OS가 구동, 커널(실행자)과 사용자영역(프로세스)을 관리함.
3. Hdd에서 ELF같은 실행파일을 읽어오면서, 코드영역블록을 `TEXT세그먼트-메모리`로 할당함과 동시에 커널 메모리에 프로세스 PID 매핑.(선점형 스케쥴링-OS가 강력하게 관제)
4. 커널 내부에 보관되어 있는 페이지 테이블을 사용하여, 프로세스에 할당할 물리주소를 가상주소로 변환하여, 페이지 테이블을 마찬가지로 제공
5. 페이지 테이블(프로세스의)에서 한페이지에 대한 데이터를 `페이지 테이블 엔트리`라 칭함. 가상주소와 물리주소 대응정보.
6. 프로세스의 가상 메모리 페이지에 최초 실행과 동시에 매핑되는 것은 아래와 같다.
   * 커널 메모리주소
   * 프로세스 코드영역 데이터(실행부, 프로세스의 실행으로서의 정체)
   * 프로세스 데이터영역 데이터(초기 실행에 필요한 자원, C라이브러리 등 라이브러리 오브젝트 파일)

___

1. [**프로세스 주소 공간**][i1]
1. [**메모리 영역**][i2]

___

## 프로세스_주소_공간

[i1]: #프로세스_주소_공간

프로세스에서 직접 물리메모리 주소에 접근하지 않고,
커널이 개별프로세스에 독자적인 가상 주소 테이블을 제공한다.  
(이 주소 공간은 0에서 시작 연속적으로 늘어난다.)

### 페이지와_페이징

워드는 바이트로 구성되고, 페이지는 워드로 구성된다.
페이지는 `메모리 관리 유닛(MMU)`에서 관리할 수 있는 최소 단위다.

페이지 크기는 아키텍쳐에서 결정

* 32비트 시스템: 4KB
* 64비트 시스템: 8KB

1. 프로세스가 `mmap()`을 통해 메모리 풀(8KB)을 단위로 확보 하는 것을 **가상메모리를 확보 했음**이라 하며 페이지 단위 메모리 확보,(프로세스당 4gb정도씩 가상주소 선 할당)
2. 최초로 해당 페이지들에 `malloc()`(바이트 단위 메모리 확보)을 통해서 데이터를 매핑하려 시도하면, 최초에 물리메모리 주소가 매핑되어 있지 않기 때문에, `세그먼테이션 폴트`를 일으킨다.
3. 그러면 커널메모리의 MMU에서 해당 가상주소를 물리주소로 변환하고, 데이터를 실제로 메모리에 저장한다.
4. 어떤 변수가 사용된지 오래되었고, 다른 프로세스를 활성하다가 해당 변수를 이 별도 프로세스에서 `페이징 아웃` 하였을 경우.
5. 다시 해당 변수에 접근하면 해당 페이지의 매핑된 실제 주소가 HDD레벨로 내려가있을 것이고 그것을 `페이지 폴트`라 한다.
6. 페이지 폴트가 났다는 건 다시 메모리 레벨로 페이지를 끌어올리겠다는 것이기 떄문에, 어딘가 쓸모없다고 판단되는 페이지를 커널이 `free()`한다.

> 커널은 페이징에 따른 성능 부하(HDD접근수 증가에 따른)를 줄이기 위해, 가까운 미래에 덜 쓰일 것으로 예상되는 데이터를 페이징 아웃한다.

### 공유와_copy-on-write

* 공유 메모리란?  
가상 메모리에 존재하는 여러 페이지는 프로세스별로 유일하지만,  
여러 가상 페이지의 엔드포인트가 특정 물리 페이지로 매핑될 수 있다.  
이런 방식으로 물리 메모리에 있는 데이터를 여러 프로세스에서 공유한다.

* * 예를 들어 여러 프로세스가 표준 C라이브러리를 사용할 때,
각 프로세스는 표준 C라이브러리를 저마다의 가상 주소공간으로 매핑
실제 메모리에는 HDD에서 복사해서 올라온 하나의 페이지만이 존재한다.

* 공유 데이터를 특정 프로세스에서 수정하였을 때,  
공유 데이터는 읽기전용, 쓰기전용, 혹은 모두 가능한 형태로 존재.  
쓰기가 가능한 공유 데이터를 어떤 프로세스에서 OVERWRITE하였을 때,  
   1. 프로세스간 일정 수준의 조정과 동기화가 필요하지만, 반영된 페이지를 공유하거나,
   2. `COW방식`, MMU가 쓰기 요청을 가로체서 예외를 던진다.  
   그러면 커널은 쓰기 요청한 프로세스를 위해 해당 페이지의 복사본을 생성,  
   새로 만들어진 페이지에 대해 쓰기요청을 계속 진행

실제로 실 메모리 공간을 절약하기 위해, 여러 프로세스는 공유된 페이지에 대해서 읽기작업을 수행한다.

## 메모리_영역

[i2]: #메모리_영역

커널은 접근 권한과 같은 특정 속성을 공유하는 블록 내부에, 페이지를 배열한다. 이런 블록을 맵핑, 메모리 영역이라고 한다.  
모든 프로세스에는 다음과 같은 메모리 영역이 존재한다.

|segement|types|adds|
|:-|:-:|:-|
|Text|프로세스의 코드영역, 문자열 상수, 상수변수, 읽기전용 데이터|리눅스에서 이 세그먼트는 읽기전용. 실행파일과 라이브러리 오브젝트파일에서 직접 맵핑
|Stack|프로세스의 실행 스텍(지역변수, 함수의 반환데이터)|실행 스텍은 스택 깊이가 깊어지고 얕아짐에 따라, 동적으로 크기 변경된다. 멀티스레드의 경우 스레드당 하나의 스텍이 존재|
|Heap(DATA)|프로세스의 동적 메모리|이 세그먼트는 쓰기가 가능하고, 크기변경이 가능하다. `malloc()`은 이 영역을 할당한다.|
|BSS|초기화 되지 않은 전역변수|이 변수는 C표준에 따라(기본값 0) 특수한 값을 담고 있다.|

> 대부분의 주소공간은 매핑된 실행파일 혹은 C나 공유 라이브러리, 데이터파일을 포함.  
> `/proc/self/maps`파일을 열어 보거나 `pmap`명령어를 사용하여, 프로세스 맵핑파일을 살펴 볼 수 있다.