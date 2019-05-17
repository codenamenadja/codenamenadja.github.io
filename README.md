## TODO
    - `hexo new post -p tils/190514/todo`
    - 그날 한 TODO에 대한 과정물은 해당 폴더에 첨가
    - 정리가 끝난 것은 django/, python/ 등에 주제가 확실하게 노출한다.

## EXTRA
    - 'hexo new post -p python/s3_in_django'
    - tags = [python s3 boto3]

## 프로젝트 기록 규칙
    - 'hexo new post -p project/1905_some/01_init
    - workflow 정리
    - tags [project project_some]
    - 원본 파이썬 프로젝트 파일은 삭제하고, 언제든 다시 꺼내 올 수 재구성 하는 것이 목표.
    - workflow.md, photo_views.py, settings.py
    - 모두 소문자 표기를 기준으로 한다.


1. hexo new post -p til/1905xx/todo
2. git add .
3. git commit -m 'todo_1905xx'
4. git push origin develop
5. hexo clean
6. hexo g
7. hexo deploy
