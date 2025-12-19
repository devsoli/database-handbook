<div dir="rtl">



# چیت‌شیت SQL Server



> قبل امتحان یه نگاه بنداز!



---



## دستورات پایه



```sql

CREATE DATABASE ShopDB;          -- ساخت دیتابیس

USE ShopDB;                      -- انتخاب

DROP DATABASE ShopDB;            -- حذف

 

CREATE TABLE Products (...);     -- ساخت جدول

DROP TABLE Products;             -- حذف جدول

TRUNCATE TABLE Products;         -- خالی کردن (سریع)

```



---



## انواع داده



| نوع | برای چی |

|-----|---------|

| `INT` | عدد صحیح |

| `DECIMAL(18,2)` | قیمت (اعشاری) |

| `VARCHAR(n)` | متن انگلیسی |

| `NVARCHAR(n)` | متن فارسی |

| `DATE` | تاریخ |

| `DATETIME` | تاریخ + ساعت |

| `BIT` | بله/خیر (0/1) |



---



## محدودیت‌ها (Constraints)



```sql

PRIMARY KEY              -- کلید اصلی

IDENTITY(1,1)            -- شماره خودکار

FOREIGN KEY REFERENCES   -- کلید خارجی

NOT NULL                 -- نمی‌تونه خالی باشه

UNIQUE                   -- نباید تکرار بشه

DEFAULT مقدار            -- مقدار پیش‌فرض

CHECK (شرط)              -- بررسی شرط

```



---



## CRUD



### C - Create (اضافه کردن)

```sql

INSERT INTO Products (Name, Price)

VALUES (N'محصول', 1000);

 

-- چند تا باهم

INSERT INTO Products (Name, Price) VALUES

    (N'الف', 1000),

    (N'ب', 2000);

```



### R - Read (خوندن)

```sql

SELECT * FROM Products;

SELECT Name, Price FROM Products;

SELECT TOP 10 * FROM Products;

SELECT DISTINCT Category FROM Products;

```



### U - Update (ویرایش)

```sql

UPDATE Products

SET Price = 2000

WHERE ProductID = 1;          -- WHERE یادت نره!

```



### D - Delete (حذف)

```sql

DELETE FROM Products

WHERE ProductID = 1;          -- WHERE یادت نره!

```



---



## WHERE شرط‌ها



```sql

= > < >= <= <>               -- مقایسه

AND  OR  NOT                 -- منطقی

BETWEEN 10 AND 20            -- بازه

IN (1, 2, 3)                 -- عضویت

IS NULL / IS NOT NULL        -- خالی بودن

LIKE '%abc%'                 -- جستجو

```



### LIKE الگوها

```

%abc    پایان با abc

abc%    شروع با abc

%abc%   شامل abc

_bc     یه کاراکتر + bc

```



---



## ORDER BY مرتب‌سازی



```sql

ORDER BY Price              -- صعودی (پیش‌فرض)

ORDER BY Price ASC          -- صعودی

ORDER BY Price DESC         -- نزولی

ORDER BY Cat, Price DESC    -- چندتایی

```



---



## توابع تجمیعی



```sql

COUNT(*)       -- تعداد

SUM(Price)     -- مجموع

AVG(Price)     -- میانگین

MAX(Price)     -- بیشترین

MIN(Price)     -- کمترین

```



---



## GROUP BY گروه‌بندی



```sql

SELECT CategoryID, COUNT(*), AVG(Price)

FROM Products

WHERE IsActive = 1           -- قبل گروه‌بندی

GROUP BY CategoryID

HAVING COUNT(*) > 5          -- بعد گروه‌بندی

ORDER BY COUNT(*) DESC;

```



> **نکته:** `WHERE` قبل `GROUP BY`، `HAVING` بعدش



---



## JOIN اتصال جداول



```sql

-- INNER: فقط مشترک‌ها

SELECT p.Name, c.CategoryName

FROM Products p

INNER JOIN Categories c ON p.CategoryID = c.CategoryID;

 

-- LEFT: همه از چپ + مشترک

SELECT c.Name, COUNT(p.ProductID)

FROM Categories c

LEFT JOIN Products p ON c.CategoryID = p.CategoryID

GROUP BY c.CategoryID, c.Name;

 

-- RIGHT: همه از راست + مشترک

-- FULL: همه از دو طرف

```



---



## UNION ترکیب



```sql

SELECT Name FROM Products

UNION              -- بدون تکرار

SELECT Name FROM Archive;

 

SELECT Name FROM Products

UNION ALL          -- با تکرار

SELECT Name FROM Archive;

```



---



## Subquery زیرکوئری



```sql

-- در WHERE

SELECT * FROM Products

WHERE Price > (SELECT AVG(Price) FROM Products);

 

-- با IN

WHERE CustomerID IN (SELECT CustomerID FROM Orders);

 

-- با EXISTS

WHERE EXISTS (SELECT 1 FROM Orders WHERE ...);

```



---



## VIEW نما



```sql

CREATE VIEW vw_Active AS

SELECT * FROM Products WHERE IsActive = 1;

 

SELECT * FROM vw_Active;

DROP VIEW vw_Active;

```



---



## Transaction تراکنش



```sql

BEGIN TRANSACTION;

    UPDATE ... ;

    UPDATE ... ;

COMMIT;            -- تایید

-- ROLLBACK;       -- لغو

```



> یا همه انجام بشه، یا هیچکدوم



---



## TRY...CATCH خطا



```sql

BEGIN TRY

    -- کد اصلی

END TRY

BEGIN CATCH

    SELECT ERROR_MESSAGE();

END CATCH

```



---



## Trigger تریگر



```sql

CREATE TRIGGER trg_Name

ON Products

AFTER INSERT/UPDATE/DELETE

AS

BEGIN

    -- INSERTED: ردیف‌های جدید

    -- DELETED: ردیف‌های قدیمی/حذف‌شده

END;

```



---



## Stored Procedure پروسیجر



```sql

CREATE PROCEDURE sp_GetProducts

    @CategoryID INT,

    @MinPrice DECIMAL(18,2) = 0

AS

BEGIN

    SELECT * FROM Products

    WHERE CategoryID = @CategoryID

    AND Price >= @MinPrice;

END;

 

EXEC sp_GetProducts @CategoryID = 1;

```



---



## Function تابع



### Scalar (یه مقدار)

```sql

CREATE FUNCTION fn_Tax(@Price DECIMAL)

RETURNS DECIMAL

AS

BEGIN

    RETURN @Price * 0.09;

END;

 

SELECT dbo.fn_Tax(1000);

```



### Table (جدول)

```sql

CREATE FUNCTION fn_GetByCategory(@CatID INT)

RETURNS TABLE

AS

RETURN (SELECT * FROM Products WHERE CategoryID = @CatID);

 

SELECT * FROM fn_GetByCategory(1);

```



---



## Index ایندکس



```sql

CREATE INDEX IX_Name ON Products(Name);

CREATE UNIQUE INDEX IX_SKU ON Products(SKU);

DROP INDEX IX_Name ON Products;

```



> مثل فهرست کتاب - جستجو رو سریع‌تر می‌کنه



---



## CTE عبارت موقت



```sql

WITH TopProducts AS (

    SELECT *, ROW_NUMBER() OVER (ORDER BY Price DESC) AS Rank

    FROM Products

)

SELECT * FROM TopProducts WHERE Rank <= 10;

```



---



## Window Functions توابع پنجره‌ای



```sql

ROW_NUMBER() OVER (ORDER BY ...)      -- شماره ردیف

RANK() OVER (ORDER BY ...)            -- رتبه (1,2,2,4)

DENSE_RANK() OVER (ORDER BY ...)      -- رتبه (1,2,2,3)

LAG(col) OVER (ORDER BY ...)          -- مقدار قبلی

LEAD(col) OVER (ORDER BY ...)         -- مقدار بعدی

SUM(col) OVER (ORDER BY ...)          -- جمع تجمعی

```



### PARTITION BY

```sql

ROW_NUMBER() OVER (PARTITION BY CategoryID ORDER BY Price)

-- شماره‌گذاری جدا برای هر دسته

```



---



## مقایسه‌های مهم



| | A | B |

|---|---|---|

| **WHERE vs HAVING** | قبل GROUP BY | بعد GROUP BY |

| **DELETE vs TRUNCATE** | کند، برگشت‌پذیر | سریع، برگشت‌ناپذیر |

| **INNER vs LEFT** | فقط مشترک | همه چپ + مشترک |

| **Function vs Procedure** | توی SELECT | با EXEC |

| **RANK vs DENSE_RANK** | 1,2,2,4 | 1,2,2,3 |

| **Clustered vs Non-Clustered** | فقط ۱ عدد | چندتا |



---



## ترتیب اجرای کوئری



```

1. FROM + JOIN

2. WHERE

3. GROUP BY

4. HAVING

5. SELECT

6. DISTINCT

7. ORDER BY

8. TOP

```



---



## نکات طلایی



- `WHERE` رو توی `UPDATE` و `DELETE` فراموش نکن!

- برای فارسی از `NVARCHAR` استفاده کن

- `HAVING` فقط با `GROUP BY` کار می‌کنه

- `Transaction` = همه یا هیچکدوم

- `INDEX` جستجو رو سریع می‌کنه ولی INSERT رو کند



---



موفق باشی!



</div>