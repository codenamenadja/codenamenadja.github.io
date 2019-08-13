---
title: pytest 이해와 사용 
p: tdd/python/pytest
date: 2019-06-12 21:02:11
tags: ['testing', 'python']
---

## major deminority of Default TestCase
   1. Specific test method names (ie, 'assertIsNone', 'assertDictEqual'.. elses)
   2. no database access? then may be to use 'SimpleTestCase'
   3. Test in the same class diff methods uses identical test database in order to properly user setup method.

## differences between pure django test and pytest
   - default TestCase
    ```python
    from django.test import TestCase

    def add(a,b):
        return a+b

    class TestAdd(TestCase):
        def test_add_positive_numbers(self):
            self.assertEqual(add(2, 2), 4)

        def test_add_negative_numbers(self):
            self.assertEqual(add(-2, -2), 4)
    ```

   - Pytest
    ```python
    import pytest
    from upper import add

    @pytest.mark.parametrize('a, b, expected', [
    (2, 2, 4), (-2, -2, -4)
    ])

    def test_add(a, b, expected):
        assert add(a, b) == expected
    ```

## pytest idioms and features
   - 간단한 assert문을 사용한다. pytest는 무엇이 assert되었는지에 따라 적절한 차이를 아웃풋 할것이다.
   - 테스트를 클래스기반으로 그룹 짓지 않고 작성할 수 있다.
   - 재사용 가능한 'fixtures'를 통해서 상태를 셋업 할 수있다. 픽스쳐는 다양한 스코프에서 사용이 가능하다.
   - __Markers__를 통하여 테스트를 카테고리화 하고, 그들의 실행 자원을 변경할 수 있다.
   - 서드파티 라이브러리의, 픽스쳐, 마커, 그리고 테스트 실행 기능성을 위한 플러그인 시스템

## 어떻게 테스트주도개발을 할 것인가?
   1. 고비용의 통합테스트를 조금 작성한다.
   2. 많은 분기의 핸들링을 가지고 있는 부분을 결정한다(ie: error handling). 우리는 그들을 다수의 유닛 테스트로 커버할 수 있다.
   1. 어플리케이션 단에서 제어할 수 없는 부분을 정의한다(ie: 외부 API)
   1. 앱이 모양을 갖추고 나면, 사이에 생략된 테스트들을 작성한다.
