---
title: fabric 기본적인 스크립트 작성하기
p: python/external_resource/fabric
date: 2019-06-19 16:17:52
tags: ['fabric', 'python', 'ongoing']
---

## Fabric을 이욜한 자동 배포 스크립트 작성

1. 기본 설치
    - Fabric3
    - pycrpyto: for secure connection
    - paramiko: for secure connection either.

1. fabfile.py 생성
    - manage.py와 같은 위치에 생성해 주고,
    - deploy.json은 deploy_tool 폴더 아래로 생성해준다.
    - _로 시작하는 함수는 fab 명령에서 제외
    - 일반적인 함수 명은 fab deploy, test_connection등으로 실행가능

```python
# where directory fabfile.py with manage.py
PROJECT_DIR = os.path.dirname(os.path.abspath(__file__))
# one step over fabfile direcotry
BASE_DIR = os.path.dirname(PROJECT_DIR)

json_config_file = os.path.join(PROJECT_DIR, 'deploy_tools/deploy.json')
with open(json_config_file) as f:
    envs = json.loads(f.read())
```