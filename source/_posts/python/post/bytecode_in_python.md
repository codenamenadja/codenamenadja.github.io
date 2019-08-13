---
title: 파이썬 바이트코드에 대한 이해
p: python/post/bytecode_in_python
date: 2019-06-19 18:42:39
tags: ['python', 'python post']
---


> 어떤 언어는 직접 CPUInstruction으로 컴파일하고,  
> 어떤 언어는 실행 중에 즉시 소스코드를 해석한다.  
> 
> 어떤 언어는 중간단계의 세트의 instruction으로 컴파일한다.(실제 CPU를 향한게 아님)  
> 그리고 가상머신(인터프리터)을 실행해서,  
> 실행 중에 그것들을 CPU instruction으로 변환해서 건네준다.  
> 그게 ByteCode이다. 

> 자바도 바이트코드로 변환한다, C#도 그렇다. 파이썬도 그렇다.


```python
#fibonacci.py
#fibonacci.pyc -python2
#pycache - python3
fib.__code__
>>> <code object fib at 0x10fb76930, file '<stdin>' line 1>

fib.__code__.co_consts
>>> (None, 2, 0, 1, (0, 1))

fib.__code__.co_varnames
>>> ('n', 'current', 'next') :local-varname

fib.__code__.co_names
>>> () non-localnames

fib.__code__.co_code
>>> b'|\x00d\x01k....'

import dis
dis.opname[124]
>>> 'LOAD_FAST'

```

***

## Cpython

> CPython은 스텍기반 가상머신이다.  
> 각 함수는 호출되면, 콜 스텍위로 새로운 엔트리로 콜 프레임(=스텍프레임)을 집어 넣는다.  
> 함수가 리턴되면, 이것의 프레임은 스텍에서 popped off된다.  
> * C기반이라, 규칙처럼 적용되서 탈락을 면할 수 없지만,  
> Coroutine은 독립된 스텍프레임으로 힙안에 존재하기 때문에,  
> 함수 실행 이후에(제너레이터 초기화)도  남아있을 수 있다.
> * Cpython은 함수가 실행될 때, 두개의 스텍을 사용한다:  
> evaluation-stack or data-stack, (이것은 지역변수와 인스트럭션)  
> And block-stack(얼마나 많은 블록이 활성화되어있는지 추적한다.  
> loops, try/except, with,...) 


```
fib(8)
```


| ByteCode        | VAL     | Evauation Stack |
| :-------------- | :------ | --------------: |
| 0 LOAD_GLOBAL   | 0 (fib) |            2- 8 |
| 2 LOAD_CONST    | 1 (8)   | 0- function fib |
| 4 CALL_FUNCTION | 1       |


> 자세한 것은 [공식문서 dis 모듈](https://docs.python.org/3/library/dis.html)에 바이트코드에 대한 모든게 들어있다. 꼭 기회를 만들어 보시길.

> 아래는 실제로 c로 돌아가는 코드이다.

```c
switch (opcode){
    TARGET(NOP)
        FAST_DISPATCH();

    TARGET(LOAD_FAST) {
        PyObject *value - GETLOCAL(oparg);
        if(value == NULL) {
            format_exe_check_arg(
                PyExc_UnboundLocalError,
                UNBOUNDLOCAL_ERROR_MSG,
                PyTuple_GetItem(co->co_varnames, oparg)
                );
            goto error;
        }
        Py_INCREF(value);
        PUSH(value);
        FAST_DISPATCH();
    }
}
/*Many, many more bytecode instructions below...*/
```

***

## What can we learn from bytecode?

1. Practical한 효과를 그렇게 보기는 힘들 것이다.
2. 하지만 stack-oriented virtual machine을 확인하는 것은 다르다.
    - 넓은 이해를 가질 수 있고, 다양한 스타일의 프로그래밍을 구사하는데 도움이 된다.
    - instruction과 stack에 대해서 우리가 할수 있는 것은,  
     정말, 정말 극소수지만, 그것은 우리가 찾던 것이다.
  
3. 하지만 실용적인 목적도 있다.
    - c를 기반으로 하는 가상머신을 거의 다 이해할 수 있다.
    - 파이썬도 그냥 그 중 하나이다.
    - 공부에 대한 통찰력을 길러준다.
    - 실제 가상머신이 바이트코드를 실행하는 것에 대해서 이해하면 퍼포먼스에 접근할 수 있다.

***

## Conclusion

> <br/>
> 
> * Python은 항상 C보다 느리다는 것을 주목하라.
> * 그러나! 일반적인 가이드라인을 원하지 않는가? 여기 기술한다.
> ## 1. Local names are Faster than Global ones
> - LOAD_CONST > LOAD_FAST > LOAD_NAME or LOAD_GLOBAL
>   - 왜냐하면 글로벌 네임을 찾아보는 것은 조금 더 복잡하다.
>   - 실제 인스트럭션이 생각보다 길 것이다. (muitple namespaces 사이에서 찾아야 한다.)
> ## 2. Loops and blocks Are expensive
>   - Lookout for SETUP_LOOP, SETUP_WITH and SETUP_EXCEPTION
> ## 3. _Attribute acceses_, _dictionary lookups_ and _list Indexing_ are expensive
>   - Look out for LOAD_ATTR and BINARY_SUBSCR  
 

## addtional recommends to look for..
### 1. Obi ike-Nwosu, "inside the Python Virtual Machine":
   - [https://leanpub.com/insidethepythonvirtualmachine/](https://leanpub.com/insidethepythonvirtualmachine/)
### 2. Alison Kaptur, "A Python Interpreter Written in Python":
   - [http://www.aosabook.org/en/500L/a-python-interpreter-written-in-python.html](http://www.aosabook.org/en/500L/a-python-interpreter-written-in-python.html)
### 3. The CPython bytecode interpreter:
   - [https://github.com/python/cpython/blob/master/Python/ceval.c](https://github.com/python/cpython/blob/master/Python/ceval.c)