---
title: Inspect 모듈 파헤치기
p: python/pure_resource/module/inspect
date: 2019-06-19 16:37:06
tags: ['python', 'python module', 'ongoing']
---

## Index

[index]: #index

> 1. [공식문서 정의][i1]
>       1. [메서드 리스트][i1-1]
> 1. [Type class별 inpect지원 속성, 메서드][i2]
>       1. [Traceback module][i2-1]
>       1. [Frame module][i2-2]
>           1. [Frame object획득하기][i2-2-1]
>       1. [Code module][i2-3]
>           1. [dis모듈로 코드 해석][i2-3-1]
>       1. [Function module][i2-4]
>       1. [Generator module][i2-5]
>       1. [Coroutine module][i2-6]
> 1. [유용한 활용][i3]
>       1. [지원 타입의 source가져오기][i3-1]
> 1. [마치며][iend]

## 공식문서_정의

[i1]: #공식문서정의

인스펙트 모듈은 유효한 모듈, 클래스, 메서드, 함수, 트레이스백, 프레임 객체, 코드객체에 대해 정보를 가질 수 있는 유용한 함수들을 제공한다.

4가지 종류의 주요 서비스
- 타입체킹
- 소스코드 획득
- 클래스와 함수 측정
- 인터프리터 스텍 확인

## class_Type에_대한_Inspect_백업

[i2]: #class_Type에_대한_inspect의_백업_속성_및_메서드

### Traceback_module

[i2-1]: #traceback_module

| Type      | Attribute | Description             |
| :-------- | :-------- | :---------------------- |
| traceback | tb_frame  | 해당 레벨의 프레임객체            |
|           | tb_lasti  | 바이트코드에서 마지막 시도된 지침의 인덱스 |
|           | tb_lineno | 소스코드상 현재 라인 넘버          |
|           | tb_next   | 다음 내부의 트레이스백 객체         |

### Frame_module

[i2-2]: #frame_module

| Type  | Attribute  | Description                 |
| :---- | :--------- | :-------------------------- |
| frame | f_back     | 다음 외부 프레임 객체(이 프레임의 호출자)    |
|       | f_builtins | 해당 프레임에서 보여진 builtin 네임스페이스 |
|       | f_code     | 이 프레임에서 실행된 코드객체            |
|       | f_globals  | 이 프레임에서 보여진 Global 네임스페이스   |
|       | f_lasti    | 바이트코드에서 마지막으로 시도된 지침의 인덱스   |
|       | f_lineno   | 소스코드상 현재 라인 넘버              |
|       | f_locals   | 현재 프레임에서 보여진 Local 네임스페이스   |
|       | f_trace    | 이 프레임의 함수를 추적, 없으면 `None`   |

#### Frame_object획득하기

[i2-2-1]: #frame_object획득하기

```python
def handle_stackframe_without_leak():
    frame_start = inspect.currentframe()
    do_somethong = lambda : pass
    do_something()
    frame_end = inspect.currentframe()
    try:
        print(frame_start.f_locals)
        print(frame_end.f_locals)
        print(frame_start is frame_end) # True
        # do something with the frame
    finally:
        del frame_start, frame_end
```

### Code_module

[i2-3]: #code_module

| Type | Attribute         | Description                 |
| :--- | :---------------- | :-------------------------- |
| code | co_argcount       | `*args`의 수량                 |
|      | co_code           | 날것으로 컴파일된 바이트코드             |
|      | co_cellvars       | 튜플로 된 셀 객체의 이름들             |
|      | co_consts         | 바이트코드에서 사용된 상수로 구성된 튜플      |
|      | co_filename       | 해당 코드 객체가 생성된 파일명           |
|      | co_firstlineno    | 소스코드에서 첫번쨰 라인의 수            |
|      | co_flags          | 생략(봐도 모르겠다.)                |
|      | co_inotab         | 생략(불필요)                     |
|      | co_freevars       | free Variables의 이름들로 구성된 튜플 |
|      | co_kwonlyargcount | 키워드기반 매개변수의 수               |
|      | co_name           | 해당 코드객체가 정의한 것의 이름          |
|      | co_names          | 지역변수의 이름들로 구성된 튜플           |
|      | co_nlocals        | 지역 변수의 이름들의 숫자              |
|      | co_stacksize      | 가상머신 스텍 공간이 요구됨             |
|      | co_varnames       | 매개변수와 지역변수의 이름들로 구성된 튜플     |

#### dis모듈을_사용하여_객체_분석

[i2-3-1]: #dis모듈을_사용하여_객체_분석

```python
import dis
import pprint
def sample_func(a,b=3,msg=None):
    """
    >>> sample_func(2,msg="expected!")
    (6, "expected! 6")
    """
    return (a*b, f"{msg} {a*b}")

pprint.pprint((dis.code_info(sample)))

"""
(
    'Name: sample\n'
    'Filename: <ipython-input...>\n'
    'Argument count: 3\n'
    'nKw-only arguments: 0\n'
    'Number of locals: 3\n'
    'Stack size: 5\n'
    'Flags: OPTIMIZED, NEWLOCALS, NOFREE\n'
    'Contants: \n'
    '   0: \'\\n    >>> sample_func(2,msg="expected!")\\n    (6, "expected! '
    6")\\n    \'\n'
       1: ' '\n"
    Variable names:\n'
       0: a\n'
       1: b\n'
    '   2: msg')
)
"""
```

### Function_module

[i2-4]: #function_module

| Type     | Attribute | Description                                 |
| :------- | :-------- | :------------------------------------------ |
| function | __doc__   | 문서화 문자                                      |
|          | __func__  | 매서드의 실행부가 포함된 함수객체                          |
|          | __self__  | instance to which method is bound or `None` |

### Generator_module

[i2-5]: #generator_module

| Type      | Attribute    | Description                         |
| :-------- | :----------- | :---------------------------------- |
| generator | gi_frame     | frame                               |
|           | gi_running   | 제너레이터가 실행 중 인가?                     |
|           | gi_code      | code                                |
|           | gi_yieldfrom | `yield from`에서부터 생산되는 객체, or `None` |

### Coroutine_module

[i2-6]: #coroutine_module

| Type      | Attribute  | Description           |
| :-------- | :--------- | :-------------------- |
| coroutine | cr_await   | 대기중인 객체, or `None`    |
|           | cr_frame   | frame                 |
|           | cr_running | 코루틴이 진행 중인가?          |
|           | cr_code    | code                  |
|           | cr_origin  | 코루틴이 생성된 곳, or `None` |


[index][index]

## 유용한_활용

[i3]: #유용한_활용

### 지원_타입의_소스받아오기

[i3-1]: #지원_타입의_소스_받아오기

```python
def sample():
    return 'sample!'

def getCode(any_obj):
    """
    >>> inspect.getsource(sample)
    "def sample():\n    return 'sample!'\n"
    """
    return inspect.getsource(any_obj)
```

### python_code_tracing

[i3-2]: #python_code_tracing

inspect자원을 래핑하여 활용하는 기법을 보자

```python
import sys

def traceit(frame, event, arg):
    if event == "line":
        lineno = frame.f_lineno
        print("line", lineno)

    return traceit

def main():
    print("In main")
    for i in range(5):
        print(i, i*3)
    print("Done.")
    

sys.settrace(traceit)
main()
```
- sys.settrace()에 설정한 함수에 frame, event, arg를 넣고 해당 함수객체를 반환한다.
- 

## 예시_크롤링_동작_코드

[i4]: #예시_크롤링_동작_코드



## 마치며

[iend]: #마치며
