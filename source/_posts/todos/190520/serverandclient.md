---
title: server and client
p: todos/190520/serverandclient
date: 2019-05-20 10:19:37
tags: ['til']
---

## client Request
   1. 브라우저 동작을 통해 보내는 요청
      1. 링크를 클릭해서 해당 주소로 이동하면서 생기는 요청
      1. 주소를 브라우저 주소창에 입력해서 접속
      1. 폼의 제출 버튼을 눌러서 접속
      >  page reload first -> also req

   1. 브라우저의 요청을 흉내내서 보내는 요청
      1. Ajax(Asyncronous javascript and xml request)
        ```javascript
        $.ajax({
            url: "somepage.com/...",
            }
            ).done((data)=>{
            
            })
        ```

   1. Request header
      1. req target
      1. req method
         1. Put : 전체 수정
         1. Delete : 삭제 요청
         1. Fetch : 일부 혹은 전체 데이터의 수정
         1. HEAD : GET으로 받는 정보 중에 헤더 정보만 req
         1. OPTIONS : 해당 URL에 대해 허용하는 메서드 확인 req
      1. http version
      1. header
         1. HOST
         1. User-Agent
         1. Accept
         1. connection
         1. content-type
      1. Body
   
   1. Django
      1. Settings.py middleware
      1. URLCONF동작 : url을 파싱하고 동작시켜야 할 뷰 연결
        ```python
        path('admin/', func),
        path('update/<int:pk>/',func), # int:pk : namedGroup
        namedGroup은 뷰에 매개변수로 전달
        
        def update(req, pk):
            pass
        class DUpdate(UpdateView):
            def get(self, req, *args, **kwargs):
                pk = kwargs['pk']
        ```
      1. view
         -  with model: 해당 뷰에 유효한, 모델에 대해서 ORM을 이용해 추상화하고 데이터베이스 종류에 맞춰서 쿼리문 전달
            - ORM을 사용하면 데이터베이스 서비스 종속성이 없다.(SQL Query -> mysql, ms-sql, postgresql ...)
            - QuerySet: django에서 ORM을 다루기 위한 스킬
         - with Template : 
      
      1. bootstrap
         1. settings.py
            1. ROOT_URLCONF = `config.urls`
            1. WSGI_APPLICATION = 'config.wsgi.application'
    ```python
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
    application = get_wsgi_application()
    ```
   1. Infra
      - Tier
         - 몇 개층으로 서버 인프라를 구성 했느냐?
         1. 1: 로컬 프로그램
         1. 2: 클라이언트 - 데이터
         1. 3: 클라이언트 - 비지니스 로직 - 데이터 (웹앱까지 유효하게 구성했을때)
      - 스케일링
         - 요청이나 수행의 분담을 분할
         1. Up-scaling : 
         1. Out-scaling : 데이터의 동기화 문제 발생(racecondition)
         - 동기화 문제에 대한 해결
            1. 데이터베이스 중앙화
            1. 파일 서버 중앙화
         - 스케일링은 자동화도 가능하다. 하지만 반 자동화가 일반적
         - WarmUP : 사용자가 늘거라고 예상되는 시점이 있을 때, 미리 스케일링
         - GIB : 20GB라고 하면 실제 OS에서 운영 가능한 양은 그것보다 적다. GIB는 그것보다 크게 잡아서 실제 20GB를 보장하는 것을 얘기한다.
         - RDS region : 실제 서비스하는 지역
         - RDS Available zone : 2대 정도는 둔다.(전환 가능한)
         - 성능을 위해서 READONLYDB(Read replica)를 둔다.
         - 멀티 리전 : 실제 다른 국가에 인프라 구축, CDN

      - database
         1. RDS instance 생성
         2. settings.DATABASES 규정
        ```python
        # backends는 장고와 데이터베이스 어뎁터 사이를 연결
        # psycopg2-binary: 어댑터 - 드라이버 -> 실제로 데이터베이스에 접속해서 명령을 수행
        DATABASES = {
            'default' = {
                'ENGINE' : 'djnago.db.backends.postgresql_psycopg2',
                'NAME' : 'oh_mon_lesiles_db',
                'USER' : 'junehan',
                'PASSWORD' : '0000',
                'HOST' : 'wps10-dbsetting.cuddiucrfzmn.ap-northeast-2.rds.amazonaws.com',
                'POST' : '5432',
            }
        }
        ```
         3. migrate
            - `python manage.py makemigrations` : 변경사항이 있을때, 
            - `python manage.py migrate` : 총 제작한 쿼리로 실제 DB에 반영
            - `python manage.py createsuperuser` : 장고에서 DB에 대한 최종관리자 권한 생성

         4. 스케일링이 필요한 시점을 대비한 초기세팅
            > 웹 서비스의 입출력 병목이 발생 했을때, 스케일링.  
             스케일링은 동기화 문제를 발생시킬 수 있기 때문에,  
             DB의 경우 특이한 구조로 스케일링 진행 (Master/Slave방식)
            - master DB : 기존에 저장만 가능하고, 저장되면 Read서버로 변경내용을 전달
            - Readonly DB : 읽기만 가능.  
  
            > Master/slave 구조를 구현하기 위해 **DB Routing**을 설정한다.  
            특정 모델이 별도 DB를 이용하는 경우를 설정한다.
            
        5. DB Scaling
           - AWS RDS > DATABSE > ACTION > 읽기전용DB생성
           - MASTER에 대한 아웃 스케일링이 필요할 때, AURORADB

      - Storage
         - 개요
            1. 파일 중앙화: 파일의 동기화 문제를 해결
            2. 스토리지 용량 무제한: 미디어 중심 서비스의 경우
            3. 1파일당 용량 2TB: 대형 미디어
            4. 파일의 안정성: 파일이 깨지지 않음을 보장하는 편
            5. django 3rd-party라이브러리와 호환성이 높다
            
         - Static and media bucket
            - media url을 직접 사용하여 유저가 업로드
            - 버킷을 생성(리소스 인프라)을 완료하였다면, AWS API에 맞춰야 함
            - IAM S3에 맞춰서 생성
            - settings.py
            ```python
            AWS_S3_OBJECT_PARAMETERS = { # 브라우저가 해당 파일에 접속 했을 때 나타나는 파라미터 값
                'CacheControl' : '' # 자주 바꾸는게 아니라면, 한번 다운 받으면 계속 유지되도록
                }
            AWS_DEFAULT_ACL = 'public-read'
            AWS_S3_SECURE_URLS = False
            ```
            ```python
            # cofnig.s3media.py
            class MedaiStorage(S3Boto3Storage):
                location = ''
                bucket_name = 'media.sample.actingprogrammer.io'
                custom_domain = ''
            ```
            - CDN을 사용하는 이유, https에서 문제가..

