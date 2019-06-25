---
title: type-annotation in python
p: python/post/type_annotation_in_python
date: 2019-06-24 19:41:31
tags: ['python', 'python post', 'type']
---

## Index

[index]: #index

> 1. [Annotation의 정의와 필요성][i1]
> 1. [typing module을 통한 annotation확장][i2]
> 1. [annotation이 docstring을 대체하는가?][i3]
> 1. [포메팅 및 테스트 자동화][i4]

## Annotation

[i1]: #annotation

PEP-3107에서 annotation을 소개하고 있다. 기본 아이디어는 힌트를 주는 것이지만, 인터페이스가 구축된 코드를 다룰 수 있도록 함으로,
더욱 정교한 구현이 가능하다. 어노테이션은 타입 힌팅을 활성화 한다.

타입 뿐 아니라 변수를 이해하는데 도움이 되는 어떤형태의 메타데이터라도 지정할 수 있다.

```python
class Point:
    def __init__(self, lat, long):
        self.lat = lat
        self.long = long
def locate(latitude:float, longitude:float) -> Point:
    """맵에서 좌표에 해당하는 객체를 탑색"""
```

이를 통해 함수 사용자는 예상되는 타입을 알 수 있다. 하지만, 파이썬이 타입을 검사하거나 강제하지는 않는다.
또 위처럼 함수 반환 값에 대한 예상 타입을 지정할 수 도 있다럼

그러나 어노테이션으로 타입만 지정할 수 있는 것은 아니다. 파이썬 인터프리터에서 유효한 어떤 것도 사용할 수 있다.
예를 들어 변수의 의도를 설명하는 문자열, 콜백이나 유효성 검사 함수로 사용할 수 있는 callable등이 있다.

어노테이션을 사용하면 `__annotations__`라는 특수 속성이 생긴다. 이 속성은 어노테이션의 이름과 값을 매핑한 사전 타입의 값이다. 앞의 예제에서는 다음과 같이 출력된다.

```python
>>> locate.__annotations__
{'latitude':float, 'longitude': float, 'return': __main__.Point}
```

- PEP-484
    > "파이썬은 여전히 동적인 타입의 언어로 남을 것이다, 타입 힌트를 필수로 하자거나 심지어 관습으로 하자는 것은 전혀 아니다."

코드 전체에 올바른 타입이 사용되었는지 확인하고, 호환되지 않는 타입이 발견되었을떄 사용자에게 힌트를 주는 것이다.
이러한 검사를 도와주는 Mypy에 대해서는 나중에 프로젝트에서 사용하는 방법에 대해 자세히 설명한다.

타입 힌팅은 코드의 타입을 확인하기 위한 도구 이상의 것을 의미한다.
파이썬 3.5부터 새로운 typing모듈이 소개되었고 파이썬 코드에서 타입과 어노테이션을 정의하는 방법이 크게 향상되었다.

기본 개념은 코드의 시멘틱이 보다 읜미있는 개념을 갖게 되면,
코드를 이해하기 쉽고 특적 시점에 어떻게 될지 예측할 수 있다는 것이다.


### typing

[i2]: #typing

[python module typing](/2019/06/25/python/module/typing/) 에서 개괄적으로 다루고 있다.

typing 모듈을 사용하면 파이썬에게 이터러블, 시퀸스가 필요하다고 알릴 수 있다.
심지어 타입이나 값을 식별할 수도 있다. 예를 들어 정수의 시퀸스가 필요하다고 정의하는 것이다.

**PEP-526** 파이썬 3.6부터는 함수 파라미터와 리턴타입 뿐만 아니라, 변수에 직접 주석을 달 수있다.

```python
class Point:
    lat: float
    long: float

>>> Point.__annotations__
{'lat': <class: 'float'>, 'long': <class 'float'>}
```

## docstring_annotation

[i3]: #docstring_annotation

[python module doctest](/2019/06/26/python/module/doctest)에서 doctest와 함께 다룬다.

### 어노테이션은 docstring을 대체하는 것일까?

어노테이션을 대체하기 위한 동적언어로서 초기 서브형태가 파이썬에 있어서는 docstring이라고 할 수 있다.

기존에는 docstring을 통해 함수의 파라미터, 속성의 타입을 문서화 하였기 때문에,
docstring 대신 annotation을 사용해야 하는 것인지 궁금할 수 있다.
심지어 함수의 기본정보, 파라미터 의미와 타입, 반환값, 발생가능한 예외까지 docstring으로 작성하는 포매팅 컨벤션도 유행했었다.

대답은 '예, 사용해야합니다.'이다.
왜냐하면 docstring과 annotation은 서로 보완적인 개념이기 때문이다.

docstring에 포함된 정보를 일부 annotation으로 이동 시킬 수 있었던 덕에 docstring은 좀더 명확한 목적을 가질 수 있게 됐다.
docstring을 통해 친절한 설명과 `>>> doit()`문을 통해 doctest와 연동하는 것이다.

친절한 설명이라 함은, 동적 데이터 타입과, 중첩 데이터 타입의 경우 예상 데이터 예제를 제공하여 어떤 형태의 데이터를 다루는지 제공하는 것이 좋다.

```python
def data_from_response(responst:dict) -> dict:
    if response["status"] != 200:
        raise ValueError
    return {"data": response["payload"]}
```

이 함수는 사전 형태의 파라미터를 받아 사전 형태의 값을 반환한다. 그러나 상세한 내용은 알 수 없다.

1. response객체의 올바른 인스턴스는 어떤 형태일까?
2. 결과의 인스턴스는 어떤 형태일까?

이 두가지 질문에 모두 대답하려면 환수 반환값의 예상 형태를 docstring으로 문서화 하는 것이 좋다.

```python
def data_from_response(response:dict) -> dict:
    """if response has valid 'status' returns response["payload"]

    - response sample
    {
        "status":200, # int
        "timestamp: "....", # 현재시간 ISO포멧 문자열
        "payload": {...} # 반환하려는 데이터
    }

    - return sample
    {"data":{...}}

    - exceptions
    - if HTTP status != 200 raise ValueError
    """
    pass
```

이 문서는 입출력 값을 더 잘 이해하기 위해서 뿐만 아니라, 단위테스트에서도 유용한 정보로 사용된다.

하지만 docstring을 사용했을때 의 이슈는 코드가 좀 더 커지게 되고,
실제 효과적인 문서가 되려면 보다 상세한 정보가 필요하다는 것이다.

## Makefile_자동검사설정

[i4]: #makefile_자동검사설정

리눅스 개발환경에서 빌드를 자동화 하는 가장 일반적인 방법은 Makefile을 사용하는 것이다.
**Makefile**은 프로젝트를 컴파일하고 실행하기 위한 설정을 도와주는 파워풀한 도구이다.
빌드 외에도 포메팅검사나 컨벤션 검사를 자동화하기 위해 사용할 수도 있다.

```makefile
typehint:
mypy src/ tests/

test:
pytest tests/

lint:
pylint src/ tests/

code-format:
black - 1** 79 *.py

doc-list:
pydocstyle src/ tests/

checklist: lint typehint test

.PHONY: typehint test lint checklist
```

`make checklist`

이 중 어떤 단계라도 실패하면 전체 프로세스가 실패한 것으로 간주한다.