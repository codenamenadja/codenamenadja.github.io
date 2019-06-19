---
title: functional programming howto 번역
p: fp/python/functional_programming_howto
date: 2019-06-19 18:57:26
tags: ['python', 'functional programming', 'translates', 'ongoing']
---


## Intoduction
- 대부분의 연어는 절차적이다
  - 프로그램은 인스트럭션의 리스트이다. 그 지침들은 컴퓨터에게 프로그램의 인풋으로 부터 무얼 해야할 지에 대해서 알려준다.
  - C, pascal, Unix셸 마저 절차적인 언어이다.
- 선언형 언어에서는, 
  - 너는 풀려져야 할 문제에 대한, 특질을 기술한다. 그리고 언어의 실행 및 구현은 컴퓨터적으로 어떻게 효율성을 수행해야 할지 알려준다.
- 객체지향프로그램은 객체의 콜렉션을 다룬다리
  - 객체는 내부적인 상태를 가지고 있고, 그것을 쿼리하거나, 내부적인 상태를 수정하는 메서드를 지니고 있다.
  - Smalltalk와 java는 객체지향 언어이다.
  - C++과 파이썬은 객체지향을 지원하는 언어이다. 그러나 객체지향을 강요하진 않는다.
- 함수형 프로그램은 문제를 functions의 조합으로 분해한다.
  - 함수들은 인풋만을 받고, 아웃풋을 내놓는다.
  - 내부적인 상태가 인풋에 대응하는 아웃풋에 영향을 주지 않는다.
  - 잘알려진 함수형 언어는 MLfamily와(ML,OCaml, 그리고 다양한 것들) Haskell이 있다.

(중략)
함수형 프로그래밍은 객체지향의 반대 쪽으로 고려될 수 있다. 객체들은 내부적인 상태를 캡슐하고 있고, 그것은 메서드의 컬렉션을 통해서 내부적인 상태를 조정한다.
그리고 프로그램이 상태를 변화시키기 위한 적절한 세트를 지니고 있다.
함수형 프로그래밍은 상태의 변화를 최대한 피하려고 한다, 그리고 functions사이에서 데이터의 흐름을 다루려고 한다.
파이썬에서는 이 두 가지 접근법을 조합해서, 객체를 표현하는 인스턴스를 리턴할 수 있다.  

함수형 디자인은, 그 밑에서 일하기 에는, 이상한 강제로 보일 수 있다. 왜 객체는 사이드 이펙트를 피해야만 하는가?
함수현 언어에는 이론적이고 실용적인 이점이 있다.

1. 형식으로 입증가능
2. 모듈성
3. 조합성
4. 디버그와 테스팅의 편리함
(중략)
---

## iterators
iterators는 파이썬의 함수형 스타일의 중요한 기초이다.

iterator은 데이터의 스트림을 대표하는 객체이다.
이 객체는 한번에 하나의 데이터만 리턴한다.
파이썬 이터레이터는 `__next__()`라는 메서드를 지원해야한다. 그것은 어떤 매개변수를 받지 않고, 스트림의 다음 요소만을 반환한다.
만약 다음요소가 없다면, 그것은 `StopIteration`예외를 발생시킨다.
iterator은 한정되야할 필요는 없다. 이터레이터가 무한한 스트림의 데이터를 생성하는 것은 합리적인 것이다.

빌트인 `iter()`함수는 그 aribitary한 객체를 받아서, 그 객체의 컨텐츠나 요소들을 반환하는  **iterator**을 반환하려고 한다.
만약 객체가 **iteration**을 지원하지 않는다면, `TypeError`을 Raise한다.
몇몇 파이썬 빌트인 데이터 타입은 iteration을 지원한다. 가장 흔한 것은 lists와 dictionary들이다.
객체는 **iterable**하다고 불릴 수 있다, 만약 거기서 **iterator**을 얻을 수 있다면.

```python
>>> L = [1, 2, 3]
>>> it = iter(L)
>>> it  #doctest: +ELLIPSIS
<...iterator object at ...>
>>> it.__next__()  # same as next(it)
1
>>> next(it)
2
>>> next(it)
3
>>> next(it)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```
파이썬은 **iterable objects**를 다양한 컨텍스트에서 예상한다. 가장 중요한 개체는 `for`진술이다.
`for X in Y` 진술에서, **Y**는 이터레이터거나, `iter()`가 **iterator**을 생성할 수 있는 것이다.

아래 두 진술은 동일하다.
```python
for i in iter(obj):
    print(i)

for i in obj:
    print(i)
```
이터레이터는 <u>list, tuple등으로 물질화 될 수 있다.</u> `list() tuple()`  constructor함수를 사용함 으로써.

```python
>>> L = [1, 2, 3]
>>> iterator = iter(L)
>>> t = tuple(iterator)
>>> t
(1, 2, 3)
```
**Sequence unpacking**은 이터레이터를 지원한다.
만약 이터레이터가 N개의 엘레먼트를 반환할 것을 네가 알고 있으면, N-tuple로 Unpack할 수 있다.
```py
>>> L = [1, 2, 3]
>>> iterator = iter(L)
>>> a, b, c = iterator
>>> a, b, c
(1, 2, 3)
```
> `for X in somelist`는 물질화된 것이고  
`for X in iterator`은 그렇지 않은 것이다.
> - 갯수가 정말 작다면, 혹은 단계적으로 찾아가서 특정조건까지 Search하는 경우가 아니라면 materialed한 Iterator을 사용하는 것이 더 바람직 할 수 있으나,
> - `max() min()`
> - `if X in iterator`
> - 등의 경우에 무엇이 효과적인지 잘 알고 있을 것이다.

> 파이썬의 리스트는 지역성이 제한적으로 성립되는 value의 모임이 아닌, reference들의 모임이기 때문에.  
> offset만큼 정확히 이동하지도 않고, 다소 애매할 수 있는 존재이다.  
> 
> 만약 list의 N 번쨰 요소가 이미 메모리에서 잡아 놓았던 값이라면, 그 N번쨰 방에는 생성된 값이 지역성과 무관하게 기존에 잡혀있는 메모리에 대한 레퍼런스를 특징함으로 파이썬은 메모리를 절약한다.

> 그렇다고 봤을때, 리스트의 시퀸스가 지역성을 보장하지 않는다면, 하나씩 레퍼런스 값을 생성하는, Iterator이 우세하다.

max(), min()같은 빌트인 함수들은 하나의 이터레이터 매개변수를 받아서 하나의 결과를 돌려줄 수 있다.
"in", "not in" 연산자 또한 이터레이터를 지원한다.: stream안에서 X를 찾는다면, True아니면 False이다.

너는 명백한 문제로 달려들 것이이다, 만약 이터레이터가 무한하다면, 절때 스트림에서 최대값이나 최소값을 찾을 수 없고, 스트림안에 존자하는지 아닌지에 대한 결과도 무한히 지속되어 끝나지 않을 것이기 때문이다.
***
## 이터레이터를 지원하는 데이터 타입

우리는 이미 어떻게 리스트와 튜플이 이터레이터를 지원하는 지 보았다. 사실 어떤 파이썬 시퀸스 타입은 자동으로 이터레이터의 셍성을 도울 것이다.

`iter()`를 딕셔너리에 대해서 호출하는 것은, Keys에 대해서 루프 하게 된다.
```python
m = {'Jan': 1, 'Feb': 2, 'Mar': 3, 'Apr': 4, 'May': 5, 'Jun': 6,
  'Jul': 7, 'Aug': 8, 'Sep': 9, 'Oct': 10, 'Nov': 11, 'Dec': 12}
for key in m:
  print(key, m[key])
Jan 1
Feb 2
Mar 3
Apr 4
May 5
Jun 6
Jul 7
Aug 8
Sep 9
Oct 10
Nov 11
Dec 12
```
> 파이썬 3.7 딕셔너리 이터레이션의 순서는 주입된 순서대로 보장된다.

`iter()`을 사용하면 자동으로 키를 기반으로 이터레이션 루프가 적용된다. 그러나 딕셔너리 그 자체에 이미 다른 이터레이터를 반환하는 메서드가 있다. 

`items()` : (key, val)Pair로 원본 객체에 대한 view객체로 만들어준다.
`values(),keys()` : 위와 동일하게 원본에 대한 레퍼런스를 유지한다.(dynamic-view)

(Key, Value)- 튜플로 구성된 시퀸스를 dict()로 wrap해주면, 마찬가지로 원본의 형태로 객체로 형태를 돌려서 주지만, 원본과의 레퍼런스는 끊어진다.

> 파일에서는, `readline()` 그리고 set는 그 자체로 iterable하다.
***
## Generator expressions and list comprehensions

Two common operations on iterator's output->
1. Performing some operation for every elem
2. seslction a subset of elems that meet some condition

List comprehension N generator expressions는 그러한 실행의 구체적인 언급이다.
> 그것들은 하스켈에서부터 가져온 것이다.
```python
line_list = ['   line 1 \n', 'line 2    \n',]
#gen expression -- return iterator
stirpped_iter = (line.strip() for line in line_list)
#list comprehsions -- returns list
stripped_list = [line.strip() for line in line_list]
```

List comprehension을 통해 너는 파이썬의 리스트를 받는다.  
그것은 물질화 된 것이다.  

nested-Generator expression은 반면에,  
물질화된 것을 돌려주는 것이 아니라,
필요한 때에 하나씩 value를 compute해준다.  

그것은 list comprehension이 계속적으로 스트림을 이어가야 하는 작업, 무한히 이어지는 시퀸스에 대해서 적합하지 않다는 것이다.

Generator-exp는 "()"에 감싸여서, 사용되는데 아래와 같다.
```python
( expression for expr in sequence1
             if condition1
             for expr2 in sequence2
             if condition2
             for expr3 in sequence3 ...
             if condition3
             for exprN in sequenceN
             if conditionN )
seq1 = 'abc'
seq2 = (1, 2, 3)
[(x, y) for x in seq1 for y in seq2]  #doctest: +NORMALIZE_WHITESPACE
# [('a', 1), ('a', 2), ('a', 3),
# ('b', 1), ('b', 2), ('b', 3),
# ('c', 1), ('c', 2), ('c', 3)]
```
if는 해당 시퀸스 레벨에서 적용되는 것이고, 가장 오른쪽을 최초로 왼쪽으로 점차 wrap해 나간다.

(중략 - 아는 내용 너무 많음)

## Passing values into a generators

파이썬 2.4포함 이전 버전에서는, 제너레이터는 오직 아웃풋만을 생산할 수 있었다.
제너레이터 내부로 값을 보내는 것은 불가능 했다.
```python
def gens()
  f_val = (yield 1)
  s_val = (yield f_val)
  return 'done'

outergens = gens
f_yield = outergens.send(None)
# expect f_yield == 1 is True

s_yield = outergens.send(999)
  # send999 => execution on outergens works
    # 1. f_val delegates 999
    # 2. yield f_val => loadFast f_val(:999)
```
나는 yield표현 주변에 항상 괄호를 사용하는 것을 추천한다.

> 실제 바이트코드 단위에서 val에 대한 할당 이전에 execution을 main컨텍스트로 전환하기 때문에, 그 이전에 yield 1 이라는 컨텍스트를 돌려주기 전에 전달된 값은 무엇이든 무시된다.
>
> 그래서 관례상 conventional Rule로서, 처음에 None만을  send 할 수 있도록 강제되어 있다.

> 2번쨰 send를 할때 전달된 값이 제너레이터 내부로 컨텍스트 전환과 함께 할당이 이루어 지면서, yield에서 전달값에 대한 로컬 네임스페이스를 리턴하라는 명령을 만나면, 로드하고, 리턴한다.

> yield문이 추가로 있어도, return문을 만나면  stopIteration을 Raise한다.

