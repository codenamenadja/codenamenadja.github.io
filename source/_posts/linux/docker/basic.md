---
title: docker basic
p: linux/docker/basic
date: 2019-06-20 12:59:09
tags: ['docker']
---


## Index

[index]: #index

> 1. [DOCKER CE 설치 이후][i1]
>    1. [DOCKER][i1-1]
> 1. [sample][i2]
>    1. [another 출력기능][i2-1]
> 1. [간단한 구현][i3]
> 1. [나아가서 구현][i4]
> 1. [마치며][i5]

## 이미지_생성하기

[i1]: #이미지_생성하기

1. 우분투 이미지 다운로드

    ```bash
    docker run ubuntu:
    ```

   1. sd
      1. sds

## 파이썬_환경_구성

[i1-1]: #파이썬_환경_구성


```bash
ADD ./conf/uwsgi/uwsgi.ini /var/www/django/ini/uwsgi.ini
```

[index][index]

## 폴더_마운팅

[i2]: #폴더_마운팅

```bash
docker container run --rm -it -p 8080:80 -v /Users/junehan/Desktop/docker_test/code:/var/www/django/code django:some_tag
```

```
[uwsgi]
chdir= $(BASE_DIR)/code
```

### another_출력기능

[i2-1]: #another_출력기능

```python
def another():
    print('another')
```

## 백그라운드

[i3]: #간단한_구현



## 나아가서_구현

[i4]: #나아가서_구현

## 마치며

[i5]: #마치며