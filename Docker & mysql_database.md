# mysql_database

## 4장

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



#### 재구매율

1. 기초 틀(거래가 없으면 매칭이 안됨)

```mysql
SELECT A.customernumber, A.orderdate, B.customernumber, B.orderdate
FROM classicmodels.orders A
LEFT JOIN classicmodels.orders B
ON A.customernumber = B.customernumber
AND SUBSTR(A.orderdate, 1, 4) = SUBSTR(B.orderdate, 1, 4) -1;
```

2. Retention Rate(%)

```mysql
SELECT C.country,
SUBSTR(A.orderdate, 1, 4) YY,
COUNT(DISTINCT A.customernumber) BU_1,
COUNT(DISTINCT B.customernumber) BU_2, 
COUNT(DISTINCT B.customernumber)/COUNT(DISTINCT A.customernumber) 
RETENTION_RATE
FROM classicmodels.orders A
LEFT JOIN classicmodels.orders B
ON A.customernumber = B.customernumber
AND SUBSTR(A.orderdate, 1, 4) = SUBSTR(B.orderdate, 1, 4) -1
LEFT JOIN classicmodels.customers C
ON A.customernumber = C.customernumber
GROUP BY 1, 2;
```



#### BEST SELLER

```mysql
CREATE TABLE classicmodels.product_sales AS
SELECT D.productname,
SUM(quantityordered*priceeach) SALES
FROM classicmodels.orders A
LEFT JOIN classicmodels.customers B
ON A.customernumber = B.customernumber
LEFT JOIN classicmodels.orderdetails C
ON A.ordernumber = C.ordernumber
LEFT JOIN classicmodels.products D
ON C.productcode = D.productcode
WHERE B.country = 'USA'
GROUP BY 1;

SELECT *
FROM
(SELECT *, ROW_NUMBER() OVER(ORDER BY sales DESC) RNK
FROM classicmodels.product_sales) A
WHERE RNK <= 5
ORDER BY RNK;
```



#### CHURN RATE(%)

Churn : max(구매일, 접속일) 이후 일정 기간(3개월 이상) 구매, 접속하지 않은 상태

1. 고객의 마지막 구매일

```mysql
SELECT MAX(orderdate) MX_ORDER
FROM classicmodels.orders;
```

2. 2005-06-01 기준으로 며칠이 소요됐는지

```mysql
SELECT customernumber, MAX(orderdate) MX_ORDER
FROM classicmodels.orders
GROUP BY 1;
```

3. DATEDIFF() : DATEDIFF(date1, date2) 에서 date1 - date2 의 결과가 출력된다.

```mysql
SELECT customernumber, MX_ORDER, '2005-06-01',
DATEDIFF('2005-06-01', MX_ORDER) DIFF
FROM 
(SELECT customernumber, MAX(orderdate) MX_ORDER
FROM classicmodels.orders
GROUP BY 1) BASE;
```

4. Churn type

```mysql
SELECT *, CASE WHEN DIFF >= 90 THEN 'CHURN' ELSE 'NON-CHURN' END CHURN_TYPE
FROM
(SELECT customernumber, MX_ORDER, '2005-06-01' END_POINT, 
 DATEDIFF('2005-06-01', MX_ORDER) DIFF
 FROM
 (SELECT customernumber, MAX(orderdate) MX_ORDER
 FROM classicmodels.orders
 GROUP BY 1) BASE) BASE;
```

5. Churn Rate(%)

```mysql
SELECT CASE WHEN DIFF >= 90 THEN 'CHURN' ELSE 'NON-CHURN' END CHURN_TYPE,
COUNT(DISTINCT customernumber) N_CUS
FROM
(SELECT customernumber, MX_ORDER, '2005-06-01' END_POINT, 
 DATEDIFF('2005-06-01', MX_ORDER) DIFF
 FROM
 (SELECT customernumber, MAX(orderdate) MX_ORDER
 FROM classicmodels.orders
 GROUP BY 1) BASE) BASE
 GROUP BY 1;
```

6. Churn Table

```mysql
CREATE TABLE classicmodels.churn_list AS
SELECT CASE WHEN DIFF >= 90 THEN 'CHURN' ELSE 'NON-CHURN' END CHURN_TYPE,
customernumber
FROM
(SELECT customernumber, MX_ORDER, '2005-06-01' END_POINT, 
 DATEDIFF('2005-06-01', MX_ORDER) DIFF
 FROM
 (SELECT customernumber, MAX(orderdate) MX_ORDER
 FROM classicmodels.orders
 GROUP BY 1) BASE) BASE;
```

7. Churn 고객이 가장 많이 구매한 Productline

```mysql
SELECT C.productline,
COUNT(DISTINCT B.customernumber) BU
FROM classicmodels.orderdetails A
LEFT JOIN classicmodels.orders B
ON A.ordernumber = B.ordernumber
LEFT JOIN classicmodels.products C
ON A.productcode = C.productcode
GROUP BY 1;
```

8. Churn type, product line 별 구매자 수

```mysql
SELECT D.churn_type, C.productline,
COUNT(DISTINCT B.customernumber) BU
FROM classicmodels.orderdetails A
LEFT JOIN classicmodels.orders B
ON A.ordernumber = B.ordernumber
LEFT JOIN classicmodels.products C
ON A.productcode = C.productcode
LEFT JOIN classicmodels.churn_list D
ON B.customernumber = D.customernumber
GROUP BY 1,2
ORDER BY 1,3 DESC;
```







