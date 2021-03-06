---
title: connect s3, RDS in django
p: django/s3_in_django
date: 2019-05-15 16:47:58
tags: ['django']
---

1. ## awscli설정

   1. awscli 설치하기

    ```bash
    pip install awscli
    ```

   1. awscli 설정하기

    ```bash
    awscli --configure
    ```

   - ~/.aws/credentials 에 자동으로 키값이 들어간 파일을 만들어 준다.

    ```python
    [default]
    aws_access_key_id = *************
    aws_secret_access_key = *****************
    [guest]
    aws_access_key_id = ****************
    aws_secret_access_key = **************
    ```

   1. boto3와 awscli를 이용한, 기본 접근키 비밀 키 설정

    ```python
    # settings.py
    import botocore.session
    AWS_ACCESS_KEY_ID = botocore.session.get_credentials().access_key
    AWS_SECRET_ACCESS_KEY = botocore.session.get_credentials().secret_key
    ```

2. ## django-storages의 폴더 자동 업로딩을 위한 설정

   - 최소 업로드: ACCESS_KEY, SECRET_KEY, 버킷이름, Boto3

   1. STATICFILES_DIRS: Folder dirs to upload as static

    ```python
    #settings.py
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static'),
    ]
    #한 폴더에 static을 모은다
    ```

   1. STATICFILES_STORAGE: meaning S3 resouce on AWS_ACCESS_KEY_ID

    ```python
    STATICFILES_STORAGE = "storages.backends.s3boto3.S3Boto3Storage"
    """to allow collectstatic to automatically put file to bucket"""
    ```

   1. AWS_STORAGE_BUCKET_NAME: Pure S3 bucket name  

    ```python
    AWS_STORAGE_BUCKET_NAME = 'static.oh-mon-lesiles.shop'
    # s3 bucket name (Usually 'static' or 'sources',etc..)
    # but used DNS mapped as bucket name in this case
    ```
   1. STATIC_URL: used in `static 'images/sample.jpg'`

    ```python
    STATIC_URL = f'https://{AWS_STORAGE_BUCKET_NAME}/
    ```
   1. OPTIONALS

    ```python
    AWS_LOCATION = ''
    # prefix prepended to all uploads
    AWS_S3_OBJECT_PARAMETERS = {
   'CacheControl': 'max-age=86400',
    }
    ```

   > STATICFILES_STORAGE가 부트스트래핑 되는 자원에 대한 고유한 저장소라면 DEFAULT_FILE_STORAGE는 모델에서 저장되는 파일에 대한 기본적인 저장소로 연결되도록 django-storages가 처리해준다.

   1. DEFAULT_FILE_STORAGE: settings.py의 StaticStorage를 오버라이딩한 기본 모델의 파일 저장소 세팅
    ```python
    from storages.backends.s3boto3 import S3Boto3Storage

    class MediaStorage(S3Boto3Storage):
        location = ''
        bucket_name = 'media.oh-mon-lesiles.shop'
        file_overwrite = False
        custom_domain = f'http://{bucket_name}/'
    ```
      - STATIC_URL이 settings.py 에서 Static_storage의 엔트포인트라면,
      - custom_domain은 static을 제외한 것들이 statuc_url대신에 가질 수 있는 엔드포인트

