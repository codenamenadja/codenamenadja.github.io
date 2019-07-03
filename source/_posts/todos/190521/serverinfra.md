---
title: django-uwsgi-nginx 서버 매커니즘 정리`
p: todos/190521/serverinfra
date: 2019-05-21 10:08:30
tags: ['nginx', 'django']
---

1. 클라이언트의 요청이 서버의 NIC에 도달한 이후부터
   1. 웹서버
      - Nginx, Apache
      - 요청과 콜백함수를 같이 미들웨어에 전달
   1. 미들웨어
      - uwsgi
      - gunicorn
   1. 장고 어플리케이션

1. Req를 받고 무엇을 할 것인가.
   1. useragent
      - 사람 여부 판단
      - 데스크톱 모바일
      - 사용자 분석
   1. IP
      - 지역구분
      - 계정이 다른 플랫폼, IP에서 접속했을 때.
   
1. 쿠키와 세션
   - 세션(공통세션과 개별 세션)
      1. 메모리
         - 로그인 여부, 키
         - 세션 해시키에 따라 메모리 주소 구분
         - 
      1. 파일
      1. DB
   - 쿠키(유저)
      1. 세션 해시키를 저장(해당 도메인명 하부에 키값으로 저장)
      1. 서버 측에서 저장시간을 정할 수 있다.

1. 미들웨어
   - 리퀘스트에 대한 사전 처리
   - 세션미들웨어, CSRF미들웨어

1. URLpattern
   - urls.py (URLCONF 모듈)
   - urlpattern (url 라우팅 테이블)
   - <converter:namedGroup>
   - converter이라고 하고 validator이라고 부르기도 한다.
   - 컨버터가 없으면 기본 string, re의 타입에 대한 검사를 해준다.
   - converter : str, int, slug, uuid, path
   - 함수형 뷰에서는 해당컨버터의 네임그룹이, Kwargs로 들어온다.
    ```python
    def home(req, userid):
        pass
    ```
   - urls.py에 app_name 을 설정한다.

1. Decorators/Mixins

   - 뷰를 실행하기 이전에 추가 로직을 껴 넣고 싶을때,
    ```python
    @login_required
    def fview(req):
        pass

    login_required(fview)
    ``` 

1. View로 들어온 이후
   1. 메서드 분기
   1. get_queryset: (model = document.objects.all())
      1. get(): 단일 오브젝트
      1. all(): 전부
      1. filter(): 필터 후 가져오기
      1. exclude(): 제외
      1. annotate(): 
   1. get_object: (model.objects.filter(), model.objects.get())
   1. get_context_data: 템플릿에 바인딩 할 데이터를 key-value페어로 전송
   1. render_to_response: 템플릿을 순수한 html화 하고, response까지

1. Model
   1. Field
      - 데이터형식과 제약사항을 지정, 입력시 Form태그의 종류와 제약조건도 설정 할 수 있다.
   1. __str__
      - 객체 출력시 출력될 값을 반환
   1. class Meta
      - 객체의 옵션설정 클래스
      - Ordering: 정렬
      - db_table: 저장된 테이블 이름
   1. .save(), .delete(): 저장과 삭제시 사용
      - save(): 데이터베이스에 바로 저장후 끝남
      - save(commit=False): 쿼리만 만들고 실제로 저장은 안함
      - delete(): DB에서는 이미 지워짐, 장고 메모리에서는 생존
      - Document(): 스키마에 따른 객체, 쿼리셋만 생성
      - Document.objects.create(): 객체를 반환하고, 저장까지 완료
   1. 모델을 이용해 DB를 다루려면 QuerySet을 사용
   1. QuerySet을 이용하려면 default manager인 .objects를 이용한다.

1. Template
   1. "context_processors"
      - 모든 템플릿에서 처리해야할, 컨텍스트를 생성하고 전달해준다.
      - 모든 템플릿에서 공통적으로 명시하지 않아도 유효한 컨텍스트를 정의하고 전달한다. ({{user}})
      - 템플릿의 파일명은 INSTALLED_APP의 하부에 있는 templates를 순차적으로 탐색한다.
      - settings.Templates의 dirs를 설정하면, 모든 템플릿폴더에 우선되는 탐색을 지정할 수 있다.(layout.html)
   1. 템플릿 태그
      - {% url 'board:detail' document_id=object.id %}
   1. 템플릿 필터
      - {{object.name}}

1. csrftoken
   1. {%csrf_token%} : 템플릿에 hiddentype input의 value로 전달
   1. {{csrf_token}} : ajax에 값으로 전달할 때
   1. ajax에서 csrf토큰 헤더를 설정하는법
    ```javascript
    xhr.setRequestHeader("X-CSRFToken", csrftoken);
    ```

## TODO
   - contextmanager을 이용한 함수형뷰의 데코레이터