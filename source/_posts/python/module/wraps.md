---
title: functools의 wraps 사용하기
p: python/pure_resource/modules/wraps
date: 2019-06-19 19:35:43
tags: ['python', 'python module']
---

**INDEX**

[1. 공식문서 정의][index-1]

[index-1]: #1_공식문서_정의


## 1_공식문서_정의
   ### `functools.wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)`
   
   - 이것은 `update_wrapper()`를 함수 데코레이터로서 호출하는 편리한 함수이다.
   
   - wrapper함수를 정의할 때, 이것은 아래와 동일하다.
   - `partial(update_wrapper, wrapped=wrapped, assigned=assigned, updated=updated)`
   
   - 예를 들어,

        ```python
        >>> from functools import wraps
        >>> def my_decorator(f):
            @wraps(f)
            def wrapper(*args, **kwargs):
                print('Calling decorated func')
                return f(*args, **kwargs)
            return wrapper

        >>> @my_decorator
            def example():
                """DocString"""
                print("Called example function")

        >>> example()
        Calling decorated func
        Called example function

        >>> example.__doc__
        'DocString'

        >>> example.__name__
        'example'
        ```