---
title: 파이썬 인터프리터에 대한 이해
p: python/post/python_interpreter
date: 2019-06-19 18:44:43
tags: ['python', 'interpreter', 'python post']
---


## 1. intoduction
1. execution of python as split into three main phases

Initialization:
- 다양한 데이터 구조의 셋업, 파이썬 프로세스에 의해 필요하다. 
파이썬 인터프리터 셸과 상호적이지 않을 때 의미가 있다고 생각된다.

Compiling:
- 소스코드를 파싱한다.
- syntex트리로 만들기 위해 
- abstract syntex tree(AST)를 생성하기 위해 
- 심볼테이블을 생성하고, 코드 객체를 생성한다.

Interpreting:
- 생성된 코드객체의 실행을 의미한다.

| initialize | compile                  |
| :--------- | :----------------------- |
| 1.main     | 4.parse tree generate    |
| 2.Py_Main  | 5.AST generate           |
|            | 6.bytecode generate      |
|            | 7.bytecode optimization  |
|            | 8.code object generate   |
|            | end: codeobect execution |

> ./Programs/python.c : handles initialization

> main function call **Py_main** located in ./Modules/main.c

> main.c handles interpreter initialization process

> 초기화의 과정으로서, Py_Initialize from pylifecycle.c가 호출된다.  
> - 이것이 인터프리터의 자료구조와 쓰레드상태의 자료구조를 초기화 한다.(매우 중요한 자료구조)

- 인터프리터 상태 자료구조
```c
typedef struct _is {

    struct _is *next;
    struct _ts *tstate_head;

    PyObject *modules;
    PyObject *modules_by_index;
    PyObject *sysdict;
    PyObject *builtins;
    PyObject *importlib

    PyObject *codec_search_path;
    PyObject *codec_search_cache;
    PyObject *codec_error_registry;
    int codecs_initialized;
    int fscodec_initialized;

    PyObject *builtins_copy;
} PyInterpreterState;
```