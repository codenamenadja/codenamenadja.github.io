---
title: 파이썬의 함수형 프로그래밍에 대한 서론 번역
p: functional_programming/python/functional_programming_intro
date: 2019-06-19 18:39:22
tags: ['python', 'functional programming', 'translates']
---

출처 : [https://www.dataquest.io/blog/introduction-functional-programming-python](https://www.dataquest.io/blog/introduction-functional-programming-python)

## 서문
대부분의 우리는 파이썬을 객체지향언어라고 알고있다. 클래스와 객체가 비록 사용하기 쉽지만, 파이썬을 작성하는 데엔 다른 방법이 있다.
자바등의 언어는 객체지향에서 벗어나기 힘들게 만드는 경향이 있지만, 파이썬은 그렇지 않다.
파이썬이 코드를 작성하는데에 다른 접근법을 가능하게 한다는 걸 가지고서, 따라오는 논리적인 질문은, 코드를 적는 다른 방법은 무엇일까? 비록 거기에 많은 답들이 있더라도, 가장 흔하고 대체적인 스타일의 코드 작성법은 함수형 프로그래밍이라고 불린다. 함수형 프로그래밍은 메인 로직이 되어가고 있다.

이 포스트에서 우리는
-객체지향과 비교해 함수형의 기본을 설명한다
-당신이 왜 함수형을 적용하기 힘든지 밝혀낸다
-어떻게 파이썬이 그 둘 사이에서 전환하는지 알려준다

## 1.객체지향 언어와 비교하여,

```python

class LineCounter:
    def __init__(self, filename):
        self.file = open(filename, 'r')
        self.lines = []

        def read(self):
            self.lines = [line for line in self.file]

        def count(self):
            return len(self.lines)
```

- 클래스를 사용하는 것은 위의 사례로 대표된다.
최고의 객체지향은 아니지만, 이것은 객체지향 디자인에 대한 통찰을 보급한다. 클래스 안에서, 거기엔 친숙한 메서드와 속성이라는 개념이 있다. 속성세트와 객체의 상태를 추론하는 속성, 그리고 메서드가 그 상태를 조작한다.   

- 그 두 컨셉의 작업에서, 객체의 상태는 시기에따라 계속 변해야한다. 상태를 변화시키는 것은 증거가 된다, lines 속성 속에서, read()메서드를 부른 이후.
예로서 이것은 우리가 어떻게 이 클래스를 사용할 건지이다.


***


## Scenenario
- example_file.txt contains 100 lines.

```python

lc = LineCounter('example_file.txt')
print(lc.lines)
>> []
print(lc.count())
>> 0

```

  - The lc object must read the file to set the lines property.

```python
lc.read()
```

  -  The `lc.lines` property has been changed.
   - This is called changing the state of the lc

```python
print(lc.lines)
>> [['Hello world!', ...]]
print(lc.count())
>> 100
```

  - 계속 변하는 객체의 상태는 둘다 축복이며 저주이다.
  - 왜 상태의 변화가 부정적인지 알기 위해서, 우리는 대체제를 소개해야 한다. 대체제는 line counter를 독립적인 함수의 연속으로 설계하는 것이다.

```python
def read(filename):
    with open(filename, 'r') as f:
        return [line for line in f]

def count(lines):
    return len(lines)

example_lines = read('example_log.txt')
lines_count = count(example_lines)
```

***

# 2.순수함수로 작업하기

이전의 예제에서 우리는 라인들을 함수만을 이용해서 리스트의 길이를 셀 수 있었다.  
우리가 오직 함수만을 사용할 때, 우리는 함수형 접근을 프로그래밍에 하고 있다,  
그 접근은, 놀랍지 않게도, 함수형 프로그래밍이라 불린다.  

함수형프로그래밍 뒤에 있는 개념은 함수가 상태없이 존재 하기를 요구한다.  
그리고 오직 그들에게 주어진 매개변수으로 출력을 하기를 원한다.  
그 기준에 맞는 함수는 순수함수라 불린다. 이것은 함수형 과 아닌 것 사이를 나누기 위한 것이다.

***

Create a global variable `A`.

```python
A = 5

def impure_sum(b):
    # Adds two numbers, but uses the
    # global `A` variable.
    return b + A

def pure_sum(a, b):
    # Adds two numbers, using
    # ONLY the local function inputs.
    return a + b

print(impure_sum(6))
>> 11

print(pure_sum(4, 6))
>> 10
```

  - 이 순수함수의 이점이 외부효과가 있는 함수 명확한 것은, Reduction의 외부효과들이다.  
  - 외부효과들은 함수 밖에 있는 것에 대해서 함수의 실행이 변화를 주었을 때 일어난다.  
  - 예를 들어 그것들은 우리가 객체의 상태를 바꾸려고 했을 때 일어난다. 어떤 I/O 명령을 실행했을 때, 혹은 print()를 호출했을때,

***

```python
def read_and_print(filename):
    with open(filename) as f:
        # Side effect of opening a
        # file outside of function.
        data = [line for line in f]
    for line in data:
        # Call out to the operating system
        # "println" method (side effect).
        print(line)
```

  - 함수 내부에서 함수 밖에 메모리를 호출하고 조종한다.


  - 프로그래머들은 그들의 코드가 쉽게 테스트되고 디버깅되고 따르기 쉽게 만들기 위해서 외부효과를 줄인다. 그리고 코드의 외부효과가 커질수록, 프로그램을 진행하고 실행의 순서를 이해하기 어려워진다.

  - 비록 모든 외부효과를 없애려고 시도하는 것은 편리하더라도,  
  그들은 프로그래밍을 쉽게 만든다. 우리가 만약 모든 외부효과를 없애려고 했다면,  
  그럼 너는 파일도 읽을 수 없고, print를 호출할 수도 없고  
  모든 변수를 함수 내에서만 초기화 해야한다.
  
  - 함수형 프로그래밍의 지지자들은 이 트레이드 오프를 이해하고 있다.  
  그리고 개발의 결과의 퍼포먼스를 해치지 않는 선에서 가능한 많은 외부효과를 제거하려고 노력한다.

***

## 3.람다함수

- def 형식 이외에 함수 선언에서 우리는 lambda표현을 할 수 있다.  
- 람다 형식은 def선언형식을 가깝게 따라가지만, 이것은 1-1 매핑이 아니다.  
이것은 2개의 정수를 더하는 함수의 예이다.

```python
# Using `def` (old way).
def old_add(a, b):
    return a + b
# Using `lambda` (new way).
new_add = lambda a, b: a + old_add(10, 5) == new_add(10, 5)
>> True
```

  -  람다 표현식은 콤마를 사용하여, 매개변수의 연속성을 분리한다.  
그리고, 정확히 인터프리터가 그 문장의 콜론에 닿는 때에,  
이것은 따로 return에 대한 진술없이 표현식을 리턴한다.  
결국, 변수에 람다함수를 할당하는 것은, 파이썬함수처럼 작동하는 것이고,  
변수를 함수처럼 사용할 수 있다.new_add()

  - 우리가 비록 람다 차체에 변수이름을 할당하지 않았더라도, 이것은 익명의 함수가 된다.  
이 익명함수는 매우 도움이 된다. 특히 다른 함수의 매개변수가 되는 때에.  
  - 예를들어, sorted()함수는 옵션의 Key매개변수를 받는다.  
  그 매개변수는 어떻게 아이템들이 리스트에서 정렬되야 하는지를 나타낸다.

```python
unsorted = [('b', 6), ('a', 10), ('d', 0), ('c', 4)]

# Sort on the second tuple value (the integer).
print(sorted(unsorted, key=lambda x: x[1]))
>> [('d', 0), ('c', 4), ('b', 6), ('a', 10)]
```

***

## 4.Map함수

파이썬에서 매개변수로 함수를 전달하는게 다른언어와 구별되는 일은 아니지만, 이것은 최근 프로그래밍 언어들에서 사용되는 개발이다. 이러한 타입의 행동을 따르는 함수들을 first-class fucntions라고 부른다. 어떤 first-class Func를 포한하는 언어는 함수형 스타일로 작성될 수 있다.

first-class Func와 함께 세트로 함수형 패러다임에서 주요 사용되는 것이 있다. 이 함수들은 iterable 객체를 받는다. 마치 sorted()처럼, 요소에 개별적으로 함수를 적용시킨다. 다음 일부 섹션에서, 우리는 그들을 개별적으로 시험해볼 것이다.
그러나 그들은 일반적인 포멧인 
function_name(function_to_apply, iterable_of_elements).
를 따른다.

- 우리가 실험해볼 함수는 map()함수이다. map()함수는 iterable객체를 받아서, 새로운 iterable객체를 생성한다. 특별한 map객체이다. 새로운 객체는 모든 요소에 적용되는 일급함수를 가지고 있다.

```python
# Pseudocode for map.
def map(func, seq):
    # Return `Map` object with
    # the function applied to every
    # element.
    return Map(
        func(x)
        for x in seq
    )
# 이것은 우리 어떻게 맵을 사용하여 개별 리스트 요소에 10또는 20을 다하는 방식이다.
values = [1, 2, 3, 4, 5]

# Note: We convert the returned map object to
# a list data structure.
add_10 = list(map(lambda x: x + 10, values))
add_20 = list(map(lambda x: x + 20, values))

print(add_10)
>> [11, 12, 13, 14, 15]

print(add_20)
>> [21, 22, 23, 24, 25]
```

map()을 리턴하는 것에서 리스트객체를 리턴하는 것으로 바뀌었다는 점을 주목하라.  
돌려받은 map객체룰 사용하는 것은 네가 list처럼 나오는 것을 기대한다면 좀 불편한 것이다. 첫째로, 그것을 출력하는 것은 모든요소를 보여주지 않으며, 둘째로 너는 한번만을 순회하는 것이 가능하다.  

왜냐면, 그 기록이 map객체의 인스턴스에 기록되기 때문이다.

***

## 4.Filter함수

map과 유사하니 일단 통과

***

## 5.Reduce함수

우리가 볼 마지막 함수인 reduce()는 functools패키지 내부에 있다. 그 함수는 이터러블 객체를 받아, 이터러블객체를 하나의 value로 줄인다. Reduce는 filter(), map()과는 다르다. 왜냐하면 이것은 해당 함수가 2개의 인자를 받기 때문이다.
이것은 우리가 어떻게 reduce()를 사용하여 모든 리스트이 요소를 합하는 예이다.

```python

values = [1, 2, 3, 4]

summed = reduce(lambda a, b: a + b, values)
print(summed)
>> 10
실행도
new = 1+2
(3)
new = new+3
(6)
new = new+4
(10)
# 기억해야할 흥미로운점은 람다표현을 사용하면, 두번째 value에 대해선 실행하지 않아도 된다는 점이다.
# 예를 들어 어떤 함수가 이터러블 객체의 첫 요소만 항상 반환한다고 보자

# By convention, we add `_` as a placeholder for an input
# we do not use.
first_value = reduce(lambda a, _: a, values)
print(first_value)
>> 1

```

***

## 6.List comprehensions로 재작성

우리가 결국 리스트로 변환했기 때문에, 우리는 map(),filter()함수를 리스트-C로 재작성해야한다. 우리가 파이썬고유 표현형식을 통해 리스트를 만드는 것으로 이점을 얻을 수 있기 때문에, 이것이 더 그들을 쓰기에 파이써닉한 방법이다. 이것인 이전의 예제를 리스트-C로 변환할 수 있는 방법이다.

```python

values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Map.
add_10 = [x + 10 for x in values]
print(add_10)
>> [11, 12, 13, 14, 15, 16, 17, 18, 19, 20]

# Filter.
even = [x for x in values if x % 2 == 0]
print(even)
>> [2, 4, 6, 8, 10]

```

예제를 보면, 우리는 람다함수를 더할 필요도 없다! 네가 만약 map()filter()를 너의 코드에 적용하려 한다면, 이것을 가장 추천한다. 먼저 고려해라. 그러나 다음 섹션에서, 우리는 map(),filter()를 사용하여 예시를 보여줄 것이다.

***

## 7.function Partials, 작성하기

떄로 우리는 함수의 행동은 그대로 사용하고 싶으면서, 들어오는 매개변수의 수를 줄이고 싶다. 목적은 하나의 인풋을 저장하여, 저장된 매개변수를 default로 하는 새로운 함수를 만든다. 예를 들어 우리가 2를 더해주는 함수를 작성한다고 가정하자.

```python

def add_two(b):
    return 2 + b

print(add_two(4))
>> 6

```

add_two함수는 일반적 함수인 $f(a,b)=a+b$ 와 유사하다, 오직 다른건 하나의 매개변수를 기본으로 하고 있다는 것이다.  
(a=2)
우리는 partial모듈을 사용하여, argument기본이 설정할 수 있다.  
(람다는 초기화 당시에 라서 람다가 평가되는 시점에 유효한 Reference을 읽어내지만,  
이것은 평가시점이 아니라 설정한 당시 value를 유지한다. 마치 클로저변수처럼.

partial 모듈은 함수를 받아 매개변수를 freezes(value로서)로 고정시킨다, 첫번째 매개변수로부터 시작해서, 디폴트 매개변수 value로서 완성된 새로운 함수를 리턴한다. 

```python

def add(a, b):
    return a + b

add_two = partial(add, 2)
add_ten = partial(add, 10)

print(add_two(4))
>> 6
print(add_ten(4))
>> 14

#partials는 어떤 함수도 받을 수 있다. 표준 라이브러리들도 포함해서.

# A partial that grabs IP addresses using
# the `map` function from the standard library.

extract_ips = partial(
    map,
    lambda x: x.split(' ')[0]
)
lines = read('example_log.txt')
ip_addresses = list(extract_ip(lines))

```