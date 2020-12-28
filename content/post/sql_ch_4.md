---
title: "좋은 테이블 설계(정규화)와 기본키"
date: 2020-12-28T19:14:52+09:00
draft: False
tags: ["SQL"]
categories: ["SQL"]
---

## 4장

이 글은 MYSQL 문법을 복습할 겸해서 [HEAD FIRST SQL 책](http://www.yes24.com/Product/Goods/2922303?OzSrank=2)을 요약정리 한 것입니다.

3장에서는 테이블의 데이터를 지우거나(DELETE) 변경(UPDATE)하는 방법에 대해 알아 봤습니다. 관련된 sql 문법에 대해 다시 확인하고 싶으시다면 [링크](https://choosunsick.github.io/post/sql_ch_3/)를 참조해 주시기 바랍니다. 데이터를 변경하거나 지우는 일은 중요한 일입니다. 그러나 처음부터 그런일이 없게 테이블을 잘 만드는 방법을 아는 것은 더 중요하다. 이번 장에서는 그 방법에 대해 알아보자.

테이블을 만들 때는 테이블의 사용 목적을 고려해야 한다. 사용 목적에 따라 같은 자료를 보고도 서로 다른 테이블이 만들어질 수 있습니다. 물고기의 정보가 담긴 데이터가 있다고 해봅시다. 이때 같은 물고기의 정보를 가지고 학문적 접근을 하는 경우와 낚시 기록이 중요한 경우 서로 만들어지는 테이블이 달라질 수 있습니다.

먼저 학문적으로 접근할 때는 물고기의 종 정보가 중요한 정보가 됩니다. 즉 아래와 같이 species 열이 필요하다.

```sql
CREATE TABLE fish_info(common VARCHAR(30), species VARCHAR(30), location VARCHAR(30), weight VARCHAR(30));
```

```sql
INSERT INTO fish_info  
VALUES
('bass, largemouth', 'M. sakmoides', 'Montgomery Lake, GA', '22 lb 4 oz'),
('walleye', 'S. vitreus', 'Old Hickory Lake, TN', '25 lb 0 oz'),
('trout, cutthroat', 'O. Clarki', 'Pyramid Lake, NV', '41 lb 0 oz'),
('perch, yellow', 'P. Flavescens', 'Boredentown, NJ', '4 lb 3 oz'),
('bluegill', 'L. Macrochirus', 'Ketana Lake, AL', '4 lb 13 oz'),
('gar, longnose', 'L. Osseus', 'Trinity River, TX', '50 lb 5 oz'),
('crappie, white', 'P. annularis', 'Enid Dam, MS', '5 lb 3 oz'),
('pickerel, grass', 'E. americanus', 'Dewart Lake, IN', '1 lb 0 oz'),
('goldfish', 'C. auratus', 'Lake Hodges, CA', '6 lb 10 oz'),
('salmon, chinook', 'O. Tshawytscha', 'Kenai River, AK', '97 lb 4 oz');

```

만들어진 테이블의 결과는 아래와 같다.

```bash
MariaDB [gregs_list]> SELECT * FROM fish_info;
+------------------+----------------+----------------------+------------+
| common           | species        | location             | weight     |
+------------------+----------------+----------------------+------------+
| bass, largemouth | M. sakmoides   | Montgomery Lake, GA  | 22 lb 4 oz |
| walleye          | S. vitreus     | Old Hickory Lake, TN | 25 lb 0 oz |
| trout, cutthroat | O. Clarki      | Pyramid Lake, NV     | 41 lb 0 oz |
| perch, yellow    | P. Flavescens  | Boredentown, NJ      | 4 lb 3 oz  |
| bluegill         | L. Macrochirus | Ketana Lake, AL      | 4 lb 13 oz |
| gar, longnose    | L. Osseus      | Trinity River, TX    | 50 lb 5 oz |
| crappie, white   | P. annularis   | Enid Dam, MS         | 5 lb 3 oz  |
| pickerel, grass  | E. americanus  | Dewart Lake, IN      | 1 lb 0 oz  |
| goldfish         | C. auratus     | Lake Hodges, CA      | 6 lb 10 oz |
| salmon, chinook  | O. Tshawytscha | Kenai River, AK      | 97 lb 4 oz |
+------------------+----------------+----------------------+------------+
10 rows in set (0.001 sec)

```

반면에 물고기를 낚은 기록이 중요한 경우 물고기의 종에 대한 정보는 필요가 없다 대신에 잡은 사람의 이름과 잡은 날짜가 중요하다.

```sql
CREATE TABLE fish_records(first_name VARCHAR(30), last_name VARCHAR(30), common VARCHAR(30), location VARCHAR(30),state VARCHAR(2), weight VARCHAR(30), date VARCHAR(30));
```

```sql
INSERT INTO fish_records  
VALUES
('George', 'Perry', 'bass, largemouth', 'Montgomery Lake', 'GA', '22 lb 4 oz', '6/2/1932'),
('Mabry', 'Harper', 'walleye', 'Old Hickory Lake', 'TN', '25 lb 0 oz', '8/2/1960'),
('John', 'Skimmerhorn', 'trout, cutthroat', 'Pyramid Lake', 'NV', '41 lb 0 oz', '12/1/1925'),
('C.C.', 'Abbot', 'perch, yellow', 'Boredentown', 'NJ', '4 lb 3 oz', '5/1/1865'),
('T.S.', 'Hudson', 'bluegill', 'Ketana Lake', 'AL', '4 lb 13 oz', '4/9/1950'),
('Townsend', 'Miller', 'gar, longnose', 'Trinity River', 'TX', '50 lb 5 oz', '7/30/1954'),
('Fred', 'Bright', 'crappie, white', 'Enid Dam', 'MS', '5 lb 3 oz', '7/31/1957'),
('Mike', 'Berg', 'pickerel, grass', 'Dewart Lake', 'IN', '1 lb 0 oz', '6/9/1990'),
('Flprentino', 'Abena', 'goldfish', 'Lake Hodges', 'CA', '6 lb 10 oz', '4/17/1996'),
('Les', 'Anderson', 'salmon, chinook', 'Kenai River', 'AK', '97 lb 4 oz', '5/17/1985');
```

만들어진 테이블에 종 정보 대신 잡은 날짜와 잡은 사람에 대한 정보가 들어가 있다.

```bash
MariaDB [gregs_list]> SELECT * FROM fish_records;
+------------+-------------+------------------+------------------+-------+------------+-----------+
| first_name | last_name   | common           | location         | state | weight     | date      |
+------------+-------------+------------------+------------------+-------+------------+-----------+
| George     | Perry       | bass, largemouth | Montgomery Lake  | GA    | 22 lb 4 oz | 6/2/1932  |
| Mabry      | Harper      | walleye          | Old Hickory Lake | TN    | 25 lb 0 oz | 8/2/1960  |
| John       | Skimmerhorn | trout, cutthroat | Pyramid Lake     | NV    | 41 lb 0 oz | 12/1/1925 |
| C.C.       | Abbot       | perch, yellow    | Boredentown      | NJ    | 4 lb 3 oz  | 5/1/1865  |
| T.S.       | Hudson      | bluegill         | Ketana Lake      | AL    | 4 lb 13 oz | 4/9/1950  |
| Townsend   | Miller      | gar, longnose    | Trinity River    | TX    | 50 lb 5 oz | 7/30/1954 |
| Fred       | Bright      | crappie, white   | Enid Dam         | MS    | 5 lb 3 oz  | 7/31/1957 |
| Mike       | Berg        | pickerel, grass  | Dewart Lake      | IN    | 1 lb 0 oz  | 6/9/1990  |
| Flprentino | Abena       | goldfish         | Lake Hodges      | CA    | 6 lb 10 oz | 4/17/1996 |
| Les        | Anderson    | salmon, chinook  | Kenai River      | AK    | 97 lb 4 oz | 5/17/1985 |
+------------+-------------+------------------+------------------+-------+------------+-----------+
10 rows in set (0.001 sec)
```

목적이 다르기에 만들어진 테이블에 대한 같은 목적에 대한 쿼리 방법도 달라진다. 예를 들어 NJ 주에서 잡힌 물고기에 대한 정보를 얻고 싶다고 할 때 주(state) 라는 열이 있는 fish_records 테이블에는 주를 통해 쿼리를 할 수 있다. 반면 location 열에 주 정보가 있는 fish_info 테이블에서는 만약 NJ라는 글자가 들어간 장소가 있다면 쿼리 결과가 달라질 수 있다. 아래와 같이 같은 목적에 대해 쿼리 방법이 다른 것을 확인할 수 있다.

```sql
SELECT * FROM fish_info WHERE location LIKE '%NJ';
SELECT * FROM fish_records WHERE state='NJ';
```

```bash
MariaDB [fish_list]> SELECT * FROM fish_info WHERE location LIKE '%NJ';
+---------------+---------------+-----------------+-----------+
| common        | species       | location        | weight    |
+---------------+---------------+-----------------+-----------+
| perch, yellow | P. Flavescens | Boredentown, NJ | 4 lb 3 oz |
+---------------+---------------+-----------------+-----------+
1 row in set (0.010 sec)
```

```bash
MariaDB [fish_list]> SELECT * FROM fish_records WHERE state='NJ';
+------------+-----------+---------------+-------------+-------+-----------+----------+
| first_name | last_name | common        | location    | state | weight    | date     |
+------------+-----------+---------------+-------------+-------+-----------+----------+
| C.C.       | Abbot     | perch, yellow | Boredentown | NJ    | 4 lb 3 oz | 5/1/1865 |
+------------+-----------+---------------+-------------+-------+-----------+----------+
1 row in set (0.006 sec)
```

SQL은 관계형 데이터 베이스 관리를 위한 언어이다. 관계형이란 말은 테이블을 설계할 때 여러 테이블의 열들이 서로 어떠한 연관이 있는지를 고려해야한다는 말이다. 테이블 내의 열들에 관계에 대해서 고려하면서 테이블을 설계할 때는 3가지 과정을 거친다.

1. 테이블로 표현하려는 것을 선택한다. - 테이블을 만드는 이유
2. 테이블을 사용하여 얻어야 하는 정보들의 목록을 정리한다. - 테이블에 담겨야 할 내용 정리
3. 테이블을 정리한 정보들의 조각으로 나누어 구성한다. - 테이블의 열을 어떻게 구성할지에 대한 고민

데이터를 조각으로 나누어 구성한다는 말은 테이블을 만드는 이유에 적합하도록 데이터를 나누는 작업을 의미하며 데이터가 충분히 열로 잘 쪼개져 있는 것을 데이터가 원자적이다라고 표현한다.

## 원자적 데이터

원자적 데이터란 데이터가 쪼개질 수 없는 가장 작은 부분들로 이루어졌다는 의미이다. 원자적 데이터를 만들기 위한 방법은 다음과 같다.

- 원자적 데이터의 규칙

규칙1: 원자적 데이터로 구성된 열은 그 열에 같은 타입의 데이터 여러개를 가질 수 없다.

규칙2: 원자적 데이터로 구성된 테이블은 같은 타입의 데이터를 여러 열에 가질 수 없다. 단 테이블을 사용하는 목적에 따라 원자적 데이터 테이블의 형태가 달라질 수 있다.

**원자적 데이터를 만든다는 말은 데이터를 가능한 한 작게 나누는 것이 아닌 효율적인 테이블을 만드는 한도 내에서 필요한 만큼 가능한 작게 데이터를 나누는 것이다.** 필요 이상으로 데이터를 나눌 필요는 없다. 추가 열이 사용할 필요가 없으면, 데이터를 나눌 수 있더라도 더 이상 쪼갤 필요가 없다.

예를 들면 피자 배달부에게 주소는 하나의 전체 주소의 경우가 더 편리하여 더 나눌 필요가 없는 원자적 데이터가 되며, 쿼리의 경우도 피자 배달부에게는 주문 번호가 중요하기에 주문 번호를 기준으로 쿼리를 주게 된다. 부동산 업자에게는 주소의 이름과 숫자가 나누어진 것이 원자적 데이터가 된다. 쿼리의 경우 거리 이름을 기준으로 쿼리를 주게 된다.

```sql
CREATE TABLE pizza_deliveries(order_number INT, address VARCHAR(30));
```

피자배달부에게는 주문 번호와 완전한 주소가 중요하다.

```sql
INSERT INTO pizza_deliveries  
VALUES
('246',  '59 N. Ajax Rapids'),
('247',  '849 SQL Street'),
('248',  '2348 E. PMP Plaza'),
('249',  '1978 HTML Heights'),
('250',  '24 S. Sevlets Springs'),
('251',  '807 Infinite Circle'),
('252',  '32 Disign Patterns Plaza'),
('253',  '9208 S. Java Ranch'),
('254',  '4653 W. EJB Estate'),
('255',  '8678 OOA&D Orchard');
```

쿼리는 다음과 같이 주문 번호로 줄 수 있다.

```sql
SELECT address FROM pizza_deliveries WHERE order_number = 252;
```

쿼리 결과는 다음과 같다.

```bash
MariaDB [gregs_list]> SELECT address FROM pizza_deliveries WHERE order_number = 252;
+--------------------------+
| address                  |
+--------------------------+
| 32 Disign Patterns Plaza |
+--------------------------+
1 row in set (0.001 sec)
```

부동산 업자의 경우 주소를 세부 주소로 나누어 열을 만들고 테이블을 만든다.

```sql
CREATE TABLE real_estate(street_number INT, street_name VARCHAR(30), property_type VARCHAR(30), price INT);
```

```sql
INSERT INTO real_estate  
VALUES
('59', 'N. Ajax Rapids', 'condo',189000),
('849', 'SQL Street', 'apartment', 109000),
('2348', 'E. PMP Plaza', 'house', 355000),
('1978', 'HTML Heights', 'apartment', 134000),
('24', 'S. Sevlets Springs', 'house', 355000),
('807', 'Infinite Circle', 'condo', 143900),
('32', 'Disign Patterns Plaza', 'house', 465000),
('9208', 'S. Java Ranch', 'house', 699000),
('4653', 'SQL Street', 'apartment', 115000),
('8678', 'OOA&D Orchard', 'house', 355000) ;
```

쿼리는 거리 이름으로 줄 수 있다.

```sql
SELECT price, property_type FROM real_estate WHERE street_name = 'SQL Street';
```

쿼리 결과는 다음과 같다.

```bash
MariaDB [gregs_list]> SELECT price, property_type FROM real_estate WHERE street_name = 'SQL Street';
+--------+---------------+
| price  | property_type |
+--------+---------------+
| 109000 | apartment     |
| 115000 | apartment     |
+--------+---------------+
2 rows in set (0.000 sec)
```

## 정규화

테이블을 정규화 한다는 것은 테이블들이 표준 규칙을 따른다는 의미이며, 원자적 데이터로 테이블을 구성하는 것이 정규화의 첫 걸음이다. 테이블 정규화를 하면 중복 데이터가 없어서 데이터베이스의 크기를 줄여준다. 또한, 찾아야할 데이터가 적어 쿼리가 더 빨라진다. 테이블이 작은 경우에도 커질 경우를 고려하여 정규 테이블을 사용하는 것이 좋다.

정규 테이블이 되려면, 원자적 데이터를 만들고 나서 데이터의 형태가 제 1 정규형(1NF)의 형태를 가져야 한다. 제 1 정규형은 각 행의 데이터들은 원자적 값을 가져야한다. 각 행은 유일무이한 식별자인 기본키를 가져야 한다.

## 기본키 규칙

기본키는 각 행을 다른 행과 구분하는 열이다. 기본키를 구성하는데 필요한 4가지 규칙이 있다.

1. 기본키는 NULL 값이 될 수 없다. NULL 값은 유일무이하지 않기 때문이다.
2. 기본키는 행이 삽입될 때 값이 있어야 한다.
3. 기본키는 간결해야한다. 유일무이한 정보를 가지고 있어야한다.
4. 기본키의 값은 변경 불가능한 값이여야 한다. 유일무이 하기에 변경 가능하면 중복이 생길 수도 있기에 변경 불가능 해야 한다.

기본키를 만들기 위한 가장 좋은 방법은 기본키만을 위한 열을 새로 만드는 방법이다. 새로 만드는 기본키의 경우 synthetic key라 하고, 이미 존재하는 열을 기본키로 사용할 경우 natural key라 한다.

```sql
SHOW CREATE TABLE my_contacts;
```

- SHOW CREATE TABLE 명령어를 사용하면 이전에 만들었던 테이블을 만든 코드를 반환해준다.

```bash
MariaDB [gregs_list]> SHOW CREATE TABLE my_contacts;
| Table       | Create Table                                                                                                      | my_contacts | CREATE TABLE `my_contacts` (
  `last_name` varchar(30) default NULL,
  `first_name` varchar(20) default NULL,
  `email` varchar(50) default NULL,
  `gender` char(1) default NULL,
  `birthday` date default NULL,
  `profession` varchar(50) default NULL,
  `location` varchar(50) default NULL,
  `status` varchar(20) default NULL,
  `interests` varchar(100) default NULL,
  `seeking` varchar(100) default NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 |
1 row in set (0.008 sec)
```

기존에 1장에서 만들었던 테이블에는 기본키가 없었다. 이제 기본키가 있는 테이블을 생성해 보자. 아래와 같이 contact_id 열을 추가한 후 PRIMARY KEY 명령어로 기본키를 만들 수 있다.

기존의 테이블을 지우고 새롭게 테이블을 만들어준다.

```sql
DROP TABLE my_contacts;

CREATE TABLE my_contacts(
    contact_id INT NOT NULL,
    last_name varchar(30) default NULL,
    first_name varchar(20) default NULL,
    email varchar(50) default NULL,
    gender char(1) default NULL,
    birthday date default NULL,
    profession varchar(50) default NULL,
    location varchar(50) default NULL,
    status varchar(20) default NULL,
    interests varchar(100) default NULL,
    seeking varchar(100) default NULL,
    PRIMARY KEY (contact_id)
    );
```

기존의 코드와 달라진 점은 `PRIMARY KEY (contact_id)` 가 붙은 점이다. 이 부분이 contact_id 열을 기본키로 만들어주는 역할을 한다.

```bash
MariaDB [gregs_list]> DESC my_contacts;
+------------+--------------+------+-----+---------+-------+
| Field      | Type         | Null | Key | Default | Extra |
+------------+--------------+------+-----+---------+-------+
| contact_id | int(11)      | NO   | PRI | NULL    |       |
| last_name  | varchar(30)  | YES  |     | NULL    |       |
| first_name | varchar(20)  | YES  |     | NULL    |       |
| email      | varchar(50)  | YES  |     | NULL    |       |
| gender     | char(1)      | YES  |     | NULL    |       |
| birthday   | date         | YES  |     | NULL    |       |
| profession | varchar(50)  | YES  |     | NULL    |       |
| location   | varchar(50)  | YES  |     | NULL    |       |
| status     | varchar(20)  | YES  |     | NULL    |       |
| interests  | varchar(100) | YES  |     | NULL    |       |
| seeking    | varchar(100) | YES  |     | NULL    |       |
+------------+--------------+------+-----+---------+-------+
11 rows in set (0.002 sec)

```

id가 1부터 n까지 숫자를 자동증가하게 만들려면 명령어 AUTO_INCREMENT를 뒤에 추가하면 된다.

```sql
DROP TABLE my_contacts;

CREATE TABLE my_contacts(
    contact_id INT NOT NULL AUTO_INCREMENT,
    last_name varchar(30) default NULL,
    first_name varchar(20) default NULL,
    email varchar(50) default NULL,
    gender char(1) default NULL,
    birthday date default NULL,
    profession varchar(50) default NULL,
    location varchar(50) default NULL,
    status varchar(20) default NULL,
    interests varchar(100) default NULL,
    seeking varchar(100) default NULL,
    PRIMARY KEY (contact_id)
    );
```

- auto_increment 를 설정하면 키 값을 비워두어도 자동으로 숫자가 1부터 오름차순으로 진행된다.

```bash
MariaDB [gregs_list]> desc my_contacts
    -> ;
+------------+--------------+------+-----+---------+----------------+
| Field      | Type         | Null | Key | Default | Extra          |
+------------+--------------+------+-----+---------+----------------+
| contact_id | int(11)      | NO   | PRI | NULL    | auto_increment |
| last_name  | varchar(30)  | YES  |     | NULL    |                |
| first_name | varchar(20)  | YES  |     | NULL    |                |
| email      | varchar(50)  | YES  |     | NULL    |                |
| gender     | char(1)      | YES  |     | NULL    |                |
| birthday   | date         | YES  |     | NULL    |                |
| profession | varchar(50)  | YES  |     | NULL    |                |
| location   | varchar(50)  | YES  |     | NULL    |                |
| status     | varchar(20)  | YES  |     | NULL    |                |
| interests  | varchar(100) | YES  |     | NULL    |                |
| seeking    | varchar(100) | YES  |     | NULL    |                |
+------------+--------------+------+-----+---------+----------------+
11 rows in set (0.004 sec)
```

예시

```sql
CREATE TABLE name_lists(
    id INT NOT NULL AUTO_INCREMENT,
    first_name varchar(20) default NULL,
    last_name varchar(30) default NULL,
    PRIMARY KEY (id)
);
```

```sql
INSERT INTO name_lists (id, first_name, last_name)
VALUES (NULL, 'Marcia', 'Bredy');

INSERT INTO name_lists
VALUES (2, 'Bobby', 'Bredy');

INSERT INTO name_lists (first_name, last_name)
VALUES ('Cindy', 'Bredy');

INSERT INTO name_lists (id, first_name, last_name)
VALUES (99, 'Peter', 'Bredy');

```

id를 NULL로 입력한 곳에 자동으로 1이 들어간 것을 확인할 수 있다. id 열의 값을 입력하지 않은 3번 insert 문의 경우 자동으로 2 다음에 3이 들어간 것을 확인할 수 있다. 물론 맨 마지막의 경우처럼 자동으로 증가 외에도 중복이 아닌 임의의 숫자를 id로 지정할 수 있다.

```bash
MariaDB [gregs_list]> SELECT *  FROM name_lists;
+----+------------+-----------+
| id | first_name | last_name |
+----+------------+-----------+
|  1 | Marcia     | Bredy     |
|  2 | Bobby      | Bredy     |
|  3 | Cindy      | Bredy     |
| 99 | Peter      | Bredy     |
+----+------------+-----------+
4 rows in set (0.001 sec)
```

## 기존 테이블에 기본키 추가하기 ALTER TABLE

그렇다면 이미 만들어진 테이블에 기본키를 추가하려면 어떻게 해야할까 ? ALTER TABLE 문을 이용하면 된다. 5장에서 자세히 다룰 것이기에 여기서는 코드 사용 방법만 한번 보고 넘어간다.

```sql
DROP TABLE my_contacts;

CREATE TABLE my_contacts(
    last_name varchar(30) default NULL,
    first_name varchar(20) default NULL,
    email varchar(50) default NULL,
    gender char(1) default NULL,
    birthday date default NULL,
    profession varchar(50) default NULL,
    location varchar(50) default NULL,
    status varchar(20) default NULL,
    interests varchar(100) default NULL,
    seeking varchar(100) default NULL
    );

INSERT INTO my_contacts(
    last_name, first_name, email, gender, birthday, profession, location, status, interests, seeking
)
VALUES (
    'Anderson', 'Jillian', 'jill_anderson@breakneckpizza.com', 'F', '1980-09-05', 'Technical Writer', 'Palo Alto, CA', 'Single', 'Kayaking, Reptiles', 'Relationship, Friends'
);

INSERT INTO my_contacts(
    first_name, email, profession, location
)
VALUES(
    'Pat', 'patpost@breakneckpizza.com', 'Postal Worker', 'Princeton, NJ'
);
```

기존의 my_contactㄴ 테이블을 다시 복원했다. 여기서 이제 기본키를 추가해보는 작업을 해보자.
새로운 열을 추가하는 명령어는 ALTER TABLE과 ADD COLUMN 명령어로 이루어진다. 아래 코드에서 FIRST는 첫번째 열로 추가해준다는 명령어이며, 여기서 따로 열 순서를 지정하지 않을 경우 마지막 열로 들어간다. 

```sql
ALTER TABLE my_contacts
ADD COLUMN contact_id INT NOT NULL AUTO_INCREMENT FIRST,
ADD PRIMARY KEY (contact_id);
```

- ALTER TABLE 테이블의 구조를 변경하는 방법

기존 테이블에 들어 있던 데이터에 id 열이 생기고 id가 1이 부여된 것을 확인할 수 있다.

```bash
MariaDB [gregs_list]> SELECT * FROM my_contacts;
+------------+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
| contact_id | last_name | first_name | email                            | gender | birthday   | profession       | location      | status | interests          | seeking               |
+------------+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
|          1 | Anderson  | Jillian    | jill_anderson@breakneckpizza.com | F      | 1980-09-05 | Technical Writer | Palo Alto, CA | Single | Kayaking, Reptiles | Relationship, Friends |
|          2 | NULL      | Pat        | patpost@breakneckpizza.com       | NULL   | NULL       | Postal Worker    | Princeton, NJ | NULL   | NULL               | NULL                  |
+------------+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
2 rows in set (0.001 sec)
```