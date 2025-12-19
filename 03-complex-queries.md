<div dir="rtl">



# کوئری‌های پیچیده - گروه‌بندی، اتصال جداول و خیلی چیزای دیگه



---



## این بخش چیه؟



تا الان یاد گرفتی چطور از یه جدول داده بخونی. ولی توی دنیای واقعی:

- می‌خوای بدونی **مجموع فروش** چقدره

- می‌خوای اطلاعات **چند جدول** رو باهم ببینی

- می‌خوای **گروه‌بندی** کنی و آمار بگیری



بزن بریم!



---



## فهرست



1. [توابع تجمیعی (COUNT, SUM, AVG, ...)](#1-توابع-تجمیعی)

2. [GROUP BY - گروه‌بندی](#2-group-by)

3. [HAVING - فیلتر روی گروه‌ها](#3-having)

4. [JOIN - اتصال جداول](#4-join)

5. [UNION - ترکیب نتایج](#5-union)

6. [Subquery - کوئری داخل کوئری](#6-subquery)

7. [VIEW - نما](#7-view)

8. [TRY...CATCH - مدیریت خطا](#8-try-catch)



---



## 1. توابع تجمیعی



روی یه عالمه ردیف محاسبه می‌کنن و یه جواب میدن.



| تابع | کار |

|------|-----|

| `COUNT(*)` | تعداد ردیف‌ها |

| `SUM(column)` | مجموع |

| `AVG(column)` | میانگین |

| `MAX(column)` | بیشترین |

| `MIN(column)` | کمترین |



### COUNT - شمردن

```sql

-- چند تا محصول داریم؟

SELECT COUNT(*) AS TotalProducts FROM Products;

 

-- چند تا محصول فعال داریم؟

SELECT COUNT(*) AS ActiveProducts FROM Products WHERE IsActive = 1;

 

-- چند تا دسته‌بندی متفاوت؟

SELECT COUNT(DISTINCT CategoryID) AS CategoryCount FROM Products;

```



### SUM - مجموع

```sql

-- کل موجودی انبار

SELECT SUM(Stock) AS TotalStock FROM Products;

 

-- مجموع فروش امروز

SELECT SUM(TotalAmount) AS TodaySales

FROM Orders

WHERE CAST(OrderDate AS DATE) = CAST(GETDATE() AS DATE);

```



### AVG - میانگین

```sql

-- میانگین قیمت محصولات

SELECT AVG(Price) AS AvgPrice FROM Products;

 

-- میانگین مبلغ سفارشات

SELECT AVG(TotalAmount) AS AvgOrderAmount FROM Orders;

```



### MAX و MIN

```sql

-- گرون‌ترین و ارزون‌ترین

SELECT

    MAX(Price) AS MostExpensive,

    MIN(Price) AS Cheapest

FROM Products;

 

-- اولین و آخرین سفارش

SELECT

    MIN(OrderDate) AS FirstOrder,

    MAX(OrderDate) AS LastOrder

FROM Orders;

```



### ترکیب همه باهم

```sql

SELECT

    COUNT(*) AS 'تعداد کل',

    SUM(Stock) AS 'موجودی کل',

    AVG(Price) AS 'میانگین قیمت',

    MAX(Price) AS 'گرون‌ترین',

    MIN(Price) AS 'ارزون‌ترین'

FROM Products

WHERE IsActive = 1;

```



---



## 2. GROUP BY



وقتی می‌خوای **به تفکیک** آمار بگیری.



### ساختار

```sql

SELECT ستون_گروه‌بندی, تابع_تجمیعی(ستون)

FROM جدول

GROUP BY ستون_گروه‌بندی;

```



### تعداد محصولات هر دسته

```sql

SELECT CategoryID, COUNT(*) AS ProductCount

FROM Products

GROUP BY CategoryID;

```



### فروش هر مشتری

```sql

SELECT

    CustomerID,

    COUNT(*) AS OrderCount,

    SUM(TotalAmount) AS TotalSpent

FROM Orders

GROUP BY CustomerID;

```



### فروش روزانه

```sql

SELECT

    CAST(OrderDate AS DATE) AS SaleDate,

    COUNT(*) AS OrderCount,

    SUM(TotalAmount) AS DailySales

FROM Orders

GROUP BY CAST(OrderDate AS DATE)

ORDER BY SaleDate DESC;

```



### فروش ماهانه

```sql

SELECT

    YEAR(OrderDate) AS SaleYear,

    MONTH(OrderDate) AS SaleMonth,

    COUNT(*) AS OrderCount,

    SUM(TotalAmount) AS MonthlySales

FROM Orders

GROUP BY YEAR(OrderDate), MONTH(OrderDate)

ORDER BY SaleYear DESC, SaleMonth DESC;

```



### محصولات بر اساس رنج قیمت

```sql

SELECT

    CASE

        WHEN Price < 5000000 THEN N'ارزون'

        WHEN Price < 20000000 THEN N'متوسط'

        WHEN Price < 50000000 THEN N'گرون'

        ELSE N'لوکس'

    END AS PriceRange,

    COUNT(*) AS ProductCount

FROM Products

GROUP BY

    CASE

        WHEN Price < 5000000 THEN N'ارزون'

        WHEN Price < 20000000 THEN N'متوسط'

        WHEN Price < 50000000 THEN N'گرون'

        ELSE N'لوکس'

    END;

```



---



## 3. HAVING



فیلتر کردن **بعد از** گروه‌بندی.



فرق WHERE و HAVING:

- **WHERE**: قبل از گروه‌بندی اعمال میشه

- **HAVING**: بعد از گروه‌بندی اعمال میشه



### دسته‌هایی که بیش از 5 محصول دارن

```sql

SELECT CategoryID, COUNT(*) AS ProductCount

FROM Products

GROUP BY CategoryID

HAVING COUNT(*) > 5;

```



### مشتری‌های VIP (بیش از 10 میلیون خرید)

```sql

SELECT

    CustomerID,

    COUNT(*) AS OrderCount,

    SUM(TotalAmount) AS TotalSpent

FROM Orders

WHERE Status <> 'Cancelled'

GROUP BY CustomerID

HAVING SUM(TotalAmount) > 10000000

ORDER BY TotalSpent DESC;

```



### محصولات پرفروش (بیش از 100 فروش)

```sql

SELECT

    ProductID,

    SUM(Quantity) AS TotalSold

FROM OrderItems

GROUP BY ProductID

HAVING SUM(Quantity) > 100

ORDER BY TotalSold DESC;

```



### ترکیب WHERE و HAVING

```sql

SELECT

    CategoryID,

    COUNT(*) AS ProductCount,

    AVG(Price) AS AvgPrice

FROM Products

WHERE IsActive = 1            -- فیلتر قبل از گروه‌بندی

GROUP BY CategoryID

HAVING AVG(Price) > 10000000  -- فیلتر بعد از گروه‌بندی

ORDER BY AvgPrice DESC;

```



---



## 4. JOIN



وقتی می‌خوای اطلاعات **چند جدول** رو باهم ببینی.



### انواع JOIN



```

INNER JOIN: فقط مشترک‌ها ████

LEFT JOIN:  همه از چپ + مشترک ████████

RIGHT JOIN: مشترک + همه از راست     ████████

FULL JOIN:  همه از هر دو طرف ████████████████

```



### INNER JOIN - فقط مشترک‌ها

```sql

-- محصولات با نام دسته‌بندیشون

SELECT

    p.ProductName,

    p.Price,

    c.CategoryName

FROM Products p

INNER JOIN Categories c ON p.CategoryID = c.CategoryID;

```



```sql

-- سفارشات با اطلاعات مشتری

SELECT

    o.OrderNumber,

    o.OrderDate,

    o.TotalAmount,

    c.FirstName + ' ' + c.LastName AS CustomerName,

    c.Phone

FROM Orders o

INNER JOIN Customers c ON o.CustomerID = c.CustomerID;

```



### LEFT JOIN - همه از جدول چپ

```sql

-- همه دسته‌ها با تعداد محصولاتشون (حتی دسته‌های خالی)

SELECT

    c.CategoryName,

    COUNT(p.ProductID) AS ProductCount

FROM Categories c

LEFT JOIN Products p ON c.CategoryID = p.CategoryID

GROUP BY c.CategoryID, c.CategoryName;

```



```sql

-- همه مشتری‌ها با تعداد سفارشاتشون

SELECT

    c.FirstName + ' ' + c.LastName AS CustomerName,

    COUNT(o.OrderID) AS OrderCount,

    ISNULL(SUM(o.TotalAmount), 0) AS TotalSpent

FROM Customers c

LEFT JOIN Orders o ON c.CustomerID = o.CustomerID

GROUP BY c.CustomerID, c.FirstName, c.LastName;

```



### JOIN چندتایی

```sql

-- جزئیات کامل سفارش

SELECT

    o.OrderNumber,

    c.FirstName + ' ' + c.LastName AS Customer,

    p.ProductName,

    cat.CategoryName,

    oi.Quantity,

    oi.UnitPrice,

    oi.Quantity * oi.UnitPrice AS LineTotal

FROM Orders o

INNER JOIN Customers c ON o.CustomerID = c.CustomerID

INNER JOIN OrderItems oi ON o.OrderID = oi.OrderID

INNER JOIN Products p ON oi.ProductID = p.ProductID

INNER JOIN Categories cat ON p.CategoryID = cat.CategoryID

WHERE o.OrderID = 1;

```



### SELF JOIN - جدول با خودش

```sql

-- دسته‌بندی‌ها با والدشون

SELECT

    child.CategoryName AS Category,

    parent.CategoryName AS ParentCategory

FROM Categories child

LEFT JOIN Categories parent ON child.ParentID = parent.CategoryID;

```



---



## 5. UNION



ترکیب نتایج چند SELECT.



قوانین:

- تعداد ستون‌ها باید برابر باشه

- نوع ستون‌ها باید سازگار باشه



### UNION (بدون تکرار)

```sql

SELECT Name, Email, 'Customer' AS Type FROM Customers

UNION

SELECT Name, Email, 'Employee' AS Type FROM Employees;

```



### UNION ALL (با تکرار)

```sql

-- همه تراکنش‌ها

SELECT OrderID, TotalAmount, 'Sale' AS Type FROM Orders

UNION ALL

SELECT PaymentID, Amount, 'Payment' AS Type FROM Payments;

```



---



## 6. Subquery



کوئری داخل کوئری.



### در WHERE

```sql

-- محصولات گرون‌تر از میانگین

SELECT ProductName, Price

FROM Products

WHERE Price > (SELECT AVG(Price) FROM Products);

```



```sql

-- مشتری‌هایی که سفارش دادن

SELECT * FROM Customers

WHERE CustomerID IN (SELECT DISTINCT CustomerID FROM Orders);

```



### در SELECT

```sql

-- محصولات با تعداد نظراتشون

SELECT

    ProductName,

    Price,

    (SELECT COUNT(*) FROM Reviews r WHERE r.ProductID = p.ProductID) AS ReviewCount

FROM Products p;

```



### EXISTS

```sql

-- مشتری‌هایی که حداقل یه سفارش دارن

SELECT * FROM Customers c

WHERE EXISTS (

    SELECT 1 FROM Orders o WHERE o.CustomerID = c.CustomerID

);

 

-- محصولاتی که هیچوقت فروش نرفتن

SELECT * FROM Products p

WHERE NOT EXISTS (

    SELECT 1 FROM OrderItems oi WHERE oi.ProductID = p.ProductID

);

```



---



## 7. VIEW



یه کوئری ذخیره‌شده که مثل جدول ازش استفاده می‌کنی.



### ساختن VIEW

```sql

CREATE VIEW vw_ActiveProducts AS

SELECT ProductID, ProductName, Price, Stock

FROM Products

WHERE IsActive = 1 AND Stock > 0;

```



### استفاده از VIEW

```sql

SELECT * FROM vw_ActiveProducts;

SELECT * FROM vw_ActiveProducts WHERE Price < 10000000;

```



### VIEW با JOIN

```sql

CREATE VIEW vw_ProductDetails AS

SELECT

    p.ProductID,

    p.ProductName,

    p.Price,

    p.Stock,

    c.CategoryName

FROM Products p

INNER JOIN Categories c ON p.CategoryID = c.CategoryID;

```



### VIEW داشبورد

```sql

CREATE VIEW vw_Dashboard AS

SELECT

    (SELECT COUNT(*) FROM Products WHERE IsActive = 1) AS ActiveProducts,

    (SELECT COUNT(*) FROM Products WHERE Stock < 10) AS LowStockProducts,

    (SELECT COUNT(*) FROM Customers) AS TotalCustomers,

    (SELECT COUNT(*) FROM Orders WHERE Status = 'Pending') AS PendingOrders,

    (SELECT SUM(TotalAmount) FROM Orders WHERE CAST(OrderDate AS DATE) = CAST(GETDATE() AS DATE)) AS TodaySales;

```



### حذف VIEW

```sql

DROP VIEW IF EXISTS vw_ActiveProducts;

```



---



## 8. TRY CATCH



برای وقتی که ممکنه خطا پیش بیاد.



### ساختار

```sql

BEGIN TRY

    -- کدی که ممکنه خطا بده

END TRY

BEGIN CATCH

    -- مدیریت خطا

END CATCH

```



### مثال ساده

```sql

BEGIN TRY

    INSERT INTO Products (CategoryID, ProductName, SKU, Price, Stock)

    VALUES (999, N'تست', 'TEST-001', 100000, 10);  -- CategoryID 999 وجود نداره

 

    PRINT N'محصول اضافه شد';

END TRY

BEGIN CATCH

    PRINT N'خطا!';

    PRINT N'شماره خطا: ' + CAST(ERROR_NUMBER() AS VARCHAR);

    PRINT N'پیام خطا: ' + ERROR_MESSAGE();

END CATCH

```



### توابع خطا



| تابع | چی برمی‌گردونه |

|------|---------------|

| `ERROR_NUMBER()` | شماره خطا |

| `ERROR_MESSAGE()` | پیام خطا |

| `ERROR_LINE()` | خط خطا |



### پرتاب خطای سفارشی

```sql

BEGIN TRY

    DECLARE @Stock INT;

    SELECT @Stock = Stock FROM Products WHERE ProductID = 1;

 

    IF @Stock < 5

    BEGIN

        THROW 50001, N'موجودی کافی نیست!', 1;

    END

 

    PRINT N'موجودی کافیه';

END TRY

BEGIN CATCH

    PRINT ERROR_MESSAGE();

END CATCH

```



---



## یه مثال کامل: گزارش فروش



```sql

-- ساخت VIEW برای گزارش

CREATE VIEW vw_SalesReport AS

SELECT

    CAST(o.OrderDate AS DATE) AS SaleDate,

    c.CategoryName,

    COUNT(DISTINCT o.OrderID) AS OrderCount,

    SUM(oi.Quantity) AS TotalQuantity,

    SUM(oi.Quantity * oi.UnitPrice) AS Revenue

FROM Orders o

INNER JOIN OrderItems oi ON o.OrderID = oi.OrderID

INNER JOIN Products p ON oi.ProductID = p.ProductID

INNER JOIN Categories c ON p.CategoryID = c.CategoryID

WHERE o.Status <> 'Cancelled'

GROUP BY CAST(o.OrderDate AS DATE), c.CategoryID, c.CategoryName;

GO

 

-- استفاده از VIEW

SELECT * FROM vw_SalesReport

ORDER BY SaleDate DESC, Revenue DESC;

 

-- فروش هفته اخیر

SELECT

    CategoryName,

    SUM(OrderCount) AS TotalOrders,

    SUM(Revenue) AS TotalRevenue

FROM vw_SalesReport

WHERE SaleDate >= DATEADD(DAY, -7, GETDATE())

GROUP BY CategoryName

ORDER BY TotalRevenue DESC;

```



---



## خلاصه



| مبحث | کار | نکته |

|------|-----|------|

| **توابع تجمیعی** | COUNT, SUM, AVG, MAX, MIN | محاسبه روی کل داده‌ها |

| **GROUP BY** | گروه‌بندی | آمار به تفکیک |

| **HAVING** | فیلتر گروه‌ها | بعد از GROUP BY |

| **INNER JOIN** | اتصال جداول | فقط مشترک‌ها |

| **LEFT JOIN** | اتصال جداول | همه از چپ + مشترک |

| **UNION** | ترکیب نتایج | تعداد ستون‌ها برابر باشه |

| **Subquery** | کوئری در کوئری | در WHERE یا SELECT |

| **VIEW** | کوئری ذخیره‌شده | مثل جدول استفاده کن |

| **TRY...CATCH** | مدیریت خطا | |



---



**بخش قبلی:** [عملیات CRUD](02-crud-operations.md)

**بخش بعدی:** [مفاهیم پیشرفته](04-professional-concepts.md) - Transaction، Trigger، Stored Procedure و خیلی چیزای حرفه‌ای!



</div>