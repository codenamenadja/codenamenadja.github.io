---
title: pointer in python 번역
p: python/post/pointer_in_python
date: 2019-08-30 20:21:27
tags: ["python", "python post"]
---

# [Pointer in python: What's the point](https://realpython.com/pointers-in-python/)

> 번역, 정리글입니다.

## index

1. [**Why Doesn't Python have Pointers?**][i1]
2. [**Objects in Python**][i2]
3. [**Immutables vs Mutable objects**][i3]
4. [**Understading Variables**][i4]
   1. [**Variables in C**]
   2. [**Names in Python**]
   3. [**A Note on Intern Objects in Python**]
5. [**Simulating Pointers in Python**]
   1. [**Using Mutable Types as Pointers**]
   2. [**Using Python Objects**]
6. [**Real Pointers With ctypes**]
7. [**Conclusion**]

## 왜_파이썬에는_포인터가_없는가

[i1]: #왜_파이썬에는_포인터가_없는가

포인터가 파이썬에서 네이티브하게 존재할 수 는 없을까? 잘은 모르지만 아마 가능할 것이다.  
그러나 파이썬의 포인터는 **Zen of Python**에 위배되는 것처럼 보인다.

> **Zen of python** / Cpython의 구현의 주요 기여자인 Tim peters의 20개 명언
>
> Beautiful is better than ugly.  
> Explicit is better than implicit.  
> Simple is better than complex.  
> Complex is better than complicated.  
> Flat is better than nested.  
> Sparse is better than dense.  
> Readability counts.  
> Special cases aren't special enough to break the rules.  
> Although practicality beats purity.  
> Errors should never pass silently.  
> Unless explicitly silenced.  
> In the face of ambiguity, refuse the temptation to guess.  
> There should be one-- and preferably only one --obvious way to do it.  
> Although that way may not be obvious at first unless you're Dutch.  
> Now is better than never.  
> Although never is often better than *right* now.  
> If the implementation is hard to explain, it's a bad idea.  
> If the implementation is easy to explain, it may be a good idea.  
> Namespaces are one honking great idea -- let's do more of those!

포인터는 명시적인 변화보다 내재적인 변화를 촉구한다.  
때로, 그들은 단순하기보다 복잡하고(특히 초보자들에게,) 더욱 나쁜것은,  
그들은 당신이 직접 하기를 원하고, 또는 메모리에 접근하기 때문에 매우 위험할 수 있다.

파이썬은 메모리 주소 같은 디테일의 구현을 추상화하려한다.  
파이썬은 때로 속도보다 사용감에 집중한다.  
결과적으로 파이썬에서 포인터는 말이 되지 않는다.  
그러나 파이썬은 결과적으로 포인터를 사용하는데 있어서의 이점을 준다.

파이썬의 포인터를 이해하는 것은 파이썬 구현부의 디테일을 돌아볼 필요가 있다.

1. Immutable vs Mutable objects
2. Python variables/names
를 이해할 필요가 있다.

## Object_in_Python

[i2]: #object_in_python

파이썬의 모든 자료구조는 기본 object를 상속받는 인스턴스 클래스들이다.  
각 객체는 3개의 데이터를 최소한 포함한다.

* Reference count
* Type
* Value
레퍼런스 카운트는 메모리 관리를 위한 것이고, 파이썬의 메모리 관리를 이해하고 싶다면,  
[Memory Management in Python](https://realpython.com/python-memory-management/)를 읽어보면 좋다.

___
> **Python object**
>
> `PyObject`라고 불리는 구조체가 있다. 모든 Cpython의 객체가 사용하는 것이다.
> `PyObject`는 모든 객체의 부모이며, 아래의 2요소를 포함한다.
>
> * `ob_refcnt`: 레퍼런스 카운트
> * `ob_type`: 어떤 타입에 관한 포인터
>
> 각 객체는 그들 각자의 object-specific memory allocator을 가지고 있다.
> 그것은 그들 객체를 위한 메모리를 가져온다.
> 각 객체는 또한 object-specific memory deallocator을 가지고 있다.
> 그것은 그들의 객체들이 refcount에 따라 메모리를 `free`한다.
>
> 그러나 메모리를 할당하고 해제하는 것에 있어서, 중요한 요소가 있다.
> 메모리는 컴퓨터의 공유자원이며, 두 개의 프로세스가 동시 접근하려고 하면 문제가 생긴다는 것이다.
>
> GIL은 공유자원 Race Condition에 대한 해결책중 하나이다.
> ___
> >
> > [**GIL**](https://realpython.com/python-gil/)
> >
> > GIL은 짧게 말해서 mutex이며, 하나의 쓰래드 만이 파이썬 인터프리터를 제어할 수 있도록 한다.
> >
> > 이 말은, 하나의 쓰레드만이 실행부의 상태에 있을 수 있다는 것이다.
> > GIL의 부하는 싱글쓰레드 프로그램을 하는 사람들에게는 나타나지 않는다.
> > 이것은 CPU-bound한 문제, 멀티쓰레딩 코드 에서 bottleneck이 될 수 있는 것이다.
> >
> > **GIL이 파이썬의 어떤 문제를 해결해 주는가?**
> >
> > 파이썬은 메모리 관리를 위해 refcount를 사용한다. 그것은 파이썬에서 생성된 objects 구조체가 refcount변수를 가지고 있고 추적되고 있다는 것을 의미한다.
> > 이것이 0이 되었을 때, 해당 객체가 점유하는 메모리는 해제된다.
> >
> > ```python
> > import sys
> > a = []
> > b = a
> > sys.getrefcount(a)
> > #3
> > ```
> >
> > a에 처음 할당된 메모리 주소에 대해서 a,b가 참조하고, sys.getrefcount에 전달된 argument가 순간적으로 참조하고 해제된다. 그래서 3이 나온다.
> > GIL로 돌아가서,
> >
> > 문제는 Refcount변수가 race-condition에서 보호받아야 한다는 것이다.
> > 만약 2개의 쓰레드가 동시에 그 수치를 늘리거나 줄이면,
> > 메모리 누수를 초래해서, 절때 해제되지 않거나, 원치 않게 해제된다는 것이다.
> >
> > 이 refcount변수는 locks를 더함으로 안전하게 보관되어,
> > 쓰레드에 의해 공유되는 모든 자료구조가 일관적이지 않게 수정되는 것을 막는다.
> > 하지만 각 객체에 lock을 더하는 것은 너무 많은 lock이 존재하여 다른 문제를 발생 시킬 수 있다는 것을 의미한다.
> >
> > `Deadlocks` (데드락은 하나이상의 락이 있을 때 생길 수 있다.)
> > 특정 자원에 2쓰레드 혹은 프로세스가 순차적으로 접근하려고 하면, 두번째 쓰레드는 첫번째 쓰레드가 락을 해제할 때까지 끝없이 기다리게 된다.
> >
> > 다른 문제로는 락을 반복적으로 생성하고 해제하는 것으로 인한 퍼포먼스 저하가 있다.
> > GIL은 인터프리터에 대한 단일 lock을 의미하며,
> > GIL은 바이트코드를 실행하려면 인터프리터 잠금을 획득해야 한다는 규칙을 추가한다.
> > 이것이 deadlocks를 방지하고, 그렇게 많은 퍼포먼스를 요구하지 않도록 한다.(성능적 오버헤드를 방지)
> > 하지만 Cpu-bound한 파이썬 프로그램을 싱글쓰레드로 효과적으로 만들어 버린다.
> >
> > 하지만 Refcount와 GC를 이용한 GIL을 사용하지 않는 경우도 있다
> > 반면에 그것을 그러한 언어들이 때로 GIL이 효과적으로 싱글 쓰레드에 대해서 퍼포먼스 이점을 주는 것을
> > 다른 보상으로 채워야 한다는 것을 의마한다, 예를 들면 JIT컴파일러 같은 것을 통해서.
> > ___
> >
> > **왜 GIL이 해결책으로 선택 되었는가?**
> >
> > [**Larry Hastings**](https://www.youtube.com/watch?v=KVKufdTphKs&feature=youtu.be&t=12m11s)의 말에 따르면, GIL을 선택한 것은 파이썬을 오늘날 만큼 유명하게 만든 하나의 이유라고 한다.
> > 파이썬은 OS가 쓰레드의 개념이 없을때 생겨 났다.
> > 파이썬은 쉽게 사용될 수 있고 빠르게 개발할 수 있도록 만들어 졌다.
> >
> > 이후 파이썬의 일관적이지 않은 변화를 예방하기 위해서, 이 파이썬을 지탱하는 C라이브러리는,
> > Thread-safe 메모리 관리를 필요로 했고, 그래서 GIL이 생겨 났다.
> > GIL은 실행하기 쉽고 파이썬에 쉽게 더해졌다.
> > 이것은 싱글 쓰레드 상에서 하나의 락만이 관리되야한다는 점에 의해 퍼포먼스 향상을 가져왔다.
> > 쓰레드세이프 하지 않은 C라이브러리 들은 더욱 통합하기 쉬워졌다.
> > 그렇게 통합된 C라이브러리들이 파이썬이 여러 커뮤니티에서 손쉽게 적용될 수 있었던 이유이기도 하다.
> >
> > 니가 볼 수 있듯이, GIL은 Cpython개발자들이 초기 파이썬에서 복잡한 문제를 빠르게 해결할 수 있는 방법이었다.
> > ___
> >
> > **왜 GIL은 아직 사라지 않았는가?**
> >
> > GIL은 명백하게 사라질 수 있고, 그런 시도들이 많았으나, 그러한 시도들은 GIL이 주는 해결책에 의존하는 C-extensions를 망가뜨렸다.
> > 물론, GIL을 대체하는 해결책도 있지만, 하지만 그들의 일부는 싱글쓰레드, 멀티쓰레드/IO바운드 프로그램에 성능을 떨어뜨렸고,
> > 일부는 너무 어려웠다.
> > 어찌됐든, 지금 존재하는 파이썬 프로그램이 새로운 버전이 나옴으로 더 느려지는 바라는 사람은 없지 않은가?
> >
> > 귀도 반 로썸이 [It isn't Easy to Remove the GIL](https://www.artima.com/weblogs/viewpost.jsp?thread=214235)의 포스트로 2007년에 해명을 했다.
> > > "나는 Py3k에 대한 패치들을 환영할 것이다. 만약! **퍼포먼스가 떨어지지 않는다면.**"
> ___
>
> **Garbage Collection**
>
> 파이썬의 모든 구조체에는 Refcount라는 것이 있고 아래와 같은 경우들에 증가한다.
>
> ```python
> numbers = [1,2,3]
> more_numbers = numbers
> # refcount = 2
> total = sum(numbers) # refcount = 3
> # refcount = 2
> matrix = [numbers, numbers]
> # refcount = 4
> ```
>
> 파이썬은 현재 오브젝트의 러퍼런스 카운트를 sys모듈을 통해서 확인 할 수 있게 해준다. `sys.getrefcount(object)`
> 그렇지만 deallocate기능을 사용하게 되면, 어떻게 메모리가 `free`되는가? 어떻게 객체가 그 과정을 거치는가?
> ___
>
> **Cpython의 메모리 관리자**
>
> 위에서 말했듯, 하드웨어에서 Cpython으로 이어지는 추상화 계층이 있다.
> OS는 실제 물리메모리를 추상화 하고 가상메모리 계층을 만들어 어플리케이션이 접촉할 수 있도록 한다.
>
> OS에 따른 메모리 관리자들은 파이썬 프로세스를 위한 메모리 청크를 조각해낸다.
> 파이썬은 일단 운영체제에 malloc함수로 메모리할당을 요구한 다는 것을 알아야한다.
>
> 파이썬 프로세스는 내부적인 사용을 위한 메모리공간과 비객체적인 메모리를 위한 공간을 사용한다.
> 마지막 공간은 object storage를 위해 사용된다.(`int, dict, and elses`)
> 메모리 관리의 구현부인 [Cpython source](https://github.com/python/cpython/blob/7d6ddb96b34b94c1cbdf95baa94492c48426404e/Objects/obmalloc.c)를 참고해도 좋다.
>
> 객체 메모리 공간에 파이썬은 object allocator을 두고 있다. 그것이 대부분의 기능이 이루어 지는 곳이다.
> 흔한 경우로, `list`나 `int`같은 파이썬 객체를 생성하거나 삭제하는 것은 그렇게 많은 데이터를 필요로 하지 않는다.
> 따라서, 메모리 할당자는 작은 양의 데이터 많으로 잘 작동하도록 되어 있다.
> 그리고 그것은 실제로 그 객체가 필요해지기 전까지 메모리에 할당하지 않으려고 한다.
>
> 위에 `Cpython source`구현부에서는, 객체 할당자를
> "작은 블록을 위한 빠르고, 특별한 용도를 위한 메모리 할당자이다. 그리고 일반적인 목적의 `malloc`위에서 실행된다."
> 라고 설명한다.
>
> 이제 Cpython의 메모리 할당 전략에 대해 살펴볼 것 이다.
> `Arenas`는 페이지 범위내에서 가장 크고 정렬된 단위이다.
> 페이지 영역은 고정된 길이의 연속적인 메모리 청크라고 할 수 있다.(실제 주소가 달라도, 가상주소상으론 연속적인 주소 그리고 실제주소는 랜덤 엑세스가 가능하기 때문에,)
> 파이썬은 시스템의 페이지 사이즈를 256kb정도 라고 가정한다.(내 컴퓨터 페이지 사이즈는 8kb이다;;)
>
> `Arenas`내부에는 `pools`가 있고, 그것은 하나의 가상 메모리 페이지이다.
> 그리고 풀은 실제로 동적 할당으로서 작은 메모리 사이즈로 조각난다.
> 풀 내부에 모든 블록은 동일한 `size class`이다.
> `size class`는 특정 블록 사이즈를 정의하고, 요청한 정보가 들어간다.
> 아래 차트는 소스코드에서 직접 가져온 커멘트이다.
>
> |Request in bytes|Size of allocated block|Size class Index|
> |:-:|:-:|:-:|
> |1-8|8|0|
> |9-16|16|1|
> |17-24|24|2|
> |25-32|32|3|
> |...|..|.|
> |505-512|512|63|
>
> 예를 들어 42바이트의 데이터가 요청되었다면, 데이터는 48-byte블록까지 차지하게 될 것이다.
> ___
>
> **Pools**
>
> Pools는 단일 size class로부터 조합된 블록들로 구성된다.
> 각 풀은 double-linked list로 같은 size class로 구성된 다른 풀들로 연결되어 있다. 그런 방식에서 볼 때,
> 더블링크드리스트를 통한 알고리즘은 쉽게 블록사이즈 만큼 요구된 공간을 다른 풀들 사이에서 찾을 수 있는 것이다.
>
> 사용된 Pool들의 리스트는 할당가능한 블록들이 남아있는 모든 풀들을 추적한다.
> 어떤 블록 사이즈가 요구되면, 알고리즘이 `usedpools`리스트를 확인하여, 블록사이즈가 할당 가능한 pool list를 반환한다.
> Pool들은 각자 3가지 상태가 존재한다.
>
> * used: 저장가능한 `avaliable blocks`를 지님.
> * full: 모든 블록들에 데이터가 할당되었음.
> * empty: 아무런 데이터가 없고, 언제든 어떤 사이즈의 블록들도 할당 할 수 있다.
>
> `freepools`리스트가 비어있는 상태의 풀들의 리스트를 추적한다. 그러면 언제 비어있는 풀은 사용되는 가?
> 네가 8byte의 청크 메모리를 할당해야 한다고 치자.
> 만약 거기에 8바이트를 할당 할 수 있는 `usedpools`가 없다면, 새로운 `empty pool`애 8바이트가 초기화 될 것이다.
> 이 새로운 `pool`은 `usedpools`리스트로 더해지고, 이것은 이후에 올 요청의 첫 대상이 된다.
> 마찬가지로 `full`상태인 `pool`이 일부 블록을 해제한다면, `usedpools`리스트로 옮겨진다.
> ___
>
> **Blocks**
>
> 아까 블록에 실제로 데이터가 할당 되는 것은 해당 블록의 데이터가 실제로 필요해질 떄 그럴 것이라고 했었다.
> 그러한 이유로 blocks도 3가지 상태를 가진다.
>
> * untoched: 아직 할당 되지 않은 메모리블록.
> * free: 데이터가 할당된 블록이나, Cpython에 의해서 `free`되어서 현재 유효한 데이터가 들어있지 않은 블록.
> * allocatd: 실제로 데이터를 가지고 있는 블록.
>
> `freeblock` 포인터는 single linked list인 비어있는 메모리 블록의 list를 가르킨다.
> 더 많은 길이의 블록이 필요하면, 메모리 할당자는 해당 풀 안에 `untouched`블록을 더해서 돌려줄 것이다.
> 할당되어 있던 블록(allocated)가 해제되어 free상태가 되면,
> single linked list인 Free block list의 head로 오게 된다.
> ___
>
> **Arenas**
>
> 아레나들은 pools를 포함한다. pools와 blocks는 3가지 상태가 있었던 반면, 아레나들은 명시적인 상태가 없다.
> 아레나들은 반면에 `usable_arenas`라 불리는 Double linked list로 구성 및 조직된다.
> 이 리스트는 freepools를 얼마 가지고 있냐에 따라 정렬된다.
> freepools가 적을수록 해당 아레나는 리스트의 head쪽으로 오게 된다.
>
> 그런데 왜 가장 사용가능한 데이터가 많은 arena가 아니라 그 반대인가? 그곳에 메모리해제의 개념이 있다.
> block이 free하게 전환되면, 그 메모리는 실제로 OS에게로 돌아가는 게 아니라,
> 파이썬 프로세스가 그 공간을 계속 점유하고, 다음 데이터를 위해 사용할 것이다.
> 실제로 메모리를 해제한다면 OS에게 돌려주는 것을 의미한다.
>
> 아레나야 말로 유일하게 정말 해제될 수 있는 메모리이다.
> 따라서, 가장 많이 비어있는 데이터가 가장 쉽게 empty arena가 될 수 있도록 그러한 것이다.
> 그 방식에서 메모리의 청크는 실제로 해제되고, 파이썬 프로세스의 점유나 감시에서 벗어날 수 있도록 돕는 것이다.

type은 Cpython계층에서 타입안전성을 런타임에서 보장할 때 사용된다.
마지막으로 value가 실제로 object와 관련된 페이로드를 말한다.

모든 객체가 동일한 객체는 아니나, 그런 단순한 사실 외에도 당신이 꼭 알아야할 더 중요할 구분이 있다.
**mutable vs immutable**객체 이다. 이 두 타입의 객체의 차이를 이해하는 것은, 파이썬의 포인터의 초반부를 이해하는데 굉장히 도움이 된다.
___

## mutable_vs_immutable

[i3]: #mutable_vs_immutable

|   type    | immutable? |
| :-------: | :--------: |
|    int    |    Yes     |
|   float   |    Yes     |
|   bool    |    Yes     |
|  complex  |    Yes     |
|   tuple   |    Yes     |
| frozenset |    Yes     |
|    str    |    Yes     |
|   list    |     No     |
|    set    |     No     |
|   dict    |     No     |

위에 보이듯 대부분 사용되는 원시타입들은 immutable하다. 아래 기능을 사용하여 그것을 검증할 수 있다.

1. `id()`: object's memory address를 반환
2. `is`: 두 개체가 동일한 메모리 주소를 가지고 있다면 True를 반환.

## Understanding_Variables

[i4]: #understanding_variables