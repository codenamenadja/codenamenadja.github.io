---
title: 파이썬이 문자열을 저장할 때 어떻게 데이터를 저장하는가? 번역
p: python/post/how_python_saves_memory_with_string
date: 2019-06-19 18:25:35
tags: ['python', 'memory', 'python post']
---

## 출처 : [https://rushter.com/blog/python-strings-and-memory](https://rushter.com/blog/python-strings-and-memory/)


파이썬3 이후로 문자 타입은 유니코드를 사용하기 시작했다. 유니코드 문자열은
4bytes(16bits)의 크기까지 캐릭터를 인코딩에 따라 수용할 수 있다.
메모리적 측면에서는 때로 고비용이라고 느껴질 수 있다.

메모리소모를 줄이고 퍼포먼스를 개선하기위해 파이썬은
유니코드 문자열의 3가지 내부적인(Internal rep..) 추상화를 이용한다.
1byte per char(Latin-1 endcoding)
2bytes per char(UCS-2 endcoding)
4bytes per char(UCS-4 endcoding)
파이썬을 프로그래밍할때 파이썬의 모든 문자열은 동일하게 동작한다.
그리고 대부분의 경우에 우리는 어떤 변화도 눈치채지 못한다.
그러나 변화는 매우 주목할만하고, 때로는 예상치못하다
(많은 양의 문자열을 다룰때,)

내부적인 추상화에서 차이를 확인하려면, 우리는 sys.getsizeof
를 이용할 수 있다.(그것은 객체의 크기를 bytes로환산해준다.)

```python
import sys
string = 'hello'
sys.getsizeof(string)
>>> 54

# 1-byte encoding
sys.getsizeof(string+'!')-sys.getsizeof(string)
>>> 1

# 2-byte encoding
string2  = '你'
sys.getsizeof(string2+'好')-sys.getsizeof(string2)
>>> 2

#4-byte encoding
string3 = '🐍'
sys.getsizeof(string3+'💻')-sys.getsizeof(string3)
>>> 4
sys.getsizeof(string3)
>>> 80
```
당신이 보는 것처럼 문자열의 내용에 따라, 파이썬은 다른 인코딩을 사용한다. 그것을 기억하라,
파이썬의 모든 개별 문자열은 49~80bytes의 메모리를 추가적인 정보를 보관하는 지점에 추가로 소모한다.
예를 들면 hash, length in bytes, encoding type, string flags
등이 있다. 그것이 빈 문자객체가 49byte의 메모리를 소모하는 이유이다.

우리는 인코딩을 추론할 수 있다, 직접 객체에 대해 ctypes: 를 사용함으로.
```python
import ctypes

class PyUnicodeObject(ctypes.Structure):
    # internal fields of the string object
    _fields_ = [("ob_refcnt", ctypes.c_long),
                ("ob_type", ctypes.c_void_p),
                ("length", ctypes.c_ssize_t),
                ("hash", ctypes.c_ssize_t),
                ("interned", ctypes.c_uint, 2),
                ("kind", ctypes.c_uint, 3),
                ("compact", ctypes.c_uint, 1),
                ("ascii", ctypes.c_uint, 1),
                ("ready", ctypes.c_uint, 1),
                # ...
                # ...
                ]


def get_string_kind(string):
    return PyUnicodeObject.from_address(id(string)).kind

get_string_kind('Hello')
>>> 1
get_string_kind('你好')
>>> 2
get_string_kind('🐍')
>>> 4
```

만약 문자열의 모든 글자들이 ASCll 범위안에 맞을 수 있다면,
그들은 1-byte Latin-1 인코딩을 사용한다. 기본적으로,
Latin-1은 최초의 256 유니코드-캐릭터를 추상화한다.
이것은 많은 라틴어를 지원한다. 그러나 라틴어 계열 언어가 아니라면 그럴 수 없다.
아시아, 아프리카등 일대의 언어에 대해서 그러하다. 이것이
그들의 코드포인트(숫자상의 인덱스들)가 1-byte(0-255)범위 밖을 정의하게 된 이유이다.
이모티콘 절때 쓰지마라 문자의 4배 이상 메모리가 소요된다.
```python
ord('a')
>>> 97
ord('你')
>>> 20320
ord('!')
>>> 33
```
###   왜 파이썬이 내부적으로 UTF-8을 사용하지 않는가?

UTF-8인코딩에 문자가 저장되어 있다면, 개별 글자는 자신이 추상화
하는 정보에 따라 1-4bytes의 메모리를 사용하여 encoded된다.
그것은 저장상 효율적인 인코딩이다.
그러나 한가지 큰 단점이 있다!
-개별 문자가 1-4bytes의 메모리에 저장되어 있어서, 문자를 열람하지 않고서, 인덱싱으로 무작위단어에 대해 접근이 불가능하다.
그래서 쉬운 명령인 string[5]의 경우도 UTF-8파이썬은 요구된 문자를 찾을 때 까지 스캔을 해야한다.
고정된 길이의 인코딩은 그런 문제가 없다.
글자를 인덱스에 따라 원본을 찾기위해, 파이썬은 그저 한 캐릭터의 인덱스 번호를 늘리면 그만이다.

### string interning

빈 문자나 ASC2의 문자열중 특정 문자를 다룰 때, 파이썬은 string interning(문자를 잡아 놓음)을 사용한다.
잡혀있는 문자는 싱글턴 객체처럼 행동해서, 그 말은,
두개의 독립적인 문자객체가 잡혀있다면, 그들은 그들중 하나의 복사본만 메모리에 보관하는 것이다.
* id()메서드를 통한 다른객체 동일 글자에 대한 동일한 주소에 대한 이야기.
이것은 파이썬의 문자객체는 immutable하기 때문이다.

파이썬에서 문자열을 잡아 놓는 것은, 글자나, 빈 문자객체에만 국한되지 않는다.
문자열,(코드를 컴파일하면서 생성된,)은 그들의 길이가 20글자를 초과하지 않는다면,
또한 저장될(잡히다) 수 있다.
이것은 포함한다.
-함수 & 클래스 이름들
-변수명들
-매개변수 이름들
-상수들(코드에 선언된 모든 문자들)
-Dictionary의 키 이름들
-속성의 이름들
```python
a = 'teststring'
b = 'teststring'
id(a), id(b), a is b

>>> (4569487216, 4569487216, True)

a = 'test'*5
b = 'test'*5
len(a), id(a), id(b), a is b

>>> (20, 4569499232, 4569499232, True)

a = 'test'*6
b = 'test'*6
len(a), id(a), id(b), a is b

>>> (24, 4569479328, 4569479168, False)
```
* open('aa.txt','r').read()등을 통한 할당은 적용되지 않는다.
그것은 일종의 외부 스트림에 의존 하는 것이지 상수가 아니기 때문이다.

문자를 잡아 놓는 기술은 동일한 내용의 문자선언을 줄여준다.
내부적으로, 문자를 잡아 놓는 것은 global dictionary에 의해
유지된다.(global dictionary, 문자들이 키로서 보관되는 곳이다. 그곳에
이미 특정 문자열이 메모리 안에 있는지 보기 위해
파이썬은 dictionary의 key조회하는 명령을 수행한다.  