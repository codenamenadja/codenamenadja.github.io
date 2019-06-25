---
title: how to use pipenv 100%
p: python/external_resource/pipenv
date: 2019-05-15 02:19:43
tags: ['python', 'virtualenv', 'pipenv']
---

1. 자신의 파이썬 버전 확인
    ```bash
    $python3 --version
    Python 3.6.7
    ```

2. pipenv 활성화
    ```bash
    <!-- 1.기본 python3 설정 -->
    $pipenv shell
    <!-- 2.버전 특정 환경 -->
    $pipenv --python 3.6.7
    ```
3. 패키지 인스톨과 리스팅
    ```bash
    $pipenv install django
    $pipenv lock -r
    $pipenv uninstall django
    $pipenv install requirements.txt -r
    ```
4. dev패키지 설치
    ```bash
    $pipenv install selenium pep8 autopep8 --dev
    ```

5. 패키지 유효성 이슈 검사
    ```bash
    $pipenv check
    <!-- vulnerable issue로 버전업데이트가 필요하면 Pipfile에 버전명을 수정 -->
    $pipenv install
    $pipenv check
    ```