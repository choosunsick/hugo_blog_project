---
title: "SELECT 문: 데이터 가져오기의 축복"
date: 2020-12-03T19:11:13+09:00
draft: False
tags: ["SQL"]
categories: ["SQL"]
---

## 2장

1장에서는 테이블 만드는 방법과 지우는 방법 데이터를 입력하는 방법을 배웠습니다. 이번 2장에서는 데이터를 가져오는 방법에 대해 알아봅시다.현재 데이터베이스에는 1장에서 만든 my_contacts 테이블과 doughnut_list 테이블이 남아있습니다. 이 테이블들을 지우거나 만드는 방법을 잊으셨다면, [링크](https://choosunsick.github.io/post/sql_ch_1/)를 참조해 주시기 바랍니다.

테이블에 있는 데이터를 가져오기 위해서는 SELECT 문을 사용해야 합니다. `SELECT` 문은 입력한 자료 전체를 확인할 수도 있고 `WHERE` 절과 함께 사용하여 일부 데이터만 확인할 수도 있습니다.

```sql
SELECT * FROM my_contacts;
```

- `SELECT`문은 테이블의 데이터를 가져오는 방법
- `*` 별 표시는 테이블의 모든 열을 선택하는 방법

```bash
MariaDB [gregs_list]> SELECT * FROM my_contacts
    -> ;
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
| last_name | first_name | email                            | gender | birthday   | profession       | location      | status | interests          | seeking               |
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
| Anderson  | Jillian    | jill_anderson@breakneckpizza.com | F      | 1980-09-05 | Technical Writer | Palo Alto, CA | Single | Kayaking, Reptiles | Relationship, Friends |
| NULL      | Pat        | patpost@breakneckpizza.com       | NULL   | NULL       | Postal Worker    | Princeton, NJ | NULL   | NULL               | NULL                  |
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
2 rows in set (0.014 sec)
```

위 테이블에서 앤더슨(Anderson)의 정보만 가져오고 싶다면 어떻게 해야할까요? `SELECT`문에 `WHERE`절을 함께 사용하면 됩니다. 아래의 쿼리문은 last_name이라는 열이 'Anderson'인 경우에 해당하는 행의 모든 열을 불러오는 쿼리문입니다. 만약에 'Anderson'인 값이 여러개 있다면 그 행의 모든 열들도 가져옵니다.

```sql
SELECT * FROM my_contacts WHERE last_name = 'Anderson';
```

- `WHERE` 절은 한 열의 값이 특정한 값일 때에 해당하는 데이터를 불러오는 방법입니다.

```bash
MariaDB [gregs_list]> SELECT * FROM my_contacts WHERE last_name = 'Anderson';
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
| last_name | first_name | email                            | gender | birthday   | profession       | location      | status | interests          | seeking               |
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
| Anderson  | Jillian    | jill_anderson@breakneckpizza.com | F      | 1980-09-05 | Technical Writer | Palo Alto, CA | Single | Kayaking, Reptiles | Relationship, Friends |
+-----------+------------+----------------------------------+--------+------------+------------------+---------------+--------+--------------------+-----------------------+
1 row in set (0.002 sec)
```

## WHERE절 사용 예시

먼저 새로운 테이블을 만들어 봅시다. 이 테이블은 음료수에 관한 테이블입니다.

```sql
CREATE TABLE easy_drinks
(drink_name VARCHAR(50), main VARCHAR(30), amount1 DEC(3,2),
second VARCHAR(50), amount2 DEC(3,2), directions VARCHAR(100));
```

데이터를 입력할 때 아래와 같이 여러줄을 한번에 입력할 수도 있습니다.

```sql
INSERT INTO easy_drinks  
VALUES
('Blackthorn',  'tonic water', 1.5, 'pineapple juice', 1 ,'stir with ice, strain into cocktail glass with lemon twist'),
('Blue moon', 'soda', 1.5, 'blueberry juice', 0.75, 'stir with ice, strain into cocktail glass with lemon twist'),
('Oh my Gosh', 'peach nectar', 1, 'pineapple juice', 1, 'stir with ice, strain into shot glass'),
('Lime  Fizz', 'sprite', 1.5, 'lime juice', 1, 'stir with ice, strain into cocktail glass'),
('Kiss on the Lips' , 'cherry juice', 2, 'pineapple juice', 7, 'serve over ice with straw'),
('Hot Gold', 'peach nectar', 3, 'orange juice', 6, 'pour hot orange juice in mug and add peach nectar'),
('Lone Tree', 'soda', 1.5, 'cherry juice', 0.75, 'stir with ice, strain into cocktail glass'),
('Greyhound', 'soda', 1.5, 'grapefruit juice', 5, 'serve over ice, stir well'),
('Indian Summer', 'apple juice', 2, 'hot tea', 6, 'add juice to mug and top offf with hot tea'),
('Bull Frog', 'iced tea', 1.5, 'lemonade', 5, 'serve over ice with lime slice'),
('Soda and It', 'soda', 2, 'grape juice', 1, 'shake in cocktail glass, no ice');
```

입력한 데이터를 확인 해봅시다. 아래와 같은 테이블이 나오면 입력이 잘 이루어진 것입니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks;
+------------------+--------------+---------+------------------+---------+------------------------------------------------------------+
| drink_name       | main         | amount1 | second           | amount2 | directions                                                 |
+------------------+--------------+---------+------------------+---------+------------------------------------------------------------+
| Blackthorn       | tonic water  |    1.50 | pineapple juice  |    1.00 | stir with ice, strain into cocktail glass with lemon twist |
| Blue moon        | soda         |    1.50 | blueberry juice  |    0.75 | stir with ice, strain into cocktail glass with lemon twist |
| Oh my Gosh       | peach nectar |    1.00 | pineapple juice  |    1.00 | stir with ice, strain into shot glass                      |
| Lime  Fizz       | sprite       |    1.50 | lime juice       |    1.00 | stir with ice, strain into cocktail glass                  |
| Kiss on the Lips | cherry juice |    2.00 | pineapple juice  |    7.00 | serve over ice with straw                                  |
| Hot Gold         | peach nectar |    3.00 | orange juice     |    6.00 | pour hot orange juice in mug and add peach nectar          |
| Lone Tree        | soda         |    1.50 | cherry juice     |    0.75 | stir with ice, strain into cocktail glass                  |
| Greyhound        | soda         |    1.50 | grapefruit juice |    5.00 | serve over ice, stir well                                  |
| Indian Summer    | apple juice  |    2.00 | hot tea          |    6.00 | add juice to mug and top offf with hot tea                 |
| Bull Frog        | iced tea     |    1.50 | lemonade         |    5.00 | serve over ice with lime slice                             |
| Soda and It      | soda         |    2.00 | grape juice      |    1.00 | shake in cocktail glass, no ice                            |
+------------------+--------------+---------+------------------+---------+------------------------------------------------------------+
11 rows in set (0.001 sec)
```

이제 이 테이블에서 WHERE절을 다양한 방법으로 사용하여 데이터를 가져와 봅시다.

- '=' : 같다  
- '<>' : 같지 않다  
- '<' : 조건보다 작다 (미만)  
- '>' : 조건보다 크다 (초과)  
- '<=' : 조건과 같거나 작다 (이하)  
- '>=' : 조건과 같거나 크다 (이상)  

```sql
SELECT * FROM easy_drinks WHERE main = 'sprite';
```

sprite를 메인으로 하는 음료는 1개로 아래와 같은 결과가 나옵니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks WHERE main = 'sprite';
+------------+--------+---------+------------+---------+-------------------------------------------+
| drink_name | main   | amount1 | second     | amount2 | directions                                |
+------------+--------+---------+------------+---------+-------------------------------------------+
| Lime  Fizz | sprite |    1.50 | lime juice |    1.00 | stir with ice, strain into cocktail glass |
+------------+--------+---------+------------+---------+-------------------------------------------+
1 row in set (0.001 sec)
```

```sql
SELECT * FROM easy_drinks WHERE main = soda;
```

soda를 메인으로 하는 음료는 4개가 있지만, soda 값에 작은 따옴표가 빠져있기 때문에 에러가 발생합니다. 어떤 값을 찾을 때는 문자열의 경우 반드시 ''를 붙여주어야 합니다.

```sql
SELECT * FROM easy_drinks WHERE amount2 = 6;
```

amount2 열의 값이 6과 같은 음료는 2개가 있습니다. 따라서 아래와 같이 2개의 음료가 나옵니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks WHERE amount2 = 6;
+---------------+--------------+---------+--------------+---------+---------------------------------------------------+
| drink_name    | main         | amount1 | second       | amount2 | directions                                        |
+---------------+--------------+---------+--------------+---------+---------------------------------------------------+
| Hot Gold      | peach nectar |    3.00 | orange juice |    6.00 | pour hot orange juice in mug and add peach nectar |
| Indian Summer | apple juice  |    2.00 | hot tea      |    6.00 | add juice to mug and top offf with hot tea        |
+---------------+--------------+---------+--------------+---------+---------------------------------------------------+
2 rows in set (0.003 sec)
```

```sql
SELECT * FROM easy_drinks WHERE second = 'orange juice';
```

두 번째 음료 재료가 오렌지 주스인 음료는 1개가 있습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks WHERE second = 'orange juice';
+------------+--------------+---------+--------------+---------+---------------------------------------------------+
| drink_name | main         | amount1 | second       | amount2 | directions                                        |
+------------+--------------+---------+--------------+---------+---------------------------------------------------+
| Hot Gold   | peach nectar |    3.00 | orange juice |    6.00 | pour hot orange juice in mug and add peach nectar |
+------------+--------------+---------+--------------+---------+---------------------------------------------------+
1 row in set (0.001 sec)
```

```sql
SELECT * FROM easy_drinks WHERE amount1 < 1.5;
```

첫 번째 음료의 양이 1.5보다 작은 것은 1개가 있습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks WHERE amount1 < 1.5;
+------------+--------------+---------+-----------------+---------+---------------------------------------+
| drink_name | main         | amount1 | second          | amount2 | directions                            |
+------------+--------------+---------+-----------------+---------+---------------------------------------+
| Oh my Gosh | peach nectar |    1.00 | pineapple juice |    1.00 | stir with ice, strain into shot glass |
+------------+--------------+---------+-----------------+---------+---------------------------------------+
1 row in set (0.001 sec)
```

```sql
SELECT * FROM easy_drinks WHERE amount2 < '1';
```

이번에는 amount2의 양이 문자 1보다 작은 것을 찾는 WHERE 절이 있습니다. 숫자가 아니라 문자로 입력해도 쿼리문의 WHERE 절이 역할을 잘 수행합니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks WHERE amount2 < '1';
+------------+------+---------+-----------------+---------+------------------------------------------------------------+
| drink_name | main | amount1 | second          | amount2 | directions                                                 |
+------------+------+---------+-----------------+---------+------------------------------------------------------------+
| Blue moon  | soda |    1.50 | blueberry juice |    0.75 | stir with ice, strain into cocktail glass with lemon twist |
| Lone Tree  | soda |    1.50 | cherry juice    |    0.75 | stir with ice, strain into cocktail glass                  |
+------------+------+---------+-----------------+---------+------------------------------------------------------------+
2 rows in set (0.001 sec)
```

```sql
SELECT * FROM easy_drinks WHERE main > 'soda';
```

이번에는 main 음료가 soda 보다 큰 것을 찾는 WHERE 절입니다. 'soda' 보다 크다는 것이 이해가 잘 안되겠지만 결과를 보면 main의 값이 soda 보다 알파벳 상으로 더 뒤에 오는 것들을 의미합니다. 실제로 tonic water의 t가 s보다 뒤에 오고, sprite의 sp가 so보다 알파벳 순서상 더 뒤에 옵니다. 따라서 소다보다 큰 음료는 2개가 있습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks WHERE main > 'soda';
+------------+-------------+---------+-----------------+---------+------------------------------------------------------------+
| drink_name | main        | amount1 | second          | amount2 | directions                                                 |
+------------+-------------+---------+-----------------+---------+------------------------------------------------------------+
| Blackthorn | tonic water |    1.50 | pineapple juice |    1.00 | stir with ice, strain into cocktail glass with lemon twist |
| Lime  Fizz | sprite      |    1.50 | lime juice      |    1.00 | stir with ice, strain into cocktail glass                  |
+------------+-------------+---------+-----------------+---------+------------------------------------------------------------+
2 rows in set (0.004 sec)
```

```sql
SELECT * FROM easy_drinks WHERE amount1 = '1.5';
```

amount1의 값이 1.5인 값은 6개가 있습니다. 숫자 대신 문자열 '1.5'를 사용했지만 결과는 숫자를 사용한 경우와 똑같이 나옵니다.

```bash
MariaDB [gregs_list]> SELECT * FROM easy_drinks WHERE amount1 = '1.5';
+------------+-------------+---------+------------------+---------+------------------------------------------------------------+
| drink_name | main        | amount1 | second           | amount2 | directions                                                 |
+------------+-------------+---------+------------------+---------+------------------------------------------------------------+
| Blackthorn | tonic water |    1.50 | pineapple juice  |    1.00 | stir with ice, strain into cocktail glass with lemon twist |
| Blue moon  | soda        |    1.50 | blueberry juice  |    0.75 | stir with ice, strain into cocktail glass with lemon twist |
| Lime  Fizz | sprite      |    1.50 | lime juice       |    1.00 | stir with ice, strain into cocktail glass                  |
| Lone Tree  | soda        |    1.50 | cherry juice     |    0.75 | stir with ice, strain into cocktail glass                  |
| Greyhound  | soda        |    1.50 | grapefruit juice |    5.00 | serve over ice, stir well                                  |
| Bull Frog  | iced tea    |    1.50 | lemonade         |    5.00 | serve over ice with lime slice                             |
+------------+-------------+---------+------------------+---------+------------------------------------------------------------+
6 rows in set (0.001 sec)
```

정리해보자면, 숫자는 작은따옴표가 있든 없든 같은 방식으로 동작하지만 문자열의 경우 반드시 작은따옴표가 있어야 쿼리문이 작동합니다.

## 문자열 데이터에 작은따옴표 넣는 방법

문자열을 다룰 때 작은따옴표가 중요하다는 것을 알았습니다. 그렇다면 문자열 값 내부에 작은 따옴표가 있는 경우는 어떻게 처리해야 할까요?

백슬래쉬 `\`를 이용해 줍니다. 영어의 어퍼스트로피와 같이 작은따옴표가 필요한 경우 해당 작은 따옴표 앞에 `\`를 붙여서 이 작은따옴표는 문자열을 나타내는 작은따옴표와 구분해줍니다. 만약 `\`없이 입력하게 될 경우 에러가 발생하게 됩니다. 에러가 발생할 경우 `';` 를 입력하면 빠져나올 수 있습니다.

```sql
INSERT INTO my_contacts
VALUES
('Funyon', 'Steve', 'steve@onionflavoredrings.com', 'M', '1970-04-01', 'Punk', 'Grover\'s MILL, NJ', 'Single', 'Smashing the state', 'compatriots, guitar players');
```

- `\`는 문자열을 나타내는 작은따옴표가 아닌 경우 작은따옴표를 표시하기 위해 사용

입력된 결과는 아래와 같습니다.

```bash
MariaDB [gregs_list]> SELECT * FROM my_contacts;
+-----------+------------+----------------------------------+--------+------------+------------------+-------------------+--------+--------------------+-----------------------------+
| last_name | first_name | email                            | gender | birthday   | profession       | location          | status | interests          | seeking                     |
+-----------+------------+----------------------------------+--------+------------+------------------+-------------------+--------+--------------------+-----------------------------+
| Anderson  | Jillian    | jill_anderson@breakneckpizza.com | F      | 1980-09-05 | Technical Writer | Palo Alto, CA     | Single | Kayaking, Reptiles | Relationship, Friends       |
| NULL      | Pat        | patpost@breakneckpizza.com       | NULL   | NULL       | Postal Worker    | Princeton, NJ     | NULL   | NULL               | NULL                        |
| Funyon    | Steve      | steve@onionflavoredrings.com     | M      | 1970-04-01 | Punk             | Grover's MILL, NJ | Single | Smashing the state | compatriots, guitar players |
+-----------+------------+----------------------------------+--------+------------+------------------+-------------------+--------+--------------------+-----------------------------+
3 rows in set (0.001 sec)
```

## 특정한 열만 가져오기

모든 열을 가져오는 것이 불편할 경우 특정한 열만 가져올 수 있습니다. 별 대신 특정한 열의 이름을 사용하면 그 열에 해당하는 데이터만 가져올 수 있습니다.

```sql
SELECT drink_name, main, second FROM easy_drinks WHERE main = 'soda';
```

- 모든 열을 보는 방법은 더 느리고 가독성에 좋지 않습니다. 원하는 열만 표시하는 방법을 사용하여 원하는 결과를 더 빠르고 가독성 좋은 결과를 가져올 수 있습니다.

```bash
MariaDB [gregs_list]> SELECT drink_name, main, second FROM easy_drinks WHERE main = 'soda';
+-------------+------+------------------+
| drink_name  | main | second           |
+-------------+------+------------------+
| Blue moon   | soda | blueberry juice  |
| Lone Tree   | soda | cherry juice     |
| Greyhound   | soda | grapefruit juice |
| Soda and It | soda | grape juice      |
+-------------+------+------------------+
4 rows in set (0.001 sec)
```


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
