# **Tableau_Project**

## [**SuperStore(Tableau Sample)**](https://public.tableau.com/app/profile/sangah.lee7642/viz/superstore_tableau_sample/1?publish=yes)

## [**Shopping(Kaggle Sample)**](https://public.tableau.com/app/profile/sangah.lee7642/viz/shoppingdata_practice/1_1)


### 캐글 데이터를 다운받아 Dbeaver에서 사용한 구문들


CREATE TABLE customers (customer_id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,  
                                                customer_name VARCHAR(23),  
                                                gender VARCHAR(11),  
                                                age INTEGER,  
                                                home_address VARCHAR(33),  
                                                zip_code INTEGER,  
                                                city VARCHAR(22),  
                                                state VARCHAR(28),  
                                                country VARCHAR(9));  
  
CREATE TABLE products (product_ID INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,  
                                            product_type VARCHAR(8),  
                                            product_name VARCHAR(15),  
                                            size VARCHAR(2),  
                                            colour VARCHAR(6),  
                                            price INTEGER,  
                                            quantity INTEGER,  
                                            description VARCHAR(50));  

CREATE TABLE sales (sales_id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,  
                                        order_id INTEGER REFERENCES 'orders'(order_id),  
                                        product_id INTEGER REFERENCES 'products'(product_ID),  
                                        price_per_unit INTEGER,  
                                        quantity INTEGER);  
  
CREATE TABLE orders (order_id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,  
                                        customer_id INTEGER,  
                                        payment INTEGER,  
                                        order_date VARCHAR(10),  
                                        delivery_date VARCHAR(10),  
                                        FOREIGN KEY(customer_id) REFERENCES 'customers'(customer_id));  
  
  
**어느나라 기준인지??**  
  
SELECT country, COUNT(*)  
FROM CUSTOMERS  
GROUP BY country  
HAVING COUNT(*);  
  
  
**나이대별 평균 및 총 쇼핑금액**  

SELECT SUM(O.PAYMENT) AS 'Total_payment', round(AVG(O.PAYMENT),0) AS 'Average_payment',  
CASE    
        WHEN C.age BETWEEN 20 AND 29 THEN '20s'    
        WHEN C.age BETWEEN 30 AND 39 THEN '30s'    
        WHEN C.age BETWEEN 40 AND 49 THEN '40s'    
        WHEN C.age BETWEEN 50 AND 59 THEN '50s'   
        WHEN C.age BETWEEN 60 AND 69 THEN '60s'    
        WHEN C.age BETWEEN 70 AND 79 THEN '70s'  
        WHEN C.age BETWEEN 80 AND 89 THEN '80s'   
END AS age_range  
FROM customers AS C  
INNER JOIN orders AS O USING(CUSTOMER_ID)  
GROUP BY age_range ;  
  
  
**개인별 쇼핑횟수(하루에 1번이라고 가정)**  
  
SELECT customer_name, COUNT(O.order_date) AS 'total_order_quantity',  
CASE  WHEN C.age BETWEEN 20 AND 29 THEN '20s'    
            WHEN C.age BETWEEN 30 AND 39 THEN '30s'    
            WHEN C.age BETWEEN 40 AND 49 THEN '40s'    
            WHEN C.age BETWEEN 50 AND 59 THEN '50s'    
            WHEN C.age BETWEEN 60 AND 69 THEN '60s'  
            WHEN C.age BETWEEN 70 AND 79 THEN '70s'   
            WHEN C.age BETWEEN 80 AND 89 THEN '80s'  
END AS age_range  
FROM customers AS C   
INNER JOIN orders AS O USING(CUSTOMER_ID)    
INNER JOIN sales AS S USING(order_id)   
GROUP BY customer_name, age_range   
ORDER BY COUNT(O.order_date) DESC;  
  
  
**vip들이 선호하는 상품이름 뭘까?(vip기준은 20번 이상 구매로 둔다)**  
  
SELECT C.CUSTOMER_NAME, P.PRODUCT_NAME, COUNT(P.product_name) AS 'total_quantity'  
FROM products AS P  
INNER JOIN sales AS S USING(product_id)  
INNER JOIN orders AS O USING(order_ID)  
INNER JOIN customers AS C USING(customer_id)  
WHERE C.customer_name IN  
(SELECT C.customer_name  
FROM customers AS C  
INNER JOIN orders AS O USING(CUSTOMER_ID)  
INNER JOIN sales AS S USING(order_id)   
GROUP BY customer_name  
HAVING COUNT(O.order_date)>19)  
GROUP BY P.product_name  
ORDER BY COUNT(P.product_name) DESC;  
    
  
**vip들이 선호하는 상품명은?(vip기준은 100,000 이상 구매로 둔다.서브쿼리 이용)**  
  
SELECT C.CUSTOMER_NAME, P.PRODUCT_NAME, COUNT(P.product_name) AS 'total_quantity'  
FROM products AS P  
INNER JOIN sales AS S USING(product_id)  
INNER JOIN orders AS O USING(order_ID)  
INNER JOIN customers AS C USING(customer_id)  
WHERE C.customer_name IN  
(SELECT C.customer_name  
FROM customers AS C  
INNER JOIN orders AS O USING(CUSTOMER_ID)  
GROUP BY customer_name  
HAVING SUM(O.PAYMENT)>100000)  
GROUP BY P.product_name  
ORDER BY COUNT(P.product_name) DESC;  
  
    
**어떤게 원가인지, 매가인지 불분명하여 sales가 판매가라고 생각하기.재고관리팀에 매가,원가,quantity의미 확인필요**  
  
SELECT S.price_per_unit, S.QUANTITY, P.price, P.QUANTITY, S.price_per_unit- P.price AS 'net_loss'  
FROM sales AS S  
INNER JOIN products AS P USING(product_id)  
WHERE S.price_per_unit- P.price <0;  
  
  
**가장 많이 팔린 상품명은?(sales의 quantity를 판매량으로 보기)**  
  
SELECT S.product_id, P.product_name,  SUM(S.quantity) AS 'total_sales_quantity'  
FROM sales AS S  
INNER JOIN products AS P USING(product_id)  
GROUP BY P.product_name  
ORDER BY SUM(S.QUANTITY) DESC;  
  
  
**가장 많이 팔린 상품 타입은?(sales의 quantity를 판매량으로 보기)**  
  
SELECT S.product_id, P.product_type,  SUM(S.quantity) AS 'total_sales_quantity'  
FROM sales AS S  
INNER JOIN products AS P USING(product_id)  
GROUP BY P.product_type  
ORDER BY SUM(S.QUANTITY) DESC;   