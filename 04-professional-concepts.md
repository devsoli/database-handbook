<div dir="rtl">



# مفاهیم پیشرفته - Transaction، Trigger، Procedure و بقیه چیزای حرفه‌ای



---



## این بخش چیه؟



اینجا چیزایی یاد می‌گیری که توی کار واقعی خیلی به درد می‌خورن:

- **Transaction**: یا همه عملیات انجام بشه یا هیچکدوم

- **Trigger**: کد خودکار

- **Stored Procedure**: تابع‌هایی که توی دیتابیس ذخیره میشن

- **Index**: سریع‌تر کردن جستجو

- و چیزای باحال دیگه!



---



## فهرست



1. [Transaction - تراکنش](#1-transaction)

2. [Trigger - تریگر](#2-trigger)

3. [Stored Procedure - پروسیجر](#3-stored-procedure)

4. [Function - تابع](#4-function)

5. [Index - ایندکس](#5-index)

6. [Cursor - کرسر](#6-cursor)

7. [CTE - عبارت جدول مشترک](#7-cte)

8. [Window Functions - توابع پنجره‌ای](#8-window-functions)



---



## 1. Transaction



### چیه؟

تراکنش یعنی یه سری عملیات که یا **همه** انجام بشن یا **هیچکدوم**.



فکر کن می‌خوای پول انتقال بدی:

1. از حساب A کم کن

2. به حساب B اضافه کن



اگه وسط کار برق بره چی؟ نباید پول از A کم بشه ولی به B نرسه!



### اصول ACID

| حرف | معنی | یعنی چی؟ |

|-----|------|---------|

| A | Atomicity | همه یا هیچکدوم |

| C | Consistency | داده‌ها همیشه درست باشن |

| I | Isolation | تراکنش‌ها قاطی هم نشن |

| D | Durability | تغییرات دائمی باشن |



### دستورات

```sql

BEGIN TRANSACTION    -- شروع

COMMIT               -- تایید و ذخیره

ROLLBACK             -- برگشت و لغو

```



### مثال ساده

```sql

BEGIN TRANSACTION;

 

UPDATE Products SET Stock = Stock - 1 WHERE ProductID = 1;

UPDATE Products SET Stock = Stock + 1 WHERE ProductID = 2;

 

-- همه چی اوکیه؟

COMMIT;

 

-- مشکل داریم؟

-- ROLLBACK;

```



### مثال کامل: ثبت سفارش

```sql

BEGIN TRY

    BEGIN TRANSACTION;

 

    DECLARE @OrderID INT;

 

    -- 1. ثبت سفارش

    INSERT INTO Orders (CustomerID, OrderNumber, TotalAmount, ShippingAddress)

    VALUES (1, 'ORD-001', 80000000, N'تهران');

 

    SET @OrderID = SCOPE_IDENTITY();

 

    -- 2. ثبت آیتم‌ها

    INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)

    VALUES (@OrderID, 1, 1, 75000000);

 

    -- 3. کم کردن موجودی

    UPDATE Products SET Stock = Stock - 1 WHERE ProductID = 1;

 

    -- بررسی موجودی منفی

    IF EXISTS (SELECT 1 FROM Products WHERE ProductID = 1 AND Stock < 0)

    BEGIN

        THROW 50001, N'موجودی کافی نیست!', 1;

    END

 

    COMMIT;

    PRINT N'سفارش ثبت شد!';

 

END TRY

BEGIN CATCH

    ROLLBACK;

    PRINT N'خطا: ' + ERROR_MESSAGE();

END CATCH

```



---



## 2. Trigger



### چیه؟

کدی که **خودکار** اجرا میشه وقتی یه اتفاق میفته (INSERT, UPDATE, DELETE).



### انواع

- `AFTER`: بعد از عملیات

- `INSTEAD OF`: به جای عملیات



### جداول مخصوص تریگر

- `INSERTED`: ردیف‌های جدید/تغییریافته

- `DELETED`: ردیف‌های قدیمی/حذف‌شده



### مثال: لاگ تغییرات قیمت

```sql

-- اول جدول لاگ بساز

CREATE TABLE PriceLog (

    LogID INT IDENTITY PRIMARY KEY,

    ProductID INT,

    OldPrice DECIMAL(18,2),

    NewPrice DECIMAL(18,2),

    ChangedAt DATETIME DEFAULT GETDATE()

);

GO

 

-- بعد تریگر

CREATE TRIGGER trg_PriceChange

ON Products

AFTER UPDATE

AS

BEGIN

    INSERT INTO PriceLog (ProductID, OldPrice, NewPrice)

    SELECT i.ProductID, d.Price, i.Price

    FROM INSERTED i

    INNER JOIN DELETED d ON i.ProductID = d.ProductID

    WHERE i.Price <> d.Price;

END;

GO

```



### مثال: کم کردن موجودی خودکار

```sql

CREATE TRIGGER trg_UpdateStock

ON OrderItems

AFTER INSERT

AS

BEGIN

    UPDATE p

    SET p.Stock = p.Stock - i.Quantity

    FROM Products p

    INNER JOIN INSERTED i ON p.ProductID = i.ProductID;

 

    -- اگه موجودی منفی شد، خطا بده

    IF EXISTS (SELECT 1 FROM Products WHERE Stock < 0)

    BEGIN

        RAISERROR(N'موجودی کافی نیست!', 16, 1);

        ROLLBACK;

    END

END;

GO

```



### مثال: جلوگیری از حذف واقعی

```sql

CREATE TRIGGER trg_SoftDelete

ON Products

INSTEAD OF DELETE

AS

BEGIN

    -- به جای حذف، فقط غیرفعال کن

    UPDATE p

    SET IsActive = 0

    FROM Products p

    INNER JOIN DELETED d ON p.ProductID = d.ProductID;

 

    PRINT N'محصولات غیرفعال شدن (حذف نشدن)';

END;

GO

```



### مدیریت تریگر

```sql

-- غیرفعال

DISABLE TRIGGER trg_PriceChange ON Products;

 

-- فعال

ENABLE TRIGGER trg_PriceChange ON Products;

 

-- حذف

DROP TRIGGER trg_PriceChange;

```



---



## 3. Stored Procedure



### چیه؟

یه تابع که توی دیتابیس ذخیره میشه و می‌تونی صداش بزنی.



### چرا؟

- سریع‌تره (یه بار کامپایل میشه)

- امن‌تره

- کد تمیزتر



### مثال ساده

```sql

CREATE PROCEDURE sp_GetActiveProducts

AS

BEGIN

    SELECT ProductID, ProductName, Price, Stock

    FROM Products

    WHERE IsActive = 1 AND Stock > 0;

END;

GO

 

-- اجرا

EXEC sp_GetActiveProducts;

```



### با پارامتر ورودی

```sql

CREATE PROCEDURE sp_GetProductsByCategory

    @CategoryID INT

AS

BEGIN

    SELECT ProductName, Price, Stock

    FROM Products

    WHERE CategoryID = @CategoryID AND IsActive = 1;

END;

GO

 

-- اجرا

EXEC sp_GetProductsByCategory @CategoryID = 1;

```



### با پارامتر خروجی

```sql

CREATE PROCEDURE sp_GetProductCount

    @CategoryID INT,

    @Count INT OUTPUT

AS

BEGIN

    SELECT @Count = COUNT(*)

    FROM Products

    WHERE CategoryID = @CategoryID AND IsActive = 1;

END;

GO

 

-- اجرا

DECLARE @Result INT;

EXEC sp_GetProductCount @CategoryID = 1, @Count = @Result OUTPUT;

SELECT @Result AS ProductCount;

```



### مثال کامل: ثبت سفارش

```sql

CREATE PROCEDURE sp_CreateOrder

    @CustomerID INT,

    @ProductID INT,

    @Quantity INT,

    @OrderID INT OUTPUT,

    @Message NVARCHAR(200) OUTPUT

AS

BEGIN

    BEGIN TRY

        BEGIN TRANSACTION;

 

        DECLARE @Price DECIMAL(18,2), @Stock INT;

 

        -- چک موجودی

        SELECT @Price = Price, @Stock = Stock

        FROM Products WHERE ProductID = @ProductID;

 

        IF @Stock < @Quantity

        BEGIN

            SET @Message = N'موجودی کافی نیست';

            SET @OrderID = 0;

            ROLLBACK;

            RETURN;

        END

 

        -- ثبت سفارش

        INSERT INTO Orders (CustomerID, OrderNumber, TotalAmount, ShippingAddress)

        VALUES (@CustomerID, 'ORD-' + CAST(NEWID() AS VARCHAR(8)), @Price * @Quantity, N'آدرس');

 

        SET @OrderID = SCOPE_IDENTITY();

 

        -- ثبت آیتم

        INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)

        VALUES (@OrderID, @ProductID, @Quantity, @Price);

 

        -- کم کردن موجودی

        UPDATE Products SET Stock = Stock - @Quantity WHERE ProductID = @ProductID;

 

        COMMIT;

        SET @Message = N'سفارش ثبت شد';

 

    END TRY

    BEGIN CATCH

        ROLLBACK;

        SET @Message = ERROR_MESSAGE();

        SET @OrderID = 0;

    END CATCH

END;

GO

```



---



## 4. Function



### فرق با Procedure

| | Function | Procedure |

|---|----------|-----------|

| خروجی | حتماً یه مقدار برمی‌گردونه | اختیاری |

| استفاده در SELECT | میشه | نمیشه |

| تغییر داده | نمیشه | میشه |



### Scalar Function (یه مقدار برگردونه)

```sql

CREATE FUNCTION fn_GetFinalPrice(@ProductID INT)

RETURNS DECIMAL(18,2)

AS

BEGIN

    DECLARE @Price DECIMAL(18,2);

 

    SELECT @Price = ISNULL(DiscountPrice, Price)

    FROM Products

    WHERE ProductID = @ProductID;

 

    RETURN @Price;

END;

GO

 

-- استفاده

SELECT

    ProductName,

    Price,

    dbo.fn_GetFinalPrice(ProductID) AS FinalPrice

FROM Products;

```



### Table Function (جدول برگردونه)

```sql

CREATE FUNCTION fn_GetProductsByCategory(@CategoryID INT)

RETURNS TABLE

AS

RETURN

(

    SELECT ProductID, ProductName, Price, Stock

    FROM Products

    WHERE CategoryID = @CategoryID AND IsActive = 1

);

GO

 

-- استفاده

SELECT * FROM fn_GetProductsByCategory(1);

```



### مثال: محاسبه درصد تخفیف

```sql

CREATE FUNCTION fn_GetDiscountPercent(@ProductID INT)

RETURNS DECIMAL(5,2)

AS

BEGIN

    DECLARE @Percent DECIMAL(5,2);

 

    SELECT @Percent =

        CASE

            WHEN DiscountPrice IS NULL THEN 0

            ELSE (Price - DiscountPrice) * 100.0 / Price

        END

    FROM Products

    WHERE ProductID = @ProductID;

 

    RETURN ISNULL(@Percent, 0);

END;

GO

```



---



## 5. Index



### چیه؟

مثل فهرست کتاب! به جای اینکه کل کتاب رو بگردی، از فهرست پیدا می‌کنی.



### انواع

| نوع | توضیح |

|-----|-------|

| Clustered | ترتیب فیزیکی داده‌ها (فقط یکی) |

| Non-Clustered | فهرست جداگانه (چندتا میشه) |



### ساختن ایندکس

```sql

-- روی نام محصول

CREATE INDEX IX_Products_Name ON Products(ProductName);

 

-- روی دسته‌بندی

CREATE INDEX IX_Products_Category ON Products(CategoryID);

 

-- یکتا

CREATE UNIQUE INDEX IX_Products_SKU ON Products(SKU);

 

-- ترکیبی

CREATE INDEX IX_Orders_Customer_Date ON Orders(CustomerID, OrderDate DESC);

```



### ایندکس Filtered (با شرط)

```sql

-- فقط محصولات فعال

CREATE INDEX IX_Products_Active

ON Products(ProductName, Price)

WHERE IsActive = 1;

```



### حذف و بازسازی

```sql

-- حذف

DROP INDEX IX_Products_Name ON Products;

 

-- بازسازی

ALTER INDEX IX_Products_Category ON Products REBUILD;

```



### کی ایندکس بذارم؟

- ستون‌هایی که زیاد توشون جستجو میشه

- ستون‌هایی که توی JOIN استفاده میشن

- ستون‌هایی که توی ORDER BY هستن



---



## 6. Cursor



### چیه؟

برای پردازش **ردیف به ردیف**.



### هشدار!

کند هستن! اگه میشه از راه‌های دیگه استفاده کن.



### مثال

```sql

DECLARE @ProductID INT, @Name NVARCHAR(200), @Stock INT;

 

DECLARE product_cursor CURSOR FOR

SELECT ProductID, ProductName, Stock

FROM Products

WHERE Stock < 10;

 

OPEN product_cursor;

 

FETCH NEXT FROM product_cursor INTO @ProductID, @Name, @Stock;

 

WHILE @@FETCH_STATUS = 0

BEGIN

    PRINT N'هشدار: ' + @Name + N' - موجودی: ' + CAST(@Stock AS VARCHAR);

 

    FETCH NEXT FROM product_cursor INTO @ProductID, @Name, @Stock;

END

 

CLOSE product_cursor;

DEALLOCATE product_cursor;

```



---



## 7. CTE



### چیه؟

CTE = Common Table Expression

یه جدول موقت که توی همون کوئری ازش استفاده می‌کنی.



### ساختار

```sql

WITH نام_CTE AS (

    SELECT ...

)

SELECT * FROM نام_CTE;

```



### مثال ساده

```sql

WITH TopProducts AS (

    SELECT ProductID, ProductName, Price

    FROM Products

    WHERE IsActive = 1

)

SELECT * FROM TopProducts

WHERE Price > 10000000;

```



### CTE چندتایی

```sql

WITH

ProductSales AS (

    SELECT ProductID, SUM(Quantity) AS TotalSold

    FROM OrderItems

    GROUP BY ProductID

),

ProductRatings AS (

    SELECT ProductID, AVG(CAST(Rating AS DECIMAL)) AS AvgRating

    FROM Reviews

    GROUP BY ProductID

)

SELECT

    p.ProductName,

    ISNULL(s.TotalSold, 0) AS TotalSold,

    ISNULL(r.AvgRating, 0) AS AvgRating

FROM Products p

LEFT JOIN ProductSales s ON p.ProductID = s.ProductID

LEFT JOIN ProductRatings r ON p.ProductID = r.ProductID;

```



### CTE بازگشتی (سلسله‌مراتب)

```sql

WITH CategoryTree AS (

    -- شروع: دسته‌های اصلی

    SELECT CategoryID, CategoryName, ParentID, 0 AS Level

    FROM Categories

    WHERE ParentID IS NULL

 

    UNION ALL

 

    -- ادامه: زیردسته‌ها

    SELECT c.CategoryID, c.CategoryName, c.ParentID, ct.Level + 1

    FROM Categories c

    INNER JOIN CategoryTree ct ON c.ParentID = ct.CategoryID

)

SELECT * FROM CategoryTree;

```



---



## 8. Window Functions



### چیه؟

محاسبه روی یه گروه از ردیف‌ها، بدون اینکه گروه‌بندی کنی.



### توابع اصلی



| تابع | کار |

|------|-----|

| `ROW_NUMBER()` | شماره ردیف |

| `RANK()` | رتبه (با پرش) |

| `DENSE_RANK()` | رتبه (بدون پرش) |

| `LAG()` | مقدار ردیف قبلی |

| `LEAD()` | مقدار ردیف بعدی |

| `SUM() OVER()` | مجموع تجمعی |



### ROW_NUMBER - شماره‌گذاری

```sql

SELECT

    ROW_NUMBER() OVER (ORDER BY Price DESC) AS RowNum,

    ProductName,

    Price

FROM Products;

```



### شماره‌گذاری به تفکیک دسته

```sql

SELECT

    ROW_NUMBER() OVER (PARTITION BY CategoryID ORDER BY Price DESC) AS RankInCategory,

    ProductName,

    CategoryID,

    Price

FROM Products;

```



### RANK vs DENSE_RANK

```sql

SELECT

    ProductName,

    Price,

    RANK() OVER (ORDER BY Price DESC) AS Rank,        -- 1,2,2,4

    DENSE_RANK() OVER (ORDER BY Price DESC) AS Dense  -- 1,2,2,3

FROM Products;

```



### LAG و LEAD

```sql

-- مقایسه با روز قبل

SELECT

    CAST(OrderDate AS DATE) AS SaleDate,

    SUM(TotalAmount) AS DailySales,

    LAG(SUM(TotalAmount)) OVER (ORDER BY CAST(OrderDate AS DATE)) AS PrevDaySales

FROM Orders

GROUP BY CAST(OrderDate AS DATE);

```



### مجموع تجمعی (Running Total)

```sql

SELECT

    OrderDate,

    TotalAmount,

    SUM(TotalAmount) OVER (ORDER BY OrderDate) AS RunningTotal

FROM Orders;

```



### صفحه‌بندی با ROW_NUMBER

```sql

WITH PaginatedProducts AS (

    SELECT

        ROW_NUMBER() OVER (ORDER BY ProductID) AS RowNum,

        ProductID,

        ProductName,

        Price

    FROM Products

)

SELECT * FROM PaginatedProducts

WHERE RowNum BETWEEN 11 AND 20;  -- صفحه 2

```



---



## خلاصه



| مبحث | کار | نکته |

|------|-----|------|

| **Transaction** | همه یا هیچکدوم | BEGIN/COMMIT/ROLLBACK |

| **Trigger** | اجرای خودکار | AFTER / INSTEAD OF |

| **Stored Procedure** | تابع ذخیره‌شده | EXEC برای اجرا |

| **Function** | تابع با خروجی | Scalar / Table |

| **Index** | سریع‌تر کردن جستجو | Clustered / Non-Clustered |

| **Cursor** | پردازش ردیف به ردیف | کنده، کمتر استفاده کن |

| **CTE** | جدول موقت | WITH...AS |

| **Window Functions** | محاسبه روی پنجره | OVER (ORDER BY...) |



---



**بخش قبلی:** [کوئری‌های پیچیده](03-complex-queries.md)

**چیت‌شیت:** [خلاصه همه چیز](05-cheatsheet.md)



</div>

