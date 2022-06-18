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















