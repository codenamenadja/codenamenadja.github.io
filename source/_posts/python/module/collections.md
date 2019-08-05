---
title: python collections 모듈
p: python/module/collections
date: 2019-08-03 15:55:54
tags: ["python", "python module"]
---

## Index

___

1. [**개요**][i1]
1. [**ChainMap**][i2]
1. [**namedtuple**][i3]
1. [**Counter**][i4]
1. [**UserDict**][i5]
1. [**UserList**][i6]

___

## 개요

[i1]: #개요
|       타입       | 설명                                                |
| :------------: | :------------------------------------------------ |
| `namedtuple()` | **팩토리 함수,** tuple형태의 명명된 필드를 가진 서브클래스를 생성하는것에 고성능 |
|    `deque`     | **list같은 컨테이너,** 양쪽 끝에서 더하고 빼는게 빠르다               |
|   `ChainMap`   | **dict같은 클래스,** 복수의 매핑들로 구성된 하나의 뷰를 생성하는데 사용.     |
|   `Counter`    | **dict의 서브클래스,** hashable객체의 키값을 세기 위한 용도             |
| `OrderedDict`  | **dict의 서브클래스,** 순서가 있는, 엔트리가 추가됐다는 것을 기억한다.       |
| `defaultdict`  | **dict의 서브클래스,** 빠진 값을 처리하기위한 팩토리 함수              |
|   `UserDict`   | **dict객체의 래퍼,** dict의 서브클래싱을 쉽게 해준다.              |
|   `UserList`   | **list객체의 래퍼,** list의 서브클래싱을 쉽게 해준다.              |
|  `UserString`  | **string객체의 래퍼,** string의 서브클래싱을 쉽게 해준다.          |

* deque는 의도가 명확하고, namedtuple또한 의도와 사용법이 한정적이기 떄문에, 그 외의 것들을 다룬다.
* 서브클래싱은 팩토리, 인스턴스를 의미한다.

___

## ChainMap

[i2]: #ChainMap

### ChainMap_introduction

ChainMap클래스는 빠르게 다수의 `mappings`를 연결하기 위해 구성되었다.
따라서 그들은 하나의 유닛으로 다뤄질 수 있다. 때로 새로운 딕셔너리를 생성해서, 여러번 `update()`를 실행하는 것보다 더 빠를 수 있다.

* 기본적으로 리스트 컴프리헨션으로 해당 mapped자료구조들을 풀면 keys를 푼다.
* 매개변수의 args에서 0번째부터의 데이터 유지를 최우선으로 한다.
* ChainMap에서 Values에 대한 접근이 nametuple에서 제한된다. keys로 접근하면 key가 아닌 value를 리턴한다.
* 리턴하는 값들의 순서는 역방향이다. 마지막 매개변수의 키값 등을 먼저 출력.
* 매개변수 후자의 객체(읽히는 순서)에 앞에있는 것을 계속 Update해나가는 식(덮어 씌워짐)

```python
"""
>>> baseline = {'music': 'bach', 'art': 'rembrandt'}
>>> adjustments = {'art': 'van gogh', 'opera': 'carmen'}
>>> list(ChainMap(adjustments, baseline))
['music', 'art', 'opera']

# 위 아래는 행동이 동일하다. 코드를 간결하게 유지하고, 속도도 때로 더 빠르다.

>>> combined = baseline.copy()
>>> combined.update(adjustments)
>>> list(combined)
['music', 'art', 'opera']
"""
```
___

## namedtuple

[i3]: #nametuple

### namedtuple_고찰(키값으로 매핑되는 데이터들 기준)

namedtuple인스턴스의 크기는 class인스턴스보다 크다.(아래표는 byte크기 기준 비교)

> key=["username","id"] values=["asd", "dvd"]로 생성

| type                 |                          nametuple                           |              class               |              class with `__slot__`              |
| :------------------- | :----------------------------------------------------------: | :------------------------------: | :---------------------------------------------: |
| class                |                             888                              |               1056               |                       888                       |
| instance             |                              56                              |                64                |                       56                        |
| ChainMap적용 .keys()   |                   `['sadf', 'sad', 'sad']`                   | `['id', 'username', 'password']` | `TypeError:Not Iterable` cant not make chainMap |
| ChainMap적용 .values() | `TypeError:Tuple indices must be integer or slices, not str` |     `['sadf', 'sad', 'sad']`     |                      pass                       |
| ChainMap적용 list()    |                   `['sadf',' sad', 'sad']`                   |  `['id','username','password']`  |                      pass                       |



* 가장 간결하게 쓰고 싶다면, 그냥 리스트나 튜플로 키값없이 약속된 값 순서대로 타입을 체크해서 처리하는게, 메모리 효율적이다.
* 그러나 인스턴스에 대해 동일한 기능이 필요하거나, 값의 타입 분리가 쉽지 않다면, 키값은 필요하다.
* 함수형 프로그래밍을 통해 해당 객체의 타입만 재정의하고, 기능을 별도함수로 처리하면, 동일함수로직을 여러 객체가 사용하기 위해, 함수가 커지는 부작용이 있다.
* 일단 클래스로 정의하고, `__slots__`를 통해서 사용할 메서드만 정의하고,
* 개별적인 validate메서드를 사용하고, 클래스 오버라이딩을 통해서 메모리를 줄이는 방법이 있다.
* namedtuple의 장점은, 코드 양이 적으니, 파일 크기가 적고, 개발자들 사이에서 더 간결한 표현일 수 있다는 점이다.

```python
"""
>>> Slot_dict.__dict__
mappingproxy({'__module__': '__main__',
              '__slots__': ['username', 'id'],
              '__init__': <function __main__.Sample.__init__(self, s1, s2)>,
              'id': <member 'id' of 'Sample' objects>,
              'username': <member 'username' of 'Sample' objects>,
              '__doc__': None})

>>> n_tuple.__dict__
mappingproxy({'__module__': '__main__',
              '__doc__': 'first(username, id)',
              '__slots__': (),
              '_fields': ('username', 'id'),
              '__new__': <staticmethod at 0x7f6aec52b7b8>,
              '_make': <classmethod at 0x7f6aec52bcf8>,
              '_replace': <function namedtuple_first.first._replace(_self, **kwds)>,
              '__repr__': <function namedtuple_first.first.__repr__(self)>,
              '_asdict': <function namedtuple_first.first._asdict(self)>,
              '__getnewargs__': <function namedtuple_first.first.__getnewargs__(self)>,
              'username': <property at 0x7f6aec4cc1d8>,
              'id': <property at 0x7f6aec4cc2c8>,
              '_source': "namedtuple 원본의 __doc__"
>>> slot_dict_instance["username"]
'asd'
>>> nt.username
'asd'
"""
```

* csv와 sqlite를 활용

```python
EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')

import csv
for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
    print(emp.name, emp.title)

import sqlite3
conn = sqlite3.connect('/companydata')
cursor = conn.cursor()
cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
for emp in map(EmployeeRecord._make, cursor.fetchall()):
    print(emp.name, emp.title)
```

* 3.7버전 이후부터 `_source`속성이 사라지고, `namedtuple(NAME:str, FieldNames:iterable, defaults:iterable = None)` 초기값을 `kwargs['defaults']`로 설정 할 수 있다.
* namedtuple을 상속받고 `__slots__`를 구현한 클래스
```python

class Point(namedtuple('Point', ['x', 'y'])):
    __slots__ = ()
    @property
    def hypot(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
    def __str__(self):
        return 'Point: x=%6.3f  y=%6.3f  hypot=%6.3f' % (self.x, self.y, self.hypot)
"""
>>> for p in Point(3, 4), Point(14, 5/7):
...     print(p)
Point: x= 3.000  y= 4.000  hypot= 5.000
Point: x=14.000  y= 0.714  hypot=14.018
"""
```
> 위의 서브클래스는 `__slots__`를 빈 튜플로 정의한다. 이것은 **Instacne dictionarie를 생성하는 것을 막도록 하여** 메모리 요구사항을 낮게 돕는다.

### namedtuple's_hidden_methods

* `somenamedtuple._()`

```python
"""
>>> t = [11,22]
>>> Point._make(t)
OrderedDict([('x', 11), ('y', 22)])
"""
```

* `somenamedtuple._asdict()`

```python
"""
>>> p = Point(x=11, y=22)
>>> p._asdict()
OrderedDict([('x', 11), ('y', 22)])
"""
```

* `somenamedtuple._replace(**kwargs)`

```python
"""
>>> p = Point(x=11, y=22)
>>> p._replace(x=33)
Point(x=33, y=22)
"""
```

### namedtuples'_hidden_attributes

* `somenamedtuple._fields`

```python
"""
>>> p._fields
('x', 'y')
>>> Color = namedtuple('Color', 'red green blue')
>>> Pixel = namedtuple('Pixel', Point._fields + Color._fields)
>>> Pixel(11, 22, 128, 255, 0)
Pixel(x=11, y=22, red=128, green=255, blue=0)
"""
```

* `somenamedtuple._fields_defaults`

> defaults가 뒤에서 부터 length에 맞춰서 처리

```python
"""
>>> Account = namedtuple('Account', ['type', 'balance'], defaults=[0])
>>> Account._field_defaults
{'balance': 0}
>>> Account('premium')
Account(type='premium', balance=0)
"""
```




## Counter

[i4]: #Counter

### Counter_개요
카운터툴은 빠르고 편리한 합계를 위해 제공된다.

* Iterable한 객체의 값들의 개수를 세기위해서 사용한다.
* dict까지는 불필요 한데, Set이 중복의 수를 처리할 수 없어서 문제라면, Counter객체를 고려할만하다.

```python
"""
>>> # Tally occurrences of words in a list
>>> cnt = Counter()
>>> for word in ['red', 'blue', 'red', 'green', 'blue', 'blue']:
...     cnt[word] += 1
>>> cnt
Counter({'blue': 3, 'red': 2, 'green': 1})

>>> # Find the ten most common words in Hamlet
>>> import re
>>> words = re.findall(r'\w+', open('hamlet.txt').read().lower())
>>> Counter(words).most_common(10)
[('the', 1143), ('and', 966), ('to', 762), ('of', 669), ('i', 631),
 ('you', 554),  ('a', 546), ('my', 514), ('hamlet', 471), ('in', 451)]
 """
```

> 기존 객체나, ChainMap등에서는 update메서드가 키값에 값을 새롭게 초기화 하지만, Counter에서 update는 중복 시 기존값에 int값을 더한다.

> 관련 메서드로는 기존값에서 가장 많은 것을 추출하거나, 요소를 그 수만큼 list로 나열하거나, 중복될 경우 수 만큼 빼거나 하는 기존 컨텍스트를 유지하는 기능들이 즐비하다.

```python
"""
# .most_common(i:int)
>>> Counter('abracadabra').most_common(3)  # doctest: +SKIP
[('a', 5), ('r', 2), ('b', 2)]

# .subtract(c:Counter) # 없던 키값은 갱신(정수는 값을 줄이고 음수는 값을 늘린다.)
>>> c = Counter(a=4, b=2, c=0, d=-2)
>>> d = Counter(a=1, b=2, c=-3, d=4 ,e=3)
>>> c.subtract(d)
>>> c
Counter({'a': 3, 'b': 0, 'c': 3, 'd': -6, 'e': 3})

# .elements()
>>> c = Counter(a=4, b=2, c=0, d=-2)
>>> sorted(c.elements())
['a', 'a', 'a', 'a', 'b', 'b']
"""
```

## UserDict

[i5]: #UserDict

### UserDict_양상

```python
# UserDict([initialdata]:dict)
"""
>>> apples:UserDict = UserDict({'r': 1, 'g':2, 'y':3})
>>> apples.data:dict
{'r':1, 'g':2, 'y':3}
"""
```

이것은 단순히 dict객체를 래핑하고 있는 객체이고, 기본 dict 클래스를 상속받는다.
2.1버전까지는 직접 dict를 서브클래싱 하는 게 불가능했기 때문에, 현재로서는 moot print같은 존재이다.(고려 대상이 아니다.)

### UserList_User_String

동일하게 data를 통해 원본 객체에 접근이 가능하고, 기본적으로 래핑하고 있어서
서브클래싱을 도와주고 다른 타입으로 만들어주는? 것이라고 만 생각하면 좋다.