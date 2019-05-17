---
title: connect s3, RDS in django
p: python/s3_in_django
date: 2019-05-15 16:47:58
tags: django s3 boto3 awscli
---

1. ## awscli설정
    1. awscli 설치하기 `pip install awscli`
    1. `awscli --configure` ---->  in `~/.aws/credential`,  save key-pair as `[default]`
***
2. ## boto3와 awscli의 긴밀함을 이용
    1. 기본 접근키 비밀 키 설정
    ```python
    # settings.py
    import botocore.session

    AWS_ACCESS_KEY_ID = botocore.session.get_session().get_credentials().access_key
    AWS_SECRET_ACCESS_KEY = botocore.session.get_session().get_credentials().secret_key
    ```
    2. django-storages의 폴더 자동 업로딩을 위한 설정
        > 기본적으로 ACCESS_KEY, SECRET_KEY와 버킷이름이 있으면 올리는것은 BOTO3 만으로 가능하는 것을 명심하라.
        1. STATICFILES_DIRS: Folder dirs to upload as static
        ```python    
        # settings.py
        STATICFILES_DIRS = [
            os.path.join(BASE_DIR, 'static'),
        ]
        # 한 폴더에 static을 모은다
        ```
        2. STATICFILES_STORAGE: meaning S3 resouce on AWS_ACCESS_KEY_ID
        ```python
        STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
        # to allow collectstatic to automatically put file to bucket
        ```
        3. AWS_STORAGE_BUCKET_NAME: Pure S3 bucket name
        ```python
        AWS_STORAGE_BUCKET_NAME = 'static.oh-mon-lesiles.shop'
        # s3 bucket name (Usually 'static' or 'sources',etc..)
        # but used DNS mapped as bucket name in this case
        ```
        4. STATIC_URL: used in `{%static 'images/sample.jpg'%}`
        ```python
        STATIC_URL = f'https://{AWS_STORAGE_BUCKET_NAME}/
        ```
        5. OPTIONALS
        ```python
        AWS_LOCATION = ''
        # prefix prepended to all uploads

        AWS_S3_OBJECT_PARAMETERS = {
            'CacheControl': 'max-age=86400',
        }
        ```
    3. media라는 별도의 bucket으로 업로드
        1. upload를 위한 설정 클래스 오버라이딩
        
        ```python
        # settings.py와 동일 레벨, config/storage_backends.py
        from storages.backends.s3boto3 import S3Boto3Storage

        class MediaStorage(S3Boto3Storage):
            location = '' # AWS_LOCATION과 동일하게 작용
            bucket_name = 'media.oh-mon-lesiles.shop'
            # AWS_STORAGE_BUCKET_NAME의 값을 settings.py에서 읽어오도록 설정
            file_overwrite = False
        ```
        
        2. static의 기존 업로드 클래스와 다른 클래스 지정
        
        ```python
        # settings.py
        DEFAULT_FILE_STORAGE = 'config.storages_backends.MediaStorage'
        ```
        
        3. upload 하는 모델 만들기

        ```python
        image = models.ImageField(upload_to='media/board_images/%Y/%m/%d')
        # root 디렉터리에서media위로 해당 폴더와 이름값을 그대로 유지한다.
        # board_images에 자료분류는 연/월/일 기준으로 폴더를 구성하고
        # 네이밍은, 글 번호를 기준으로 하는 것이 좋을 것 같다.33-1.jpg 면 33번에 1번째 이미지라는 표시이다.
        ```
        
        4. upload_to에 직접 URL을 매핑하기?

        ```python
        image = models.ImageField(upload_to='http://media.oh-mon-lesiles.com/board_images/%Y/%m/%d')
        ```
        ```html
        {%for result in results%}
        <img src='{{result.url}}'>
        {%endfor%}
        ```
        > result.url은 upload_to에 확장자를 포함한 파일네임이 적용된다.

