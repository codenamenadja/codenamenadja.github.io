---
title: 비동기 코루틴 번역-3
p: python/post/async_coroutine_3
date: 2019-06-19 18:11:02
tags: ['python', 'python post']
---

## 어떻게_제너레이터가_동작하는가

제너레이터 이전에 어떻게 파이썬 루틴들이 동작하는지 알아봐야할 것이다.
기본적으로 파이썬 함수가 서브루틴을 부를때, 서브루틴이 종료될 때까지, 컨트롤을 얻기 때문에, 블로킹이 일어난다.

```python
def foo():
    bar()

def bar():
    pass
```

파이썬의 인터프리터는 Cpython을 기반으로 작동한다.  
파이썬 함수를 실행하는 **C function**은 달콤하게도, **`PyEval_EvalFrameEx`**라고 불려진다.

**`PyEval_EvalFrameEx`**는 파이썬 스텍프레임 오브젝트를 수용해서, 프레임의 Context안에서, 파이썬의 바이트코드를 평가한다.  
여기에 foo에 대한 바이트코드가 있다.

```python
import dis
dis.dis(foo)
```

|      |                 |                                 |
| :--- | :-------------- | :------------------------------ |
| 2    | 0 LOAD_GLOBAL   | 0 (bar)                         |
|      | 3 CALL_FUNCTION | 0 (0 positional, 0 keywordpair) |
|      | 6 POP_TOP       |                                 |
|      | 7 LOAD_CONST    | 0 (None)                        |
|      | 10 RETURN_VALUE |                                 |

* dis - Disassembler for Python bytecode  
  해당 모듈은 CPython bytecode를 disassembling 함으로써 분석한다.

* CPython 실행 디테일: 바이트코드는 CPython의 실행부에 대한 명세이다.  
  2 0 load_global 0(len)  
  맨 앞에 있는 것은 라인넘버이다.

**foo** Function은 bar을 그의 스텍에 로드하고, 부르고 있다.  
그리고 **foo**의 return value(기본적으로 None)을 자신의 스텍에서 pop한다.  
**None**을 자신의 스텍에 로드하고, 마찬가지로 **None**을 리턴한다.

**Py_EvalFrameEx**가 **CALL_FUNCTION** 바이트코드를 만났을때,  
그것으로 새로운 파이썬 스텍프레임을 생성하고 recurse 한다.

파이썬의 스텍프레임이 기본적으로 힙에 존재한다는 사실을 아는 것은 매우 중요하다!  
기본 스텍이 조차도 OS의 메모리의 스텍세그먼트가 아니라 OS_heap > private heap에 명시적으로 stack-segment를 이루고 있다!  

그렇다면 명시적인 스텍세그먼트에 프레임으로 저장한다고 하면, 임의로 컨트롤 하기가 매우 쉬워지는 것이다.  
파이썬 인터프리터는 평범한 C프로그램이다. 그래서 스텍프레임 또한 일반적인 스텍프레임이다.  
그러나 파이썬 실행되는 파이썬의 스텍프레임은 힙에 있다. 다른 놀라움 사이에서,  
그것은 파이썬 스텍프레임이 그것을 존재하게한 function call stackframe보다 오래 살 수 있다는 것을 의미한다.  
이것을 상호작용적 측면에서 보기 위해서, 현재의 프레임을 bar의 내부에서 저장해라

```python
import inspect
frame = None

def foo():
    bar()

def bar():
    global frame
    frame = inspect.currentframe()

foo() # this frame was executing the code for 'bar'.

frame.f_code.co_name
# 'bar'

caller_frame = frame.f_back # its back pointer refers to the frame for 'foo'.

caller_frame.f_code.co_name
#'foo'
```

* `foo()`를 실행하면 순차적으로 벌어지는 일
  1. PyEval_EvalFrameEx in foo func Executes
  2. evaluate self.f_code
  3. made CodeByteCode of foo
  4. recursion occurs
     1. PyEval_EvalFrameEx in bar func Executes..
     2. eval self.f_code

* PyObject 설명
  * PyEval_FrameObject:
    * C구조의 객체, frameObject를 설명하기 위한 객체이다.  
      이 타입의 필드들은 언제든 바뀔 수 있도록 설계되어 있다.
  * PyEval_EvalFrame(PyFrameObject *f):
    new Reference, 실행부에 있는 프레임을 평가한다.  
    후위에 대한 호환성을 유지하기 위해 PyEval_EvalFrameEx의 축소화된 인터페이스를 갖고 있다.
  * PyEval_EvalFrameEx(PyFrameObject *f,int throwflag):  
    이것은 어떠한 가공도 덧붙여 지지 않은 파이썬 해석부(interpretation)의 메인 함수이다.  
    이것은 말 그대로 2000lines만큼 길고, 대상 바이트코드와 실행-Calls를 필요한 만큼 해석하며  
    코드오브젝트와 연관된 execution-프레임인 **f** 는 실행된다.

