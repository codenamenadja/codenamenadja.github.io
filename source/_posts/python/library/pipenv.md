---
title: how to use pipenv 100%
p: python/external_resource/pipenv
date: 2019-05-15 02:19:43
tags: ['python', 'virtualenv', 'pipenv']
---


## Index

[index]: #index

> 1. [개요][i1]
>   1. [목표][i1-1]
> 1. [2][i2]
> 1. [3][i3]
> 1. [4][i4]
> 1. [마치며][i5]

## 개요

[i1]: #개요

Pipenv는 모든 최고의 패키징(bundler, composer, npm, cargo, yarn)을 파이썬으로 가져오기 위한 툴입니다.

이것은 가상환경을 자동으로 생성하고 관리합니다. 또한 `Pipfile`에서 패키지는 추가하고 삭제함으로써 자동으로 가상환경을 생성하고 관리합니다.

Pipenv가 해결하는 복합적인 면 문제는 이러합니다.

- 당신은 더 이상 `pip`와 `virtualenv` 개별적으로 사용하지 않아도 된다.
- reqirements.txt를 관리하는 것은 문제를 발생시킬 여지가 있습니다.  
그래서 우리는 `Pipfile`, `Pipfile.lock`을 사용하여 최근에 테스트된 조합에서부터 추상적인 의존성 선언을 분리합니다.
- 해쉬들은 보안상 어디에든 쓰입니다. 자동적으로 취약접을 드러냅니다.
- 업데이트가 된 요소에 대해서 강하게 최근 버전을 권유함으로써 위험성을 최소화합니다.
- `pipenv graph`를 통해 의존성에 대한 통찰을 드립니다.
- `.env`파일들을 불러옴으로써 개발의 워크플로우를 연결해줍니다.

### Basic_concepts

* 가상환경이 자동으로 생성됩니다.
* 어떤 파라메터도 `install` 명령어에 전달되지 않아도, 명시된 `[packages]`는 설치될 것입니다.
* python3 가상환경을 초기화 하려면 `pipenv --three`


### 목표

[i1-1]: #목표

1. Pipfile과 Pipfile.lock을 이해한다.
2. dev-dependancy와 그렇지 않은 것을 구분한다.
3. 자유롭게 다룬다.

## Pipfile

[i2]: #pipfile

- Example Pipfile

    ```text
    [[source]]
    url = "https://pypi.python.org/simple"
    verify_ssl = true
    name = "pypi"

    [packages]
    requests = "*"

    [dev-packages]
    pytest = "*"

    [requires]
    python_version = ''
    ```
- 일반적인 권유, 버전관리
  - 일반적으로 `Pipfile`, `Pipfile.lock`은 버전컨트롤에 포함하십시오.
  - 만약 복수의 파이썬 버전이 타겟이라면 `Pipfile.lock`은 포함하지 마십시오.
  - 당신이 요구하는 파이썬 버전을 Pipfile의 `[requires]` 섹션에 명시하십시오. 이상적으로라면 하나의 파이썬 버전만 가져야합니다.
  - `pipenv install`은 `pip install`과 완벽히 호환됩니다.

## 기본_워크플로우

[i3]: #기본_워크플로우

1. 당신의 프로젝트 저장소를 생성하고 클론하십시오: `cd my project`
2. Pipfile로부터 설치한다면: `pipenv install`
3. 새로운 패키지를 추가한다면: `pipenv install <package>`
4. Pipfile이 초기화 되었다면: `pipenv shell && python --version`
    - 이것은 새로운 shell subprocess를 생성합니다. `exit`를 통해 deactivate할 수 있습니다.

### 업데이트_워크플로우

[i3-1]: #업데이트_워크플로우

- 무엇이 변했는지 확인할 때: `pipenv update --outdated`
- 패키지를 업그레이드할 때 2가지 옵션:
  1. 모든 것을 업그레이드: `pipenv update`
  2. 하나씩 업그레이드: `pipenv update <pkg>`i

### requirements.txt를_통해

[i3-2]: #requirements.txt를_통해

requirements.txt파일이 존재할 때 `pipenv install`을 실행하면,   pipenv는 자동적으로 파일의 내용을 가져와 Pipfile을 생성해 줄 것입니다.

아래처럼 명시적으로 처리하는 것도 가능합니다.  
`pipenv install -r path/to/requirements.txt`


## 가상환경관리

[i4]: #가상환경관리

Pipenv환경을 관리하기 위한 3가지 주요 커맨드가 있습니다.  
`pipenv install`, `pipenv uninstall`, `pipenv lock`


### pipenv_install

`pipenv install [package names]`
- options
  - `--two`
  - `--three`
  - `--python [version number]`
  - `--dev`: `Pipfile`에 명시된 `develop`와 `default`모두 설치합니다.
  - `--system`: 가상환경이 아닌 글로벌 `pip`커맨드를 사용합니다.
  - `--ignore-pipfile`: `Pipfile`을 무시하고 `Pipfile.lock`으로 설치합니다.
  - `--skip-lock`: `Pipfile.lock`을 무시하고 `Pipfile`로 설치합니다.  
`Pipfile`에 생기는 변화가 `Pipfile.lock`에 반영되지 않습니다.

- install via vcs
  -  `pipenv install -e git+https://github.com/requests/requests.git@v2.20.1#egg=requests`

### pipenv_uninstall

`pipenv uninstall`은 `pipenv install`의 모든 커맨드를 지원합니다. 
- addtional options
  - `--all`: 가상환경의 모든 것을 지우지만 `Pipfile`을 남깁니다.
  - `--all-dev`: `Pipfile`까지 삭제합니다.

## Advanced_usage

[i5]: #advanced_usage

1. 패키지 유효성 이슈 검사
    ```bash
    $pipenv check
    <!-- vulnerable issue로 버전업데이트가 필요하면 Pipfile에 버전명을 수정 -->
    $pipenv install
    $pipenv check
    ```

2. lock이 잘 안된다?
    - black을 설치할때, lock파일 해시 생성하는 곳에서 문제가 생기는데 해결법은 2가지가 있다.
        1. `pipenv lock --pre`: Pre release버전에 대해서 lock을 처리해준다
        2. `pipenv install --dev --skip-lock`: 만약 

## 마치며

[i6]:마치며

간단한 시행착오를 통해 최종적으로 이렇게 다루면 Dev depency와 분리해서 사용하기 좋다.

1. 일반 의존모듈은 pre release가 아니어야한다.
2. dev모듈은 lock을 하지 말자
3. 배포 환경에서는,  pipfile을 가지고 설치하되 dev는 제외하고 설치한다.
```bash
pipenv --three
# 초기화
pipenv install --dev --skip-lock pylint selenium black fabric mypy pytest
pipenv install
```