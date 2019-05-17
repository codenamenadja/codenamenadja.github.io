---
title: post
p: sql/postgresql_basic
date: 2019-05-14 19:03:54
tags: ['sql', 'postgresql']
---

## 1.login to database
- login
```bash
>>> sudo -u postgres psql
postgres=#
```
- Listig DATABASE
```bash
>>> \l
List of databases ...
```

- change Root password
```sql
ALTER USER postgres with encrypted password '******';
```

- quit and restart
```bash
>>> \q
quit
>>> sudo /etc/init.d/postgresql restart
>>> psql --username=postgres --host=127.0.0.1
ENTER PASSWORD...:
```
***
## 2.DB User and DATABASE

- CREATE and DROP DATABASE
```sql
>>> CREATE DATABASE test;
CREATE DATABASE
>>> DROP DATABASE test;
DROP DATABASE
```

- see user list of db
```sql
\du
```
- add user and DROP user
```sql
>>> DROP ROLE testuser;
DROP ROLE
>>> CREATE ROLE testuser;
CREATE ROLE
```
- Change ROLE password
    - make no rights about DATABASE
    - but make login avails to specific user(guest)
```sql
>>> ALTER ROLE guest LOGIN password 0000;
```
## 3.Connect to aws-rds psql
- connect through psql client
```bash
>>> psql --host=oh-my-lesiles-db.cuddiucrfzmn.ap-northeast-2.rds.amazonaws.com --port=5432 --username=junehan --password --dbname=oh_my_lesiles_db
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
- psycopg2
    - psycopg2 DBAPI는 내부적으로 serialization을 HSTORE타입을 사용하는 extension을 포함한다.
    - 
```
