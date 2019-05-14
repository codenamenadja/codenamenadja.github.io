## POST파일 생성 규칙
    - `hexo new post -p tils/190514/todo`
    - 그날 한 TODO에 대한 결과는 해당 폴더에 포함시킨다.(md아니게)
    - 정리가 끝난 것은 django/, python/ 등에 주제가 확실하게 노출한다.

## 프로젝트 기록 규칙
    - project는 projects/someproject/workflow.md 등으로 정리
    - 개발자-임의코드가 많은 파이썬 파일과 requirements.txt등을 첨가해도 좋다.
    - 이를 통해 원본 파이썬 프로젝트 파일은 삭제하고, 언제든 다시 꺼내 올 수 있도록 한다.
    - workflow.md, photo_views.py, settings.py

## git flow branch규칙
    - TIL = TIL_190514
    - project = Project_djangorestservice
    - traslate = Translate_pythontestingPost
    - 할 일이 총 3개라고 할때, 각 업무가 끝나면 feat->release
    - 모든 release의 3개가 꽉 차면, release 먼저 rel->feat
    - feat또한 finish해서 소멸 시키지만,
    - 각 작업에는 title_workflow.md 를 생성하여
    - 각 작업의 순서를 테스트코드 적듯이 순차적으로 짧고 확실하게 전달한다.

- 태그 기준
    - TIL TODO 적는 것은 todo til 이라고 한다.
    - 모두 소문자 표기를 기준으로 한다.
    - project의 경우 project 를 꼭 첨가한다. 