이제 놀라운 효과를 위해서 같은 building blocks(codeObj, stackObj)를 사용하는 무대는 파이썬 제너레이터를 위해 준비가 되었다.

이것이 제너레이터 함수이다.

```python
def gen_fn():
    result = yield 1
    print('result of yield: {}'.format(result))
    result2 = yield 2
    print('result of 2nd yield: {}'.format(result2))
    return 'done'

generator_bit = 1 << 5 # 00100000
bool(gen_fn.__code__.co_flags & generator_bit)
True
```

파이썬이 gen_fn을 바이트코드로 변환할 때, 그것은 **yield**문을 발견하고  
**gen_fn**이 일반적인 함수가 아닌 제너레이터 함수라는 것을 안다.
해당 frameobject에 flag를 세워서 이 사실을 기억하도록 한다.  
초기화 이후 `next()`같은 것으로 첫번 째 콜을 할 때, 플래그를 인식해서, 힙의 어딘가에  
기존 서브루틴이 맺히는 곳이 아닌 다른 힙의 어느 공간에 생성.

```python
gen = gen_fn()
"""
type(gen)
>>> <class 'generator'>
"""
```

네가 제너레이터 함수를 호출할 때, 파이썬은 그때 그것의 제너레이터 플레그를 알아볼 수 있다.  
그리고 실제로 함수를 실행하는게 아니라 대신에, 제너레이터를 생성한다.

```python
"""
gen.gi_code.co_name
>>> 'gen_fn'
"""
```

`gen_fn`를 호출함으로써 생기는 모든 제너레이터 네임은 이 모두 이 코드를 가르킨다.  
하지만 이들 모두는, 각각 자신의 스텍프레임을 갖고 있다.  
이 스텍프레임은 파이썬의 스텍프레임에 쌓이는게 아니라, 독특하게도,  
파이썬의 힙에서 언젠간 사용될 것을 기다리며 저장된다.

| object Attr          |  value Object  |
| :------------------- | :------------: |
| PyGenObject.gi_frame | :PyFrameObject |
| PyGenObject.gi_code  | :PyCodeObject  |

PyGenObj madeOf,

| object        | attr                                 |
| :------------ | :----------------------------------- |
| PyFrameObject | f_lasti: 마지막 인스트럭션 포인터(서브루틴과 구별) |
|               | f_locals: 스텍-지역변수                    |
| PyCodeObject  | gen_fn's bytecode                    |

프레임에는 **Last instruction**이라는 포인터가 있고, 마지막에 실행된 지침을 보관하고 있다.  
최초 초기화 시에 last instruction poiter은 -1이다.  
이는 제너레이터가 시작되지 않았음을 의미한다.

```python
gen.gi_frame.f_lasti
>>> -1
```

우리가 **send**를 호출 할떄, 제너레이터는 그의 첫번째 yield를 만나고 멈춘다.  
send에 대한 리턴 값은 1, 왜냐하면 그것이 gen이 yield호출에게 전달한 것이기 때문이다.

```python
gen.send(None)
>>> 1
```

이제 제너레이터의 인스트럭션 포인터는 시작에서부터 3 bytecodes 만큼 떨어졌다.
56바이트로 컴파일된 바이트코드 실행부 중에서 그만큼 진행한 것이다.

```python
gen.gi_frame.f_lasti
>>> 3
len(gen.gi_code.co_code)
>>> 56
```

제너레이터는 언제든 계속 될 수 있다. 왜냐하면 기존의 서브루틴들이 맺히는 부분에 존재하지 않기 때문이다.  
이것이 콜트리(Call hierarchy)에 있는 위치는 고정된게 아니다.  
그리고 이것은 FIFO 의 실행 순서를 따르지 않아도 된다.  
이건 자유롭고, 구름같이 떠다니는 정도의 자유이다.

우리는 제너레이터에게 'hello'라고 value를 send할 수 있다.  
그리고 그것은 yield표현에 대한 결과가 될 것이다.
그리고 제너레이터는 2를 yield할 때까지 수행된다.

```python
res = gen.send("hello")
res
# 2

gen.gi_frame.f_locals
# {"result": "hello"}
```

지금은 `gen`이라는 네임에 묶인 하나의 제너레이터 스텍이지만,  
다른 것으로 또 할당을 하면, 그것의 고유의 스텍프레임과, 지역변수를 가지고 있을 것이다.

우리가 다시 `send`를 호출하면, 제너레이터 코루틴은 그것의 두번째 `yield`를 루틴 컨텍스트로 돌려줄 것이고  
그리고 그것을 `StopIteration` 예외를 발생시켜서, 그것이 종료되었음을 알린다.

```python
gen.send("goodbye")
# generator마지막에 리턴하는 것이 'done' 문자열.
"""
Traceback (most recent call last) :
...
StopIteration: done
"""
```
