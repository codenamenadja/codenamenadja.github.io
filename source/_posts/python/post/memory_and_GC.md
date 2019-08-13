---
title: 파이썬의 GC와 메모리 관리자
p: python/post/memory_and_GC
date: 2019-06-19 18:22:23
tags: ['python', 'python post']
---
- ## reference counting -official Python doc
1. Py_INCEREF(), Py_DECREF()등을 통하여 레버런스 Count를 관리하고,
(파이썬 인터프리터와 메모리관리자가)
refcount가 최초로 0에서 1로 Init되면서, garbage collection의 Generation[0]부터 등록을 시작한다.

2. refcount에 따라서 자연스럽게 Deallocate memory를 시도하는데, 만약 Cyclic Garbage Collector에 최초에 참가한다는 타입이라고 초기화된 객체라면(PyObject_New()를 Exnted 하는 PyObject_init()으로 인한 선언이었을때,
: 추가적으로 타입등의 정보를 추가하여 객체의 Fields를 더해서 선언하는,

3. detector's Set of observed Objects에 추가된다. 이것이 객체 생성의 최종 마무리 이며 다른 필드에 대한 영향은 없다. 함수 실행과 종료와함께 Deallocate되는 객체에 대해서는 따로 GC에 대한 등록없이 스텍과 함께 사라지도록 구현되어있을것이다.

4. 그러나 순환참조가 일어나는 객체라면 초기화 당시, 타입과 함께 Python_init()으로 선언될 때 Garbage collection의 어느 부분에 dictionary의 형태로 관리된다.
****
- ## GC의 라이프사이클에 대해
gc.get_thredhold()를 통하여 리턴되는 것은 generation[0]~ generation[2] 까지 수용할 수 있는 총 객체의 수다.
1. 최초 객체 선언시 Refcount도 올라가지만, Generation[0]에 등록함으로써 
2. generation에 바인딩된 이벤트가 특정 수치에 도달하였을 때 
   1. generation[0].count=0
   2. gen[1].count++,
3. 그리고 gen[0]에서 아직 레퍼런스가 남아있거나, cyclic garbage collector의 set of observed object에 등록되어 있고 아직 유효하다면
   1. 0세대에 등록되어 있는 객체들은 1세대로 등록된다.
   
4. 메모리 해제를 위한 검사가 실행되면 gen[2]부터 gen[0]의 순서로 검사가 실행된다.
****
- ## 순환참조가 일어나지 않는 객체의 메모리 해제
기본적으로 객체는 Refcount라는 개념이 내장되어있다, 그 객체가 Gc의 generation에 등록되어 있지 않다고 한다면, 자체적으로 Ref가 0이 되는 스스로의 allocation을 해제한다.
****
- ## 인스타그램은 GC를 사용하지 않기로 했다고한다.
1. 인스타그램의 서버는 순환참조를 일으키는 객체의 생성을 최대한 막고 함수지향적인 코드를 작성하여 서버를 운영해 왔다고 한다.
2. 그러나 GC가 늘 onloadState이라는 점이나 일부 순환참조가 일어나는 객체들 등에 의해서, cache miss가 적지 않다고 느꼈다.
3. 그래서 파이썬 오피셜이 제안했던 것처럼 gc.disable()을 통해 해제 했으나,
   - 서드파티에서 gc.enable()을 호출하는 등 결국 효과를 보지 못했기 때문에,
  
4. threshold의 제한수를 없애주어서 Gc가 수행되는 시점을 사라지게 했다.\
   - 수행되는 시점에 그 모든 객체에 대해서 순환참조 탐지 알고리즘을 적용하는 것이 기존의 Cache를 가득 메우는 점이 맘에 안들었던 것이다.\
   그 양이 적지 않아 워낙에 대규모 서비스 이다보니, 다른일로 돌아가야 하는데도 가상메모리상에 pageFault가 일어났다고 한다.

- ## 결과
1. 개별 서버에 8gb 정도의 메모리사용량을 비울 수 있다고 했다.
   > 이는 더 많은 메모리 소모를 필요로 하는 서버를 생성할 수 있다는 얘기이다.

2. CPU의 IPC(Cycle 당 instruction 수)가 10% 가까이 증가하였다.
    >cache miss가 2~3%가량 이득을 보았던 점이 IPC성능의 관건있다고 한다.
3. cache miss는 큰 자원소모이다. cache miss -> CPU Pipeline에 Stall발생
   > youtube(https://www.youtube.com/watch?v=twQKAoq2OPE)에 등장하는 것처럼 최대한 많은 공유 라이브러리를 공유 메모리로서 캐쉬안에 보관 될 수 있어서, 많은 page들이 캐싱상태에 존재하고이는 Stall을(CPU와 캐시사이의 새로운 allocation등을 위한 Fetch를 포함한 결과까지) 최대한 막아주기 때문에 Cpu가 빠르게 동작하도록 한다.
****

OS는 프로세스들이 (코드 혹은 데이터 세그먼트)공유를 통해 최적화를 하기 원하지만, 힙 안에서, 개별적인 세그먼트를 통해 독립적인 네임스페이스를 형성하기 때문에, reference-counted code objects는 이 이점을 가질 수 없다.

> 1---> 36-- r libc.so (C 기반-라이브러리를 읽어와서)
>
> 0---> 35-- r python.exe (파이썬을 플랫폼으로)

프레임 워크가 기인하는 라이브러리(C등으로 구성된,)는 stdio를 임포트하는 라인이 있더라도,\
해당라인은 명령으로서 미리 힙의 크기만 빈 크기만 할당하고, 실제로 필요해졌을때, 해당요소를 HDD레벨에서 로드하여 가져온다. 

동등한 프로세스의 경우 PID가 달라서 프로그램은 그들의 코드레벨 혹은 Data 레벨이 어떤 동일한 데이터를 공유할 수 있는지 알수 없다.

ChildProcess개념에서 childprocess.spawn() as fork까지 개념이 늘어가면, 기존의 프로세스가 spawn한 child process가 자신의 부모프로세스에 대해 코드레벨에서 동일한 프레임을 참조하는 페이지 공유환경이 성립된다. 스텍이 구현된 프레임은 자식프로세스가 새로운 콜스텍 메모리 w해야 하기 때문에 다른 페이지로 매핑된다.