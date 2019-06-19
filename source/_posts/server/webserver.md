---
title: ec2에서 nginx uwsgi로 장고앱 세팅
p: server/webserver
date: 2019-05-22 10:17:35
tags: ['nginx', 'server', 'uwsgi']
---

## webserver

   - 웹서버가 하는 일은 사용자가 네트워크를 통해 요청을 보내면, 요청에 대한 해석과 응답을 돌려주는 역할. 요청을 해석할때, 어떤 리소스를 찾느냐에 대한 처리를 진행, WAS와 통신하요 사용자 요텅을 해성하는 어플에 대한 호출과 콜백을 처리

   1. APACHE
      - 메인 쓰레드가 있고, 하위에 워커등의 방식으로 서브 프로레스를 만들어서 다량의 요청을 처리
   1. NGINX
      - 이벤트 처리방식, 요청이 들어오면 어플리케이션에 요청처리를 부탁하고 본인은 계속 들어온 요청에 대한 대응을 수행한다. Apache에 비해서 적은 리소스를 사용하고 빠른 처리가 가능

## nginx설치
   1. `sudo apt install nginx`
   1. `service nginx status` or `systemctl status nginx`
   1. `ec2-52-78-46-217.ap-northeast-2.compute.amazonaws.com:80` 웹 브라우저 동작 테스트
   1. 설정파일로 프로그램 설정
   1. `sudo cp /etc/nginx/site-available/default /etc/nginx/sites-available`
   1. `vim /etc/nginx/site-available/staticweb`
    ```
    server 80;
    listen [::]:80;

    root /var/www/staticweb;
    
    server_name ec2-52-78-46-217.ap-northeast-2.compute.amazonaws.com; #EC2 인스턴스의 퍼블릭 인스턴스 DNS

    location / {
    }
    ```

   1. 설정파일을 제대로 했다면, Nginx를 재시작 해여한다.
   1. 설정파일이 올바른지 확인하는 법
    
    ```bash
    $:sudo nginx -t
    test is successful
    ```
   1. 해당 설정파일을 Nginx로 연결시켜야 한다. sudo ln -s /etc/nginx/sites-available/staticweb /etc/nginx/sites-enabled/
   1. test
   1. `sudo vim etc/nginx/nginx.conf`
    ```bash
    24 server_name_hash_size 128
    ```
   1. 고정 IP가 필요한 이유는 도메인 연결을 하기 위해서
   1. route53에서 ip주소에 따른 주소 생성하고
   1. IP를 바인딩 하는 순간 기존 퍼블릭 도메인네임 으로 접속하는 것은 끝남
   1. `ssh -i ~/.ssh/aws-ec2-junehan.pen ubuntu@52.78.68.43`
   1. 서비스 한번 리스타트 해주고
   1. 접속이 잘 된다
   1. staticweb의 servername을 route53에 매칭한 주소로 바꿔준다.

## 백업, 재활용을 위한 이미지 생성
   1. 인스턴스 탭에서 이미지 생성
   1. images AMIs에 있음


## 웹서버가 웹 어플을 관리하기 위한 계정 생성
   1. `sudo useradd -g www-data -b /home -m -s /bin/bash django`
       - www-data: nginx설치시 생성된 그룹
       - `-b` : 루트 디렉터리(유저의 홈폴터
       - `-m` : 자신의 계정 명의 
       - `-s` : 로그인 기본쉘
       
   1. `sudo mkdir -p /var/www/django` 
   1. `sudo chown django:www-data /var/www/django`
   1. `sudo usermod -a -G www-data ubuntu` 루트권한 유저 그룹 변경
        - a :add
        - G :group
   1. `sudo chmod g+w /var/www/django`
   1. python manage.py runserver 0:8000
   1. 제대로 출력되지 않으면 ALLOWED_HOST 설정 += '*'  
   1. 80번 포트에 nginx가 req를 받으면, uwsgi를 통해서 8000번 포트로 인계하라는 명령을?
   1. `pip install uwsgi`
        - nginx랑 uwsgi 붙어 있는가?
        - uwsgi랑 django랑 잘 붙어 있는가?
   1. `uwsgi --http :8000 --home /var/www/django/venv/ --chdir /var/www/django/ --module config.uwsgi`
      - uwsgi는 run을 통해 소켓 파일을 생성한다.
   1. `mkdir run logs`
   1. `chown django:`
   1. uwsgi의 실행 프로세스 자동화, 컴퓨터 재시작시 자동 시작 프로세스에 등록하기, 그 시작에 해당 자동화 설정을 포함하기
   
   1. mkdir
   1. uswgi.ini 
      - vacuum = true : 원래 만들어져 있던 파일을 지운다.
      - uswgi.ini외에도 [Service]의 ExecStart에서는 폴더를 기준으로 실행하기 때문에, 설정 파일이 복수로 있다면, 다수의 장고앱을 실행한다.
   1. staticweb 일때만 root가 필요하다i.
   
## 흐름 정리
   1. nginx<->uwsgi site-available/django `restart nginx`
   1. uWSGI uwsgi.service `systemctl deamon-reload`, `restart uwsgi`   
   1. uWSGI <-> djnago uwsgi.ini or django code diff? `uwsgi restart`

## step 24. 장고 소스코드가 있는 폴더의 소유자와 권한을 변경
`sudo chown -R django:www-data /var/www/django`
`sudo chmod -R g+w /var/www/django`

## 위지윅 에디터
- 자바스크립트 라이브러리를 가지고 설정
