---
title: "DElETE와 UPDATE 문"
date: 2020-12-14T19:27:28+09:00
draft: False
tags: ["SQL"]
categories: ["SQL"]
---

## 3장

이 글은 MYSQL 문법을 복습할 겸해서 [HEAD FIRST SQL 책](http://www.yes24.com/Product/Goods/2922303?OzSrank=2)을 요약정리 한 것입니다.

2장에서는 데이터를 가져오는 방법에 대해 알아보았습니다. SELECT 문의 사용 방법에 대해 다시 확인하고 싶으시다면 [링크](https://choosunsick.github.io/post/sql_ch_2/)를 참조해 주시기 바랍니다.

이번 장에서 테이블 내 데이터를 제거 및 변경하는 방법에 대해 알아봅니다. 기존의 테이블을 사용하지 않고 새로운 테이블을 만들어 사용합니다.
광대들의 정보가 담긴 테이블을 만듭니다.

```sql
CREATE TABLE clown_info(name VARCHAR(20), last_seen VARCHAR(50), appearance VARCHAR(200), activities VARCHAR(50));
```

광대들의 정보 테이블에는 이름, 등장 장소, 외형, 활동 등의 정보가 나타납니다.

```sql
INSERT INTO clown_info
    VALUES
        ('Elsie', 'Cherry Hill Sennior Center', 'F, red hair, green dress, huge feet', 'balloons, little car'),
        ('Pickles', 'Jack Green\'s Party', 'M, orange hair, blue suit, huge feet', 'mime'),
        ('Sunggles', 'Ball-Mart', 'F, yellow shirt, baggy red pants', 'horn, umbrella'),
        ('Mr.Hobo', 'BG Circus','M, cigar, black hair, tiny hat', 'violin'),
        ('Clarabelle', 'Belmont Senior Certer', 'F, pink hair, huge flower, blue dress', 'yelling, dancing'),
        ('Scooter', 'Oakland Hospital', 'M, blue hair, red suit, huge nose', 'balloons'),
        ('Zippo', 'Millstone Mall', 'F, orange suit, baggy pants', 'dancing'),
        ('Babe', 'Earl\'s Autos', 'F, all pink and sparkly', 'balancing, little car'),
        ('Bonzo', NULL, 'M, in drag, polka dotted dress', 'singing, dancing'),
        ('Sniffles', 'Tracy\'s', 'M, green and puple suit, pointy nose', NULL);
```

완성된 테이블은 아래와 같습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM clown_info;
+------------+----------------------------+---------------------------------------+-----------------------+
| name       | last_seen                  | appearance                            | activities            |
+------------+----------------------------+---------------------------------------+-----------------------+
| Elsie      | Cherry Hill Sennior Center | F, red hair, green dress, huge feet   | balloons, little car  |
| Pickles    | Jack Green's Party         | M, orange hair, blue suit, huge feet  | mime                  |
| Sunggles   | Ball-Mart                  | F, yellow shirt, baggy red pants      | horn, umbrella        |
| Mr.Hobo    | BG Circus                  | M, cigar, black hair, tiny hat        | violin                |
| Clarabelle | Belmont Senior Certer      | F, pink hair, huge flower, blue dress | yelling, dancing      |
| Scooter    | Oakland Hospital           | M, blue hair, red suit, huge nose     | balloons              |
| Zippo      | Millstone Mall             | F, orange suit, baggy pants           | dancing               |
| Babe       | Earl's Autos               | F, all pink and sparkly               | balancing, little car |
| Bonzo      | NULL                       | M, in drag, polka dotted dress        | singing, dancing      |
| Sniffles   | Tracy's                    | M, green and puple suit, pointy nose  | NULL                  |
+------------+----------------------------+---------------------------------------+-----------------------+
10 rows in set (0.002 sec)
```

광대들의 최신 정보가 변경되었다고 할 때 우리는 기존의 `INSERT INTO` 문을 통해 새로운 정보를 갱신할 수 있습니다. 예를들면 아래와 같이 새로운 정보가 담긴 데이터를 추가할 수 있습니다.

```sql
INSERT INTO clown_info
    VALUES
        ('Zippo', 'Millstone Mall', 'F, orange suit, baggy pants', 'dancing, singing'),
        ('Sunggles', 'Ball-Mart', 'F, yellow shirt, baggy blue pants', 'horn, umbrella'),
        ('Bonzo', 'Dickson Park', 'M, in drag, polka dotted dress', 'singing, dancing'),
        ('Sniffles', 'Tracy\'s', 'M, green and puple suit, pointy nose', 'little car'),
        ('Mr.Hobo', 'Eric Gray\'s Party', 'M, cigar, black hair, tiny hat', 'violin');
```

추가된 데이터까지 광대들의 정보가 담긴 테이블은 아래와 같이 만들어집니다.

```bash
MariaDB [gregs_list]> SELECT * FROM clown_info;
+------------+----------------------------+---------------------------------------+-----------------------+
| name       | last_seen                  | appearance                            | activities            |
+------------+----------------------------+---------------------------------------+-----------------------+
| Elsie      | Cherry Hill Sennior Center | F, red hair, green dress, huge feet   | balloons, little car  |
| Pickles    | Jack Green's Party         | M, orange hair, blue suit, huge feet  | mime                  |
| Sunggles   | Ball-Mart                  | F, yellow shirt, baggy red pants      | horn, umbrella        |
| Mr.Hobo    | BG Circus                  | M, cigar, black hair, tiny hat        | violin                |
| Clarabelle | Belmont Senior Certer      | F, pink hair, huge flower, blue dress | yelling, dancing      |
| Scooter    | Oakland Hospital           | M, blue hair, red suit, huge nose     | balloons              |
| Zippo      | Millstone Mall             | F, orange suit, baggy pants           | dancing               |
| Babe       | Earl's Autos               | F, all pink and sparkly               | balancing, little car |
| Bonzo      | NULL                       | M, in drag, polka dotted dress        | singing, dancing      |
| Sniffles   | Tracy's                    | M, green and puple suit, pointy nose  | NULL                  |
| Zippo      | Millstone Mall             | F, orange suit, baggy pants           | dancing, singing      |
| Sunggles   | Ball-Mart                  | F, yellow shirt, baggy blue pants     | horn, umbrella        |
| Bonzo      | Dickson Park               | M, in drag, polka dotted dress        | singing, dancing      |
| Sniffles   | Tracy's                    | M, green and puple suit, pointy nose  | little car            |
| Mr.Hobo    | Eric Gray's Party          | M, cigar, black hair, tiny hat        | violin                |
+------------+----------------------------+---------------------------------------+-----------------------+
15 rows in set (0.002 sec)
```

그러나 이런 방식으로 데이터를 새로 갱신하는 것에 문제점은 무엇일까요?

이 방식의 문제점은 테이블의 데이터를 갱신하는 사람이 여려명일 경우 중복된 데이터가 생길 수 있다는 점과 어떤 데이터가 최신 정보인지 알 수 없다는 점입니다. 즉 무조건 적으로 마지막 행이 최신 데이터라 판단하기에 어려움이 있다. 또한, 정보를 갱신하기 위해 바뀐 부분만 변경하지 않고 새롭게 데이터를 추가할 경우 데이터 베이스의 공간을 낭비한다는 점도 문제가 될 수 있습니다.

예를 들면 zippo 라는 이름의 광대의 데이터의 경우 활동사항에만 변경이 있었고, 나머지 정보는 그대로인 경우 데이터 중복 및 공간 낭비라는 문제가 생깁니다.

```sql
SELECT * FROM clown_info WHERE name = 'Zippo';
```

```bash
MariaDB [gregs_list]> SELECT * FROM clown_info WHERE name = 'Zippo';
+-------+----------------+-----------------------------+------------------+
| name  | last_seen      | appearance                  | activities       |
+-------+----------------+-----------------------------+------------------+
| Zippo | Millstone Mall | F, orange suit, baggy pants | dancing          |
| Zippo | Millstone Mall | F, orange suit, baggy pants | dancing, singing |
+-------+----------------+-----------------------------+------------------+
2 rows in set (0.005 sec)
```

## 데이터를 지우는 방법 DELETE

이런 문제를 해결하기 위해서 이전의 데이터를 지우는 방법인 DELETE 문이 필요합니다. 기존의 `zippo`의 활동인 춤을 기록하고 있는 행을 지워봅시다.

```sql
DELETE FROM clown_info WHERE activities = 'dancing';
```

- DELETE 문은 테이블의 행을 지우는 방법입니다.

위의 명령어는 활동이 `'dancing'`인 모든 행을 지우는 명령어 입니다. 현재 테이블에 활동이 'dancing'인 경우는 `Zippo`의 행 밖에 없기에 이 명령어로 이전 기록을 지울 수 있습니다.

```sql
SELECT * FROM clown_info WHERE name = 'Zippo';
```

다시 `Zippo`의 기록을 확인해보면 아래와 같이 데이터가 하나만 남은 것을 확인할 수 있습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM clown_info WHERE name = 'Zippo';
+-------+----------------+-----------------------------+------------------+
| name  | last_seen      | appearance                  | activities       |
+-------+----------------+-----------------------------+------------------+
| Zippo | Millstone Mall | F, orange suit, baggy pants | dancing, singing |
+-------+----------------+-----------------------------+------------------+
```

만약 활동이 `'dancing'`인 행이 더 있다면 그 행도 지워지게 됩니다. 따라서 이런 것을 방지하기 위해서는 `WHERE` 조건문에 이름이 `Zippo`라는 조건을 추가해줍니다.

그러면 명령어는 아래와 같습니다.

```sql
DELETE FROM clown_info WHERE name = 'Zippo' AND activities = 'dancing';
```

**DELETE 문 핵심**

`DELETE` 문은 한 열이나 여러 열의 값을 지우는 데 사용할 수 없다. 대신 `WHERE` 절의 조건에 따라 하나의 행이나 여러 행을 지울 수 있다.
`DELETE` 문을 사용할 때 `WHERE` 절을 누락할 경우 테이블의 모든 행을 지우게 된다.
`DELETE` 문을 사용할 때 `WHERE` 절의 조건을 정확히 사용해서 원하는 데이터(행)만 지울 수 있도록 해야한다.

## 데이터를 갱신하는 방법 UPDATE

`INSERT INTO`와 `DELETE`로 데이터를 갱신할 수 있지만, `UPDATE` 문을 사용하는 것이 더 편리합니다.

```sql
SELECT * FROM clown_info WHERE name = 'Mr.Hobo';
```

기존에 호보의 정보는 아래와 같습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM clown_info WHERE name = 'Mr.Hobo';
+---------+-------------------+--------------------------------+------------+
| name    | last_seen         | appearance                     | activities |
+---------+-------------------+--------------------------------+------------+
| Mr.Hobo | BG Circus         | M, cigar, black hair, tiny hat | violin     |
| Mr.Hobo | Eric Gray's Party | M, cigar, black hair, tiny hat | violin     |
+---------+-------------------+--------------------------------+------------+
2 rows in set (0.001 sec)
```

이 때 에릭 그레이의 파티 대신 트레이시 라는 장소로 업데이트 하고 싶다면 아래와 같은 명령어를 사용할 수 있습니다. 만약 `INSERT INTO`와 `DELETE`를 이용할 경우 기존의 last_seen에 Eric Gray's Party가 입력된 행을 지우고 새로운 행을 INSERT INTO 하는 번거로운 방법을 사용해야 합니다.

```sql
UPDATE clown_info
SET last_seen = 'Tracy\'s'
WHERE name = 'Mr.Hobo' AND last_seen = 'Eric Gray\'s Party';
```

- UPDATE 문은 기존의 열의 값을 변경하는 데 사용합니다.

```bash
MariaDB [gregs_list]> SELECT * FROM clown_info WHERE name = 'Mr.Hobo';
+---------+-----------+--------------------------------+------------+
| name    | last_seen | appearance                     | activities |
+---------+-----------+--------------------------------+------------+
| Mr.Hobo | BG Circus | M, cigar, black hair, tiny hat | violin     |
| Mr.Hobo | Tracy's   | M, cigar, black hair, tiny hat | violin     |
+---------+-----------+--------------------------------+------------+
2 rows in set (0.001 sec)
```

**UPDATE 문 요약**

`UPDATE` 문은 하나의 열 또는 여러 열 값들을 변경하는데 사용할 수 있다.
`UPDATE` 문을 사용할 때 `WHERE` 절을 누락할 경우 테이블의 모든 열 값을 바꾸게 된다.
`UPDATE` 문을 사용할 때 `WHERE` 절의 조건을 정확히 사용해서 원하는 데이터(행)만 지울 수 있도록 해야한다.
`UPDATE` 문은 `SET` 과 함께 사용된다.