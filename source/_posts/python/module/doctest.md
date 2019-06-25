---
title: python module doctest
p: python/module/doctest
date: 2019-06-26 15:36:31
tags: ['python', 'python module', 'linting']
---


## Index

[index]: #index

> 1. [개요][i1]
> 1. [doctest.testfile method][i2]

## 개요

[i1]: #개요

doctest모듈은 파이썬 인터프리터 세션과 비슷하게 생긴 텍스트 조각을 찾는다.
그리고 그 세션들을 실행하여, 그들이 보여지는데로 실행되는지 확인한다.
doctest를 사용하기 위한 몇가지 일반적인 방식이 있다.

- 해당 모듈의 docstring들이 잘 업데이트 되어, 모든 상호적인 예시가 적혀진대로 잘 작동하는지 확인하기 위해서.
- 선형회기 테스팅을 실행하여, 테스트파일 혹은 테스트 객체에 적혀있는 상호적인 예시가 예상한대로 잘 작동하는지 실행하기 위해서.
- 인풋과 아웃풋을 통해 패키지의 튜토리얼 문서를 작성하기 위해서. 

```python
"""
this is "example" module.

example module supplies 1 functions, factorial().

>>> factorial(5)
120
"""

def factorial(n:int) -> int:
    """Returns the factorial of n, an exact integer >= 0.
    >>> [factoral(n) for n in range(6)]
    [1, 1, 2, 6, 24, 120]
    >>> factorial(30)
    265252859812191058636308480000000
    >>> factorial(-1)
    Traceback (most recent call lst):
        ...
    ValueError: n must be >= 0

    >>> factorial(30.1)
    Traceback (most rescent call last):
        ...
    ValueError: n must be exat integer
    """
    import math
    if not n >= 0:
        raise ValueError("n must be >= 0")
    if math.floor(n) != n:
        raise ValueError("n must be exact integer")
    if n+1 == n:  # catch a value like 1e300
        raise OverflowError("n too large")
    result = 1
    factor = 2
    while factor <= n:
        result *= factor
        factor += 1
    return result

if __name__ == "__main__":
    import doctest
    doctest.testmod()
```

이것이 생산적으로 **doctest**모듈을 사용하는데 필요한 모든 정보이다.
이후 섹션은 모든 디테일을 포함한다.
특히 표준 테스트 파일의 `Lib/test/test_doctest.py`에서는 유용한 예제를 포함한다.

## TextFile의_docstring_체크

[i2]: #textfile의_docstring_체크

다른 간단한 doctest의 어플리케이션은 텍스트파일의 상호적인 예제를 테스트하는 것이다.

```python
import doctest doctest

>>> doctest.testfile('example.txt')
File "./example.txt", line 14, in example.txt
Failed example:
    factorial(6)
Expected:
    120
Got:
    720
```

이 짧은 스크립트는 `example.txt`안에 상호적인 파이썬 예제를 실행하고 확인한다.

```rst
The ``example`` module
======================

Using ``factorial``
-------------------

This is an example text file in reStructuredText format.  First import
``factorial`` from the ``example`` module:

    >>> from example import factorial

Now use it:

    >>> factorial(6)
    120
```
