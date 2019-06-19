---
title: 파이썬의 가상 메모리
p: python/pure_resouce/virtual_memory_with_malloc
date: 2019-06-19 18:05:23
tags: ['python', 'memory']
---

* ### Heap을 구현한(및 연결점인) C의 malloc()이 느리기 때문에, HEAP이 느리다
Heap 의 자료구조는 헤더를 통해 다음 bytesArray를 연결하는 double Link list
1. 처음에는 단순하게 스텍처럼 쌓이다가 일부 메모리가 해제되어,
2. 새로운 크기의 메모리를 할당하려고 할때,
3. 데이터가 사라진 남아있는 헤더들이 유지하고 있는 사이즈와 맞는 요소를 찾아 linked List를 순회하며 찾는다. 

이로 인해 생기는 현상을,
**Memory Fragmentation(메모리 단편화)** 라고 한다.

Heap에 배치된 총 메모리에 비해 남은 메모리가 있어도,\
맞는 크기가 없으면, Scaleup 하는 것 외에는 방법이 없다.

파이썬등 인터프리터언어들의 엔진은 라인별로 실행하는 특성상 대부분 동적할당 혹은 부분적으로에 의해 일이 잦은데,\
프로세스를 실행하는 때에, Heap의 정크메모리(많은 양의)를 할당 받아, 그 커다란 메모리를 엔진이 직접 관리하면서,
- CPU-Memory-HeapSegement
    1. python requires Junk Heap Memeory space.
    2. python call it **Private Heap**.
    - python.exe's private heap
      - Data Segment
      - Code Segment
      - Stack Segment
      - Heap Segment

역할을 나눠 직접 모니터링 하면서 관리한다. 그래서 c의 malloc()을 이어받아, 기능적으로 힙을 관리하는 자체 메서드를 생성했다.
***

## virtual memory
1. OS: when Process starts?
2. give mapped page-Table to Process
> sytex of pageElem ({page#num:[] , frame#num:[], valid bit:[]})
3. 4gb정도의 크기를 가상 메모리로 할당하고, 실제로 프로레스를 시작하는데 필요한 만큼은 바로 Page Fault를 일으킨 후 실제 메모리와 매칭된다.
4. pageFault(validbit is 0) -> map to real ram addr -> valid bit up to 1

preparing: 첫 주소도 알려주고 4기가 라고 알려주지만, 실제 바로 필요한 메모리 만큼은 실제로 연결 해준다.

pc가 다음 명령을 검토 -> 페이지 테이블 첫 주소로 이동 -> 매핑된 프레임주소 -> 실제 메모리에 저장 -> vaild bit = 1(실제 할당됨)
매핑되는 프레임주소(MMU 하드웨어로서 페이지를 프레임으로 변환해줌)

관건은 코드세그먼트에서 알고있는 메모리주소로 이동하면 그 메모리 주소는 페이지 테이블에 실제 메모리 주소가 매핑되어 있다는 것.
Logical메모리인 페이지 테이블에 Validbit가 -이면 실제 매핑된 메모리주소에 없다는 것임.

최초 프로그램 실행(가짜메모리인 테이블만 있다) -> valid bit 가 0이다.(읽은 적이 없으니까 최초시작이니까) -> 하드디스크 -> 실제주소로 가져오고 저장 -> 페이지테이블로 돌아온다 -> Valid bit =1 .-> 프로그램 카운터가 해당 실제 지침을 가지고 돌아.. -> 지침레지스터에 전달

프로그램에서 이전에 실행했던 컴포넌트는 이미 메모리에 있어서 최초 프로그램내의 컴포넌트를 실행할 때보다 빠르게 반응하는 것에 대한 실재.