# mysql_database

show databases; : 데이터 베이스 목록 보기

#### workbench에서 database의 ERD를 보는 방법

1. workbench에서 위쪽 database를 클릭
2. Reverse Engineer... 을 클릭
3. Stored Connection에 볼 곳을 선택 후 다음 누르기
4. user 비밀번호 입력하기
5. 다음을 누르다가 database를 선택하는 창이 나오면 보고 싶은 database를 체크하고 계속 진행
6. 계속 다음을 누르면 끝!



#### 일별 매출액 조회

```mysql
SELECT A.orderdate, priceeach*quantityordered
FROM classicmodels.orders A
LEFT 
JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber;
```



#### 일별 매출액 계산

```mysql
SELECT A.orderdate, SUM(priceeach*quantityordered) AS Sales
FROM classicmodels.orders A
LEFT 
JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
GROUP BY 1
ORDER BY 1;
```

(GROUP BY 1), (ORDER BY 1) : 첫 번째 열을 기준으로 그룹화



#### 문자열에서 원하는 부분만 가져오기 SUBSTR()

```mysql
SELECT SUBSTR('ABCDE',2,3);
```

ABCDE 중 2번, 즉 B의 위치 2번부터 3개의 문자를 가지고 오게 되므로 2,3,4 번의 문자를 호출하게 된다. 따라서 BCD가 출력됨



#### SUBSTR()을 이용해 월별 매출을 구하는 방법

```mysql
SELECT SUBSTR(A.orderdate,1,7) MM, SUM(priceeach*quantityordered) AS Sales
FROM classicmodels.orders A
LEFT 
JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
GROUP BY 1
ORDER BY 1;
```



#### 중복 제거 하는 순서

1. 전체 출력

```mysql
SELECT orderdate, customernumber, ordernumber
FROM classicmodels.orders;
```

2. 중복 값이 있는지 확인

```mysql
SELECT COUNT(orderdate) N_Orders, 
		COUNT(DISTINCT ordernumber) N_Orders_Distinct
FROM classicmodels.orders;
```

3. 중복 제거 후 출력

```mysql
SELECT orderdate,
COUNT(DISTINCT customernumber) N_Purchaser,
Count(ordernumber) N_Orders
FROM classicmodels.orders
GROUP BY 1
ORDER BY 1;
```



#### 인당 연도별 매출액

1. 연도별 매출액

```mysql
SELECT SUBSTR(A.orderdate, 1, 4) YY,
COUNT(DISTINCT customernumber) N_Purchaser,
SUM(priceeach*quantityordered) AS Sales
FROM classicmodels.orders A
LEFT
JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
GROUP BY 1
ORDER BY 1;
```

2. 인당 연도별 매출액

```mysql
SELECT SUBSTR(A.orderdate, 1, 4) YY,
COUNT(DISTINCT customernumber) N_Purchaser,
SUM(priceeach*quantityordered) AS Sales,
SUM(priceeach*quantityordered) / COUNT(DISTINCT customernumber) AMV
FROM classicmodels.orders A
LEFT
JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
GROUP BY 1
ORDER BY 1;
```



#### 건당 구매 금액(ATV, Average Transaction Value)(연도별)

```mysql
SELECT SUBSTR(A.orderdate, 1, 4) YY,
COUNT(DISTINCT customernumber) N_Purchaser,
SUM(priceeach*quantityordered) AS Sales,
SUM(priceeach*quantityordered) / COUNT(DISTINCT A.ordernumber) ATV
FROM classicmodels.orders A
LEFT
JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
GROUP BY 1
ORDER BY 1;
```



#### 그룹별 구매 지표 구하기

1. 3가지 테이블 결합

```mysql
SELECT *
FROM classicmodels.orders A
LEFT JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
LEFT JOIN classicmodels.customers C
ON A.customernumber = C.customernumber;
```

2. 국가별, 도시별 매출액(나라가 뒤죽박죽)

```mysql
SELECT C.country, C.city,
SUM(priceeach*quantityordered) SALES
FROM classicmodels.orders A
LEFT JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
LEFT JOIN classicmodels.customers C
ON A.customernumber = C.customernumber
GROUP BY 1,2;
```

3. 가독성있게 정렬(나라 명별로 정렬)

```mysql
SELECT C.country, C.city,
SUM(priceeach*quantityordered) SALES
FROM classicmodels.orders A
LEFT JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
LEFT JOIN classicmodels.customers C
ON A.customernumber = C.customernumber
GROUP BY 1,2
ORDER BY 1,2;
```

4. CASE WHEN 구문(이름을 지정가능)

```mysql
SELECT CASE WHEN country IN ('USA', 'Canada') THEN 'North America' ELSE 'Others' END COUNTRY_GRP
FROM classicmodels.customers;
```

5. 북미와 비북미 매출액 비교

```mysql
SELECT  CASE WHEN country IN ('USA', 'Canada') THEN 'North America' ELSE 'Others' END COUNTRY_GRP, 
SUM(priceeach*quantityordered) SALES
FROM classicmodels.orders A
LEFT JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
LEFT JOIN classicmodels.customers C
ON A.customernumber = C.customernumber
GROUP BY 1
ORDER BY 2 DESC;
```

디폴트(ASC) : 오름차순

DESC : 내림차순



6. Rank

```mysql
SELECT name, goals,
RANK() OVER(ORDER BY goals DESC) AS 'RANK',
DENSE_RANK() OVER(ORDER BY goals DESC) AS 'Dense Rank',
ROW_NUMBER() OVER(ORDER BY goals DESC) AS 'Row Number'
FROM football.players;
```

7. 매출 별로 새 테이블 만들기

```mysql
CREATE TABLE classicmodels.stat AS
SELECT C.country,
SUM(priceeach*quantityordered) SALES
FROM classicmodels.orders A
LEFT JOIN classicmodels.orderdetails B
ON A.ordernumber = B.ordernumber
LEFT JOIN classicmodels.customers C
ON A.customernumber = C.customernumber
GROUP BY 1
ORDER BY 2 DESC;
```

8. 위 테이블 실행

```mysql
SELECT *
FROM classicmodels.stat;
```

9. Dense_Rank를 활용해 매출액 등수 매기기

```mysql
SELECT country, SALES, 
DENSE_RANK() OVER(ORDER BY SALES DESC) RNK
FROM classicmodels.stat;
```

10. 출력 결과를 다시 테이블로 생성

```mysql
CREATE TABLE classicmodels.stat_rnk AS
SELECT country, SALES,
DENSE_RANK() OVER(ORDER BY SALES DESC) RNK
FROM classicmodels.stat;
```

11. 위 테이블 실행

```mysql
SELECT *
FROM classicmodels.stat_rnk;
```

12. 상위 5개 국가 추출

```mysql
SELECT *
FROM classicmodels.stat_rnk
WHERE RNK BETWEEN 1 AND 5;
```

13. SUBQUERY

```mysql
SELECT *
FROM
(SELECT*
FROM) A;
```

```mysql
SELECT *
FROM (SELECT country, SALES, 
     DENSE_RANK() OVER(ORDER BY SALES DESC) RNK
     FROM 
     (SELECT C.country,
     SUM(priceeach*quantityordered) SALES
     FROM classicmodels.orders A
     LEFT JOIN classicmodels.orderdetails B
     ON A.ordernumber = B.ordernumber
     LEFT JOIN classicmodels.customers C
     ON A.customernumber = C.customernumber
     GROUP BY 1) A) A
     WHERE RNK <= 5;
```















