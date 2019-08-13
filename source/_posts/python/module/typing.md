---
title: typing
p: python/module/typing
date: 2019-06-25 13:07:47
tags: ['python', 'python module']
---

## Index

[index]: #index

> 1. [개요][i1]
> 1. [타입 알리아싱][i2]
> 1. [NewType][i3]
> 1. [Callable][i4]
> 1. [Generic][i5]
> 1. [사용자 정의 generic types][i5]

## 개요

[i1]: #개요

가장 주요한 구조적 지원은 Any, Union, Tuple, Callable, TypeVar, Generic의 타입들로 이루어진다.


## Type_aliases

[i2]: #type_aliases

타입-별칭은 타입을 `alias`로 치환함으로서 정의된다. 예를들어 `Vector`와 `List[float]`는 서로 교환가능한 동의어로써 사용될 수 있다.

```python
from typing import List
Vector = List[float]

def scale(scalar: float, vector:Vector) -> Vector:
    return [scalar * num for num in vector]

# typechecks: 실수의 리스트는 Vector로 정의되었다.
new_vector = scale(2.0, [1.0, -4.2, 5.4])
```

`Type aliases`는 복잡한 타입을 간단히 정의하기에 좋다
```python
from typing import Dict, Tuple, Sequence

ConnectionOptions = Dict[str, str]
Address = Tuple[str, int]
Server = Tuple[Address, ConnectionOptions]

# 정적인 타입체커는 아래 타입을 위의 타입 별칭과 동일하게 취급할 것이다.
def boardcase_message(
    message: str,
    servers: Sequence[Tuple[Tuple[str, int], Dict[str, str]]],
) -> None:
    pass
```

## NewType

[i3]: #newtype

`NewType()`헬퍼 함수를 통해서 고유한 타입을 생성할 수 있다.

```python
from typing import NewType

UserId = NewType('UserId', int)
some_id = UserId(524313)

>>> type(some_id)
int
```
우리는 여전히 모든 `int`연산을 UserId타입에서 수행할 수 있다. 하지만 결과는 모두 `int`타입이 될 것이다.

이러한 테크는 정적인 타입 체커에서만 유효할 것이다.
런타입 레벨에서 `sometype = NewType('sometype', Base)`는 `sometype`을 즉시 리턴하는 함수로 취급하여, 전달한 파라메터를 해당 `Base`-타입으로 돌려보낼 것이다.

그 말인즉슨, `Derived(some_value)`라는 표현은 새로운 클래스나 어떤 오버해드도 일반적인 함수 호출에서 추가되는 것이 아니라는 것이다.

더 깊히 얘기하자면, `some_value is Derived(some_value)`는 런타임에서는 항상 옳다는 것이다.

> 이것은 런타임 소모없이 논리적 에러를 예방하고자 할때 도움이 된다.

## Callable

[i4]: #callable

프레임워크들은 특별한 정의의 콜백함수가 `Callable[[ArgType, Arg2Type], ReturnType]`
같은 형식을 이용해서 타입힌트 되는 것을 기대한다.

에를 들어,

```python
from typing import Callable

def feeder(get_next_item: Callable[[], str]) -> None:
    pass

def async_query(
    on_success: Callable[[int], None],
    on_error: Callable[[int, Exception], None]
    ) -> None:
    pass
```

> `Callable[..., ReturnType]`
> 매개변수의 리스트 타입을 생략으로 대신하는 것으로서 callable의 타입을 정의하는 것도 가능하다.

## Generic

[i5]: #generic

컨테이너 내부에 잡혀있는 객체들에 대한 타입의 정보가 정적으로 유추될 수 없기 때문에,
abstract base classes들은 에상되는 타입들을 컨테이너 요소들을 위해서 기록하기 위해 버전 업 되었다.

```python
from typing import  Mapping, Sequence

def notify_by_email(
    employees: Sequence[Employee],
    overrides: Mapping[str, str]
) -> None:
    ...
```

제네릭들은 `TypeVar`이라는 새로운 타이핑 팩토리를 통해서 파라메터화 될 수 있다.

```python
from typing import Sequence, TypeVar

T = TypeVar('T') # Type변수 선언

def first(l: Sequence[T]) -> T:
    return l[0]
```

## User-defined_generic_types

[i6]: #user-defined_generic_types

이 부분은 제네릭 타입을 통해서 클래스를 동적으로 변화시키는 것처럼 보여진다.
해당 제네릭 타입에 대한 동적인 상속이 이루어지기 때문에, 그렇게 권고하고 싶은 방식이 아니라고 생각한다.
제네릭은 매개변수로서만 쓰여지는 정도가 적당하다고 생각하기 때문에, 이 부분에 대해서는 필요한 경우 공식문서에서 별도로 참조하자.