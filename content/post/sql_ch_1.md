---
title: "HEAD FIRST SQL 챕터 1장 요약정리"
date: 2020-12-01T20:47:37+09:00
draft: False
tags: ["SQL"]
categories: ["SQL"]
---

MYSQL 문법을 복습할 겸해서 HEAD FIRST SQL 책을 요약정리 합니다.

## 준비사항

실제로 SQL 문을 치기위해 MY SQL과 유사한 MARIA DB를 도커를 통해 설치해 줍니다. 도커를 설치하는 방법은 [도커 홈페이지](https://docs.docker.com/engine/install/)를 참조하세요. 이제 MARIA D를 설치할 차례입니다.

터미널이나 CMD에서 아래의 명령어를 통해 MARIA DB를 설치합니다.
`docker pull mariadb`
설치가 완료되면 아래의 코드로 maria db를 실행시킬 수 있습니다.

```bash
docker run --name mariadb -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password mariadb

docker exec -it mariadb /bin/bash

mysql -u root -p
```

위의 첫 번째 코드에서 password 부분은 MARIA DB를 접속할 때 필요한 비밀번호로 편한 것으로 설정해주시면 됩니다. 여기서는 password로 설정했습니다. 이어서 세 번째 코드를 치고나면 이제 설정한 패스워드를 입력해 접속할 수 있습니다. MARIA DB에 접속하면 아래와 같은 화면을 보실 수 있습니다.

![마리아 DB 접속화면](https://user-images.githubusercontent.com/19144813/100723129-bec49580-3404-11eb-8f38-4023a8921af6.png)

## 1장 데이터와 테이블

데이터는 테이블 형태의 열과 행으로 나타낼 수 있다. 데이터베이스는 데이터의 저장소인 테이블들을 저장하는 것입니다. 데이터베이스에서 필요한 데이터를 요청하는 작업은 쿼리라고 합니다. 쿼리는 데이터를 요청하는 것이기에 데이터베이스 단위가 아닌 테이블 단위에서 이루어집니다. 테이블 형태의 열과 행이란 우리가 엑셀에서 보는 표와 같은 것이다. 테이블에서 열은 데이터의 속성정보나 카테고리를 의미한다. 행은 데이터가 나타내는 한 객체에 대한 속성을 나타내는 여러개의 열이다. 간단히 보면 테이블에서 세로 줄은 데이터의 속성을 의미하는 열이며, 맨위의 세로줄 밑에 가로줄들은 행을 의미합니다.

이제 데이터베이스를 직접 만들어 봅시다. 가독성과 구분을 위해 명령어는 대문자로 표기합니다.

```sql
CREATE DATABASE gregs_list;
USE gregs_list;
```

- `CREATE DATABASE`는 사용할 데이터베이스를 만드는 명령어
- `USE`는 만든 데이터베이스를 사용하는 명령어

실행 결과는 아래와 같습니다.

```bash
MariaDB [(none)]> CREATE DATABASE gregs_list;
Query OK, 1 row affected (0.007 sec)

MariaDB [(none)]> USE gregs_list;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [gregs_list]>
```

SQL 문은 명령어 끝에 ;를 붙어 명령이 끝났음을 알려줍니다. 잊지 말고 붙이자 !

### 테이블 조작하기

`CREATE TABLE` 명령어는 데이터베이스 안에 테이블을 만드는 명령어입니다. 테이블을 만들어야 데이터베이스 안에 데이터를 저장할 수 있습니다. 먼저 테이블을 구성하는 데이터 타입들에 대해 알아봅시다.

`VARCHAR`,`CHAR` or `CHARACTER`, `DATETIME` or `TIMESTAMP`, `TIME`, `DATE`, `BLOB`, `DEC` or `DECIMAL`, `INT` or `INTEGER` 등이 있습니다.
`VARCHAR()` 변하는 문자형으로 텍스트를 저장하는데 이용됩니다. () 안 숫자는 그 숫자 자리까지 저장할 수 있다는 것을 의미합니다. 단 한글의 경우 2byte이기에 1byte 괄호 안 숫자의 그 절반까지만 입력가능합니다. 총 255자리 까지 저장 가능합니다.
`CHAR` or `CHARACTER` 데이터가 정해진 길이인 문자열 저장 형식입니다.
`DATETIME` or `TIMESTAMP` 날짜와 시간을 저장하는 형식입니다.
`TIME` 시간을 저장하는 형식입니다.
`BLOB` 큰 덩어리의 문자 데이터를 저장하는 형식입니다.
`DEC` or `DECIMAL` 십진 자리수를 저장하는 형식입니다.
`INT` or `INTEGER` 정수를 저장하는 형식입니다.

직접 테이블을 만들어 봅시다.

```sql
CREATE TABLE doughnut_list(
    doughnut_name VARCHAR(10),
    doughnut_type VARCHAR(6)
);
```

- `CREATE TABLE`는 테이블을 만드는 명령어

여러개의 열을 지닌 테이블을 만들 수도 있습니다.

```sql
CREATE TABLE my_contacts(
    last_name VARCHAR(30),
    first_name VARCHAR(20),
    email VARCHAR(50),
    birthday Date,
    profession VARCHAR(50),
    location VARCHAR(50),
    status VARCHAR(20),
    interests VARCHAR(100),
    seeking VARCHAR(100)
);
```

이렇게 만들어진 테이블의 구조정보를 확인하려면 `DESC` 명령어를 이용합니다.

```sql
DESC my_contacts;
```

- `DESC`는 생성된 테이블의 정보를 보는 명령어

실행 결과는 아래와 같습니다. 각 열에 대한 정보가 출력되는데 열 이름, 열의 데이터 타입, NULL 값 여부 등이 표시됩니다.

```bash
MariaDB [gregs_list]> DESC my_contacts;
+------------+--------------+------+-----+---------+-------+
| Field      | Type         | Null | Key | Default | Extra |
+------------+--------------+------+-----+---------+-------+
| last_name  | varchar(30)  | YES  |     | NULL    |       |
| first_name | varchar(20)  | YES  |     | NULL    |       |
| email      | varchar(50)  | YES  |     | NULL    |       |
| birthday   | date         | YES  |     | NULL    |       |
| profession | varchar(50)  | YES  |     | NULL    |       |
| location   | varchar(50)  | YES  |     | NULL    |       |
| status     | varchar(20)  | YES  |     | NULL    |       |
| interests  | varchar(100) | YES  |     | NULL    |       |
| seeking    | varchar(100) | YES  |     | NULL    |       |
+------------+--------------+------+-----+---------+-------+
9 rows in set (0.002 sec)
```

기존에 만든 테이블이 맘에 들지 않거나 필요가 없어졌을 때는 `DROP TABLE` 명령어를 사용합니다.

```sql
DROP TABLE my_contacts;
```

- `DROP TABLE`은 생성된 테이블을 지우는 명령어

테이블을 지우게 되면 그안에 있는 데이터도 모두 사라지게 됩니다. 아직은 데이터를 넣지 않아서 사라지는 데이터가 없지만 이후 데이터를 넣은 경우 드롭 테이블문은 조심스럽게 사용해야 합니다.

다시 my_contacts 테이블을 만들어줍니다. 이번에는 gender라는 열이 추가 되었습니다.

```sql
CREATE TABLE my_contacts(
    last_name VARCHAR(30),
    first_name VARCHAR(20),
    email VARCHAR(50),
    gender CHAR(1),
    birthday Date,
    profession VARCHAR(50),
    location VARCHAR(50),
    status VARCHAR(20),
    interests VARCHAR(100),
    seeking VARCHAR(100)
);
```

그렇다면 테이블에 데이터를 넣는 방법은 무엇일까요 ? INSERT INTO를 이용하면 됩니다. 아래와 같이 열이름을 지정해주고 각 열의 순서에 맞게 데이터의 값을 입력해줍니다. 그러면 데이터가 테이블에 삽입됩니다.

```sql
INSERT INTO my_contacts(
    last_name, first_name, email, gender, birthday, profession, location, status, interests, seeking
)
VALUES (
    'Anderson', 'Jillian', 'jill_anderson@breakneckpizza.com', 'F', '1980-09-05', 'Technical Writer', 'Palo Alto, CA', 'Single', 'Kayaking, Reptiles', 'Relationship, Friends'
);
```

- `INSERT INTO`문은 데이터를 테이블에 넣는 명령어

만약 열에 해당하는 값을 빼먹고 입력하지 않았다면, 데이터가 입력이 실행되지 않고 에러 `ERROR 1136 (21S01): Column count doesn't match value count at row 1`를 반환합니다.

모든 열에 해당하는 데이터를 넣을 때에는 모든 열이름을 적어줄 필요 없이 생략해도 가능합니다. 단 이 경우에는 반드시 모든 열에 해당하는 값을 넣어주어야 합니다. 그렇지 않으면 에러가 발생합니다.

또한 아래와 같이 특정 열에만 값을 입력할 수 있습니다.

```sql
INSERT INTO my_contacts(
    first_name, email,  profession, location
)
VALUES (
    'Pat', 'patpost@breakneckpizza.com', 'Postal Worker', 'Princeton, NJ'
);
```

이렇게 입력된 데이터를 확인하는 방법은 SELECT 문을 이용하면 됩니다. 자세한 방법은 2장에서 첨언하겠습니다.

```sql
SELECT * FROM my_contacts;
```

```bash

MariaDB [gregs_list]> SELECT * FROM my_contacts;
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
| last_name | first_name | email                            | gender | birthday   | profession       | location      | status | interests          | seeking               |
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
| Anderson  | Jillian    | jill_anderson@breakneckpizza.com | F      | 1980-09-05 | Technical Writer | Palo Alto, CA | Single | Kayaking, Reptiles | Relationship, Friends |
| NULL      | Pat        | patpost@breakneckpizza.com       | NULL   | NULL       | Postal Worker    | Princeton, NJ | NULL   | NULL               | NULL                  |
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
2 rows in set (0.001 sec)

```

이렇게 일부 열에만 데이터 값을 입력한 경우 NULL 값이 발생합니다. 이 값들을 제어하기 위해서는 다양한 방법이 있습니다. 먼저 테이블을 만들 때 NULL 값이 발생하지 않토록 방지하는 방법이 있습니다. 위에서 만든 도넛 리스트 테이블을 지웁니다. 그리고 NULL 값을 제어하여 다시 만들어 줍니다.

```sql
DROP TABLE doughnut_list;
```

```sql
CREATE TABLE doughnut_list(
    doughnut_name VARCHAR(10) NOT NULL,
    doughnut_type VARCHAR(6) NOT NULL,
    doughnut_cost DEC(3,2) NOT NULL DEFAULT 1.00
);
```

CREATE TABLE 문에서 변수 타입 옆에 NOT NULL을 지정해주고 기본 값을 지정해주는 방식으로 NULL 값을 제어할 수 있습니다. 기본 값을 지정해주면 값을 모를때 자동으로 기본 값으로 채워지게 됩니다. 따라서 NULL 값이 발생하지 않습니다.

새로운 테이블에 데이터를 넣어봅시다. 모든 데이터를 알고 있는 경우 열이름들을 생략하고 데이터를 넣을 수 있습니다.

```sql

INSERT INTO doughnut_list
VALUES('Blooberry','filled',2.00),
('Appleblush','filled',1.40);
```

하지만 값을 몰라 기본 값으로 대체할 경우 아래와 같이 값을 아는 두 열에만 데이터를 넣어주어야 합니다.

```sql
INSERT INTO doughnut_list (
    doughnut_name,doughnut_type
)
VALUES
('Cinamondo', 'ring'),
('Rockstar','cruler');
```

최종적으로 완성된 테이블의 모습은 아래와 같습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM doughnut_list;
+---------------+---------------+---------------+
| doughnut_name | doughnut_type | doughnut_cost |
+---------------+---------------+---------------+
| Blooberry     | filled        |          2.00 |
| Appleblush    | filled        |          1.40 |
| Cinamondo     | ring          |          1.00 |
| Rockstar      | cruler        |          1.00 |
+---------------+---------------+---------------+
4 rows in set (0.001 sec)

```
