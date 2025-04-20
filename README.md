# shujukuzuoye05  

考虑关系模式product(product_no, name, price)，完成下面的题目：

## 题目一：创建关系并导入导出数据

### 使用 PostgreSQL 完成  

```
-- 创建product表
CREATE TABLE product (
    product_no INT PRIMARY KEY,
    name VARCHAR(100),
    price NUMERIC(10,2)
);

-- 从txt文件导入数据 (文件名为products.txt，格式为: 1|Product1|100.00，如：650|water|10.00)
\copy product FROM 'D:/shujuku/products.txt' DELIMITER '|';

-- 导出数据到CSV文件
\copy product TO '‪D:/shujuku/products_export.csv' WITH CSV HEADER;
```
注：如下截图包含题目一和题目二前五小问的内容：
![2834ac328151871fa593436f55bfdd0](https://github.com/user-attachments/assets/5fb636fb-93c6-474d-9489-e2ba49a84d6a)


## 题目二：数据操作

```
-- 1.添加编号为666，名字为cake，价格不详的商品
INSERT INTO product (product_no, name, price)
VALUES (666, 'cake', NULL);

-- 2.使用一条SQL语句同时添加3个商品
INSERT INTO product (product_no, name, price)
VALUES (667, 'bread', 25.50),
       (668, 'milk', 12.75),
       (669, 'cheese', 45.00);

-- 3.将商品价格统一打8折
UPDATE product
SET price = price * 0.8
WHERE price is NOT NULL;

-- 4.将价格大于100的商品上涨2%，其余上涨4%
UPDATE product 
SET price =
    CASE 
        WHEN price > 100 THEN price * 1.02 
        ELSE price * 1.04 
END
WHERE price is NOT NULL;

-- 5.将名字包含cake的商品删除
DELETE FROM product
WHERE name LIKE '%cake%';

-- 6.将价格高于平均价格的商品删除
DELETE FROM product
WHERE price > (SELECT AVG(price) FROM product);
```
注：如下截图包含题目二第六小问内容：
![fc27dbbe6c899fbae86b7ddb5eab094](https://github.com/user-attachments/assets/1b4fa17f-7626-48fa-98df-debff828729b)


## 题目三：大数据量操作性能比较

### 使用 PostgreSQL 完成  

```
-- 添加10万条测试数据【注：此处代码经过修改，原题代码因没有设置product_no会导致报错（最初将其设为primary key若为空则报错）】
INSERT INTO product (product_no, name, price)
SELECT 
    generate_series,  -- 直接用序列值作为ID
    'Product' || generate_series,   -- 生成名称 Product1, Product2, ...
    ROUND((random() * 1000)::numeric, 2)  -- 生成0到1000之间的随机价格，保留2位小数
FROM generate_series(1, 100000);

-- 测试DELETE性能 (删除所有数据)
-- 记录开始时间
\o D:/shujuku/delete_time.txt
\timing on
DELETE FROM product;
\timing off
\o

-- 重新插入数据
INSERT INTO product (product_no, name, price)
SELECT 
    generate_series,  -- 直接用序列值作为ID
    'Product' || generate_series,   -- 生成名称 Product1, Product2, ...
    ROUND((random() * 1000)::numeric, 2)  -- 生成0到1000之间的随机价格，保留2位小数
FROM generate_series(1, 100000);

-- 测试TRUNCATE性能
\o D:/shujuku/truncate_time.txt
\timing on
TRUNCATE TABLE product;
\timing off
\o
```
注：如下截图包含题目三内容：
![7addb0edc59c86bb98279f7e6fffc03](https://github.com/user-attachments/assets/bda39c2c-a6e6-40f0-87d2-538d01bc4e87)


### 性能比较说明

对于PostgreSQL:
- DELETE 是逐行删除操作，会记录每条删除的日志，速度较慢
- TRUNCATE 是直接删除整个表数据，不记录单独行的删除日志，速度非常快