- pep 342는 정확한 룰을 설명한다. yield-expression은 항상 parenthesized해야한다.
- 반면 이런표현에서 주의해야한다.
  ```python  
  val = yield i # 라고 사용해도 되지만,
  # 아래와 같은 경우는 꼭 괄호를 사용해라
  val2 = ((yield i) + 12)
  ```
- when it occurs at top-level expression on the right-hand side of an assignment.  
  : 최상위 표현에서, 오른편에 다른 연산이 있을 경우 무조건 괄호를 사용하라.

값들은 제너레이터에 `send(value)`를 통해서 전달된다.
이 메서드는 제너레이터의 `__code__`의 컨텍스트로 진입하고, `yield`표현은 특정 값을 리턴한다.

만약 일반적인 `__next__()`가 외부에서 불려진다면,
제너레이터 내부에서 yield는 아무것도 반환하지 않는다.

여기에 1씩 증가시키고, 내부적인 카운터를 변화시키는 것을 허락하는 단순한 함수가 있다.

```python
def counter(max):
  i = 0
  while i < max:
    val = (yield i) # suspend line
    # if value provided, change counter
    # 만약 send를 통해서 val이 외부에서 주입되었다면, 그 값으로 i를 바꾸고 진행한다. 
    if val is not None:
      i = val
    else:
      i += 1
```
```python
it = counter(10) # max를 10으로 잡아놓은 제너레이터

i1 = next(it) # return it.send(None)과 동일
# i1 is 0
i2 = it.send(None) # <- is same with i2 = next(it)
# i2 is 1
i3 = it.send(8) # suspend line에 8을 리턴받고, val은 8이 된다.
# 내부적으로 다음 yield문을 찾아 진행한다.
# i3 == 8
i4 = next(it)
# i4 == 9
i5 = next(it)
# 10으로 바뀌고 POPTOP을 하지 않고,
# return None문을 만나 Stopiteration을 Raise
```
send가 기본으로 일어나는 작용이니, next를 통해서 계속 하기 보다는,
`send(None)`이 본래 모습이라는 것을 정확히 캐치하고 사용하길 바란다.

`send()`에 따라서, 제너레이터에는 2가지 메서드가 더 있다:

1. `throw(type, value = None, traceback =None)`  
    - 제너레이터 내부의 컨텍스트에서 Exception을 raise한다;  
멈춰진 yield의 시점을 통해서 예외처리가 된다.
  
2. `close()`
    - 제너레이터가 이터레이션을 삭제시키기 위해서, GeneratorExit라는 exception을 발생시킨다.
    - 이 exception을 받을때에, 제너레이터의 코드는 `GeneratorExit` 또는 `StopIteration`을 내부적으로 처리해야 한다.
    - 예외를 처리하는 어긋난 것(어떠한 에러를 Raise)을 행한다면, `RuntimeError.close()`를 촉발시킨다.
    - 그것은 파이썬 garbage collector에게서 불려지는 것이며, 제너레이터가 소멸되는 것이다.
    > 만약 GeneratorExit가 일어날때, 코드를 정리하고 싶다면,
    > 
      ```python
    while True:
        try:
            res = mygen1.send(None)
        except Exception as e:
            print(type(e))
            if isinstance(e, StopIteration):
                print("iterStop")
            if isinstance(e, GeneratorExit):
                print("Gen Exit")
        finally:
            print(res)
            print("done")
            mygen1.close()
            # res를 제너레이터부터 받으면 출력하고 바로 close()
            # close()로 내부적으로 GeneratorExit가 일어나고,
            # Stopiteration으로 연결된다.
      ```
이러한 변화의 누적은, 제너레이터를 단방향 정보 생산자에서, 정보의 소비자인 동시에 생산자로 만들었다.

제너레이터는 또한 **coroutine**이 된다.
**subroutine**은 컨텍스트의 주도권을 갖은 후 최상위에서 시작해서, return 지점에서 끝나지만,

**coroutine**은, 시작되고, 종료되고, 재진입이 매우 다양한 위치에서 진행된 수 있다.(`yield`선언문을 통해서)

## Built-in functions
