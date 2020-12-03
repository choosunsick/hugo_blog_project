---
title: "HEAD FIRST SQL 챕터 2장 요약정리-2"
date: 2020-12-03T19:12:50+09:00
draft: False
tags: ["SQL"]
categories: ["SQL"]
---

## 2장 SELECT 문: 데이터 가져오기의 축복

2장 앞부분에 이어서 진행됩니다. 앞부분의 링크()는 다음과 같습니다.

### 쿼리들의 결합

하나의 조건이 아닌 여러 조건을 동시에 만족하는 데이터를 가져오고 싶은 경우 어떻게 해야할까요?

`WHERE` 절과 `AND`를 이용하면 됩니다.

예를 들면 메인재료가 소다이면서 양이 1.5보다 많이 들어가는 음료를 찾고싶다면 다음과 같이 쿼리를 줄 수 있습니다.

```sql
SELECT drink_name FROM easy_drinks WHERE main = 'soda' AND amount1 > 1.5;
```

- `AND`는 `WHERE` 절에서 두 조건을 모두 만족하는 것을 찾는 경우 사용한다.

```bash
MariaDB [gregs_list]> SELECT drink_name FROM easy_drinks WHERE main = 'soda' AND amount1 > 1.5;
+-------------+
| drink_name  |
+-------------+
| Soda and It |
+-------------+
1 row in set (0.001 sec)
```

여러 조건을 둘 중 하나만 만족해도 되는 경우 `OR`을 사용하면 됩니다.

```sql
SELECT drink_name FROM easy_drinks WHERE main = 'cherry juice' OR second = 'cherry juice';
```

- `OR`은 `WHERE` 절에서 두 조건 중 하나만이라도 만족하는 것을 모두 찾는데 사용한다.

메인이 체리주스인 것은 Kiss on the Lips 1개가 있고 세컨드가 체리주스인 것 역시 Lone Tree 라는 이름의 음료 1개가 있다.

```bash
MariaDB [gregs_list]> SELECT drink_name FROM easy_drinks WHERE main = 'cherry juice' OR second = 'cherry juice';
+------------------+
| drink_name       |
+------------------+
| Kiss on the Lips |
| Lone Tree        |
+------------------+
2 rows in set (0.001 sec)
```

`AND`와 `OR`을 쓸 때는 두 조건에 유의해야합니다. 왜냐하면, `AND`의 경우 두 조건에 모두 만족하지 않는 경우가 있을 수 있고 `OR`의 경우 두 조건 중 하나도 만족하지 않는 경우 결과가 나오지 않을 수 있다.

### NULL 값 찾기

그렇다면 NULL 값이 있는 데이터를 찾으려면 어떻게 해야할까요?

```sql
SELECT * FROM my_contacts WHERE gender IS NULL;
```

- `WHERE`절에 열이름 = `IS NULL` 로 열의 값 중 NULL 값인 데이터를 가져올 수 있습니다.

### 일부 문자열을 이용해 데이터를 찾는 방법

```sql
SELECT * FROM my_contacts WHERE location LIKE '%NJ';

SELECT * FROM my_contacts WHERE location LIKE '_NJ';

```

- `LIKE`는 특정 문자열의 값이 어떤 열의 값 내부에 있는 경우 찾아주는 역할
- `%`, `_` 이 두개의 와일드카드와 함께 쓰이는 데 `%`의 경우 찾는 문자 앞에 다수의 문자가 있는 경우를 뜻하며 `_` 찾는 문자 앞에 하나의 문자가 있는 경우를 말한다.

```bash
MariaDB [gregs_list]> SELECT * FROM my_contacts WHERE location LIKE '%NJ';
+-----------+------------+------------------------------+--------+------------+---------------+-------------------+--------+--------------------+-----------------------------+
| last_name | first_name | email                        | gender | birthday   | profession    | location          | status | interests          | seeking                     |
+-----------+------------+------------------------------+--------+------------+---------------+-------------------+--------+--------------------+-----------------------------+
| NULL      | Pat        | patpost@breakneckpizza.com   | NULL   | NULL       | Postal Worker | Princeton, NJ     | NULL   | NULL               | NULL                        |
| Funyon    | Steve      | steve@onionflavoredrings.com | M      | 1970-04-01 | Punk          | Grover's MILL, NJ | Single | Smashing the state | compatriots, guitar players |
+-----------+------------+------------------------------+--------+------------+---------------+-------------------+--------+--------------------+-----------------------------+
2 rows in set (0.010 sec)


MariaDB [gregs_list]> SELECT * FROM my_contacts WHERE location LIKE '_NJ';
Empty set (0.001 sec)
```

### 두 개의 부등호 대신 사용하는 BETWEEN

비교 연산자를 통해 두 개의 값 사이에 존재하는 값의 데이터를 찾을 수 있음을 알고 있습니다. 하지만 더 간단한 방식으로도 찾을 수 있습니다.

```sql
SELECT drink_name, second FROM easy_drinks WHERE amount1 >= 1 AND amount1 <= 1.5;
```

위의 결과는 아래와 같습니다.

```bash
MariaDB [gregs_list]> SELECT drink_name, second FROM easy_drinks WHERE amount1 >= 1 AND amount1 <= 1.5;
+------------+------------------+
| drink_name | second           |
+------------+------------------+
| Blackthorn | pineapple juice  |
| Blue moon  | blueberry juice  |
| Oh my Gosh | pineapple juice  |
| Lime  Fizz | lime juice       |
| Lone Tree  | cherry juice     |
| Greyhound  | grapefruit juice |
| Bull Frog  | lemonade         |
+------------+------------------+
7 rows in set (0.001 sec)
```

부등호 두개를 쓰는 대신 `BETWEEN`을 사용할 수 있습니다.

```sql
SELECT drink_name, second FROM easy_drinks WHERE amount1 BETWEEN 1 AND 1.5;
```

- `BETWEEN`은 두 값을 포함하여 사이에 있는 값을 찾아주는 역할

결과는 위와 같습니다.

```bash
MariaDB [gregs_list]> SELECT drink_name, second FROM easy_drinks WHERE amount1 BETWEEN 1 AND 1.5;
+------------+------------------+
| drink_name | second           |
+------------+------------------+
| Blackthorn | pineapple juice  |
| Blue moon  | blueberry juice  |
| Oh my Gosh | pineapple juice  |
| Lime  Fizz | lime juice       |
| Lone Tree  | cherry juice     |
| Greyhound  | grapefruit juice |
| Bull Frog  | lemonade         |
+------------+------------------+
7 rows in set (0.002 sec)
```

### 여러개의 OR 대신 사용하는 IN과 그 반대인 NOT IN

`OR`을 여러번 사용하여 여러 조건을 달고 있는 쿼리문을 만들 때 `OR`의 개수가 많아지면 가독성과 그 의미를 찾기 힘들어 집니다. 이에 따라 대신 사용할 수 있는 것이 `IN` 입니다. 예를 들면 메인 음료로 소다 또는 스프라이트 또는 토닉워터가 들어가는 음료를 찾고 싶은 경우 `OR`을 두번 사용해야 합니다. 그러나 IN을 사용하면 길이도 더 짧은 쿼리문을 만들 수 있습니다.

```sql
SELECT drink_name FROM easy_drinks WHERE main = 'soda' OR main = 'sprite' OR main = 'tonic water';
```

```bash
MariaDB [gregs_list]> SELECT drink_name FROM easy_drinks WHERE main = 'soda' OR main = 'sprite' OR main = 'tonic water';
+-------------+
| drink_name  |
+-------------+
| Blackthorn  |
| Blue moon   |
| Lime  Fizz  |
| Lone Tree   |
| Greyhound   |
| Soda and It |
+-------------+
6 rows in set (0.001 sec)
```

```sql
SELECT drink_name FROM easy_drinks WHERE main IN ('soda', 'sprite','tonic water');
SELECT drink_name FROM easy_drinks WHERE main NOT IN ('soda', 'sprite','tonic water');

```

- `IN`은 `WHERE`절과 함께 사용되며, 여러개의 OR 문을 대체하는 역할
- `NOT IN`은 `WHERE`절과 함께 사용되며, `IN`의 반대로 들어있지 않는 것을 찾는 역할

```bash
MariaDB [gregs_list]> SELECT drink_name FROM easy_drinks WHERE main IN ('soda', 'sprite','tonic water');
+-------------+
| drink_name  |
+-------------+
| Blackthorn  |
| Blue moon   |
| Lime  Fizz  |
| Lone Tree   |
| Greyhound   |
| Soda and It |
+-------------+
6 rows in set (0.001 sec)


MariaDB [gregs_list]> SELECT drink_name FROM easy_drinks WHERE main NOT IN ('soda', 'sprite','tonic water');
+------------------+
| drink_name       |
+------------------+
| Oh my Gosh       |
| Kiss on the Lips |
| Hot Gold         |
| Indian Summer    |
| Bull Frog        |
+------------------+
5 rows in set (0.001 sec)
```

참고로 `NOT`의 경우 IN 외에도 별개로 사용될 수 있다.
