---
title: postgresql 접속하기
p: sql/postgresql_basic
date: 2019-05-14 19:03:54
tags: [sql, postgresql]
---

## 1.초기 실행 명세

   1. 지정된 유저로 로그인

        ```bash
        $ psql -h localhost -d testdb -U testuser -W
        postgres=#
        ```

   2. 최초 슈퍼계정으로 로그인 (비밀번호 미설정?)

        ```bash
        $ sudo -u postgres psql
        ```

   3. 슈퍼계정 비밀번호 변경

        ```sql
        ALTER USER postgres with encrypted password '******';
        ```

   3. 로그인 후 기본 명령어

      - `\q`  : 종료
      - `\du` : 유저 목록 확인
      - `\l`  : DB확인
      - `\dt` : 테이블 확인
      - `\c`  : 접속 정보 확인
        
        ```sql
        <!-- 현재 로그인 유저 확인 -->
        $ SELECT current_user;
        <!-- 현재 데이터베이스 확인 -->
        $ SELECT current_database();
        ```

   4. postgreql서비스 제어

        ```bash
        $ service postgreql
        Usage: /etc/init.d/postgresql {start|stop|restart|reload|force-reload|status} [verision ..]
        
        $ /etc/init.d/postgresql status
        $ sudo service postgresql [status, restart, stop, start]
        ```

## 2.DB User and DATABASE

   - CREATE and DROP DATABASE

        ```sql
        CREATE DATABASE test;

        DROP DATABASE test;

        CREATE TABLE test(
            id int,
            name varchar(20)
        );

        DROP TABLE test;
        ```

   - add user and DROP user
        ```sql
        $ DROP ROLE testuser;
        DROP ROLE
        $ CREATE ROLE testuser;
        CREATE ROLE
        ```

   - Change ROLE password
        ```sql
        $ ALTER ROLE guest LOGIN password 0000;
        ```
   - give PRIVILEGE
        ```sql
        $ GRANT ALL PRIVILEGES ON dbname TO testuser
        ```

## 3.Connect to aws-rds psql

   - connect through psql client

        ```bash
        $ psql
        --host=oh-my-lesiles-db.cuddiucrfzmn.ap-northeast-2.rds.amazonaws.com
        --port=5432
        --username=junehan
        --password
        --dbname=oh_my_lesiles_db
        
        Password for user junehan:
        ```

   - connect and log through sqlarchemy

        ```python
        import logging
        logging.getLogger('sqlalchemy.dialects.postgresql').setLevel(logging.INFO)
        ```
   
   - create orm engine
        ```python
        engine = create_engine(
        'postgresql+psycopg2://{id}:{password}@{host}/{dbname}',
        use_native_hstore=False,
        )
        ```

   - psycopg2
      - psycopg2 DBAPI는 내부적으로 serialization을 HSTORE타입을 사용하는 extension을 포함한다.