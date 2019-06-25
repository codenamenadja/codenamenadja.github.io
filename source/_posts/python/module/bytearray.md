---
title: 파이썬 bytearray 모듈 클래스 정리
p: python/module/bytearray
date: 2019-06-19 18:55:32
tags: ['python', 'python module']
---


## Syntax

```python
bytearray([source[, encoding[,errors]]])
```

- source

    optional, 만약 소스가:
    - 문자라면, 인코딩이 필수적
    - 숫자라면, array가 해당 사이즈를 갖고 빈 바이트들로 초기화
    - 버퍼인터페이스를 따르는 객체라면, 읽기전용 버퍼객체가 bytes array를 초기화하는데 사용될 것이다.
- encoding
    
    선택적이다, 문자일 경우에만 필수적. 일반적인 값으로 "ascii", "utf-8", "windows-1250", "windows-1252"등을 받는다.
- errors

    선택적이다, 가능한 값들은,:
    - 'strict' : 인코딩 에러가 날 경우 에러를 raise
    - 'replace' : 흉하게 생긴 데이터를 적절한 마크로 변환 시킨다., '?', 'uffd'같은경우
    - 'ignore' : 흉하게 생긴 데이터를 무시하고, 알림없이 진행한다.
    - 'xmlcharrefreplace' : 적절한 XML문자 레퍼런스로 대체한다.(인코딩전용)
    - 'backslashreplace' : \를 이스케이프 시퀸스로 사용한다.(인코딩전용)

## Remarks

    바이트배열 타입은 변환할 수 있는 숫자의 시퀸스, 0<=x<256을 가진다.
    낮은 레벨의 바이너리 데이터를 다루는 데 사용될 수 있다. 예를 들면,
    이미지 내부, 네트워크에서 직접 전달 받을때.

## Example1
```python
bytearray()
# bytearray(b'')
bytearray(4)
# bytearray(b'\x00\x00\x00\x00')
bytearray([0,1,2])
# bytearray(b'\x00\x01\x02')
bytearray(buffer('hello'))
# bytearray(b'hello')
```
## Example2

```python
bytearray('hello', 'ascii')
# bytearray(b'hello')

bytearray(u'źdźbło', 'ascii', 'strict')   #’blade of grass’ in polish
# UnicodeEncodeError: 'ascii' codec can't encode character u'\u017a' in position 0: ordinal not in range(128)

bytearray(u'źdźbło', 'ascii', 'ignore')
# bytearray(b'dbo')

bytearray(u'źdźbło', 'ascii', 'replace')
# bytearray(b'?d?b?o')

bytearray(u'źdźbło', 'ascii', 'xmlcharrefreplace')
# bytearray(b'&#378;d&#378;b&#322;o')

bytearray(u'źdźbło', 'ascii', 'backslashreplace')
# bytearray(b'\\u017ad\\u017ab\\u0142o')
```