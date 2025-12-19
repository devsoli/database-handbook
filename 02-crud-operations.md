<div dir="rtl">



# عملیات CRUD - اضافه کردن، خوندن، ویرایش، حذف



---



## CRUD یعنی چی؟



CRUD یه کلمه مخففه:

- **C**reate = ساختن (INSERT)

- **R**ead = خوندن (SELECT)

- **U**pdate = ویرایش (UPDATE)

- **D**elete = حذف (DELETE)



این چهارتا کار اصلی هستن که با داده‌ها انجام میدی.



---



## فهرست



1. [INSERT - اضافه کردن داده](#1-insert)

2. [SELECT - خوندن داده](#2-select)

3. [UPDATE - ویرایش داده](#3-update)

4. [DELETE - حذف داده](#4-delete)

5. [WHERE - فیلتر کردن](#5-where)

6. [ORDER BY - مرتب‌سازی](#6-order-by)

7. [LIKE - جستجوی متنی](#7-like)



---



## 1. INSERT



### اضافه کردن یه ردیف

```sql

INSERT INTO Categories (CategoryName, Description)

VALUES (N'موبایل', N'انواع گوشی موبایل');

```



### اضافه کردن چند ردیف همزمان

```sql

INSERT INTO Categories (CategoryName, Description)

VALUES

    (N'لپ‌تاپ', N'انواع لپ‌تاپ'),

    (N'تبلت', N'انواع تبلت'),

    (N'لوازم جانبی', N'کیبورد، موس، هدفون');

```



### اضافه کردن محصول

```sql

INSERT INTO Products (CategoryID, ProductName, SKU, Price, Stock)

VALUES (1, N'آیفون 15', 'IPHONE-15', 75000000, 50);

```



### اضافه کردن مشتری

```sql

INSERT INTO Customers (FirstName, LastName, Email, Phone, City)

VALUES (N'علی', N'محمدی', 'ali@email.com', '09121234567', N'تهران');

```



### گرفتن ID ردیف جدید

وقتی یه ردیف اضافه می‌کنی، می‌خوای بدونی چه ID ای گرفت؟



```sql

INSERT INTO Orders (CustomerID, OrderNumber, TotalAmount, ShippingAddress)

VALUES (1, 'ORD-001', 75000000, N'تهران، خیابان آزادی');

 

SELECT SCOPE_IDENTITY() AS NewOrderID;

```



### کپی داده از جدول دیگه

```sql

INSERT INTO ProductsArchive (ProductID, ProductName, Price)

SELECT ProductID, ProductName, Price

FROM Products

WHERE Stock = 0;

```



---



## 2. SELECT



### خوندن همه چی

```sql

SELECT * FROM Products;

```



### خوندن ستون‌های خاص

```sql

SELECT ProductName, Price, Stock FROM Products;

```



### تغییر نام ستون در خروجی (اسم مستعار)

```sql

SELECT

    ProductName AS 'اسم محصول',

    Price AS 'قیمت',

    Stock AS 'موجودی'

FROM Products;

```



### محاسبه توی SELECT

```sql

SELECT

    ProductName,

    Price,

    Price * 0.09 AS Tax,          -- مالیات 9%

    Price * 1.09 AS TotalPrice    -- قیمت با مالیات

FROM Products;

```



### DISTINCT - حذف تکراری‌ها

```sql

-- لیست شهرهای مشتری‌ها (بدون تکرار)

SELECT DISTINCT City FROM Customers;

```



### TOP - فقط چند تای اول

```sql

-- 10 محصول اول

SELECT TOP 10 * FROM Products;

 

-- 5 تا گرون‌ترین

SELECT TOP 5 ProductName, Price

FROM Products

ORDER BY Price DESC;

 

-- 10 درصد اول

SELECT TOP 10 PERCENT * FROM Products;

```



---



## 3. UPDATE



### ویرایش یه ردیف

```sql

UPDATE Products

SET Price = 80000000

WHERE ProductID = 1;

```



**خیلی مهم:** همیشه WHERE بذار! وگرنه همه ردیف‌ها عوض میشن!



### ویرایش چند فیلد

```sql

UPDATE Products

SET

    Price = 80000000,

    Stock = 40,

    IsActive = 1

WHERE ProductID = 1;

```



### ویرایش با محاسبه

```sql

-- 10% افزایش قیمت همه محصولات موبایل

UPDATE Products

SET Price = Price * 1.10

WHERE CategoryID = 1;

 

-- کم کردن موجودی

UPDATE Products

SET Stock = Stock - 1

WHERE ProductID = 5 AND Stock > 0;

```



### ویرایش وضعیت سفارش

```sql

UPDATE Orders

SET Status = 'Shipped'

WHERE OrderID = 1;

```



### غیرفعال کردن محصولات بدون موجودی

```sql

UPDATE Products

SET IsActive = 0

WHERE Stock = 0;

```



---



## 4. DELETE



### حذف یه ردیف

```sql

DELETE FROM Products WHERE ProductID = 100;

```



**خیلی مهم:** همیشه WHERE بذار! وگرنه همه چی پاک میشه!



### حذف با شرط

```sql

-- حذف محصولات غیرفعال

DELETE FROM Products WHERE IsActive = 0;

 

-- حذف سفارشات لغوشده قدیمی‌تر از یک سال

DELETE FROM Orders

WHERE Status = 'Cancelled'

AND OrderDate < DATEADD(YEAR, -1, GETDATE());

```



### فرق DELETE و TRUNCATE



| | DELETE | TRUNCATE |

|---|--------|----------|

| شرط | میشه WHERE گذاشت | نه |

| سرعت | کندتر | خیلی سریع |

| شماره خودکار | نگه میداره | ریست میشه |

| برگشت‌پذیری | میشه ROLLBACK کرد | نمیشه |



```sql

-- حذف همه با امکان برگشت

DELETE FROM TempTable;

 

-- حذف همه خیلی سریع (برگشت‌ناپذیر)

TRUNCATE TABLE TempTable;

```



---



## 5. WHERE



WHERE برای فیلتر کردنه. فقط ردیف‌هایی که شرط رو داشته باشن.



### مقایسه‌ها

```sql

WHERE Price = 1000000          -- مساوی

WHERE Price > 1000000          -- بزرگتر

WHERE Price < 1000000          -- کوچکتر

WHERE Price >= 1000000         -- بزرگتر یا مساوی

WHERE Price <= 1000000         -- کوچکتر یا مساوی

WHERE Price <> 1000000         -- نامساوی (مخالف)

```



### AND و OR

```sql

-- هر دو شرط باید برقرار باشه

WHERE Price > 1000000 AND Stock > 0

 

-- یکی از شرط‌ها کافیه

WHERE CategoryID = 1 OR CategoryID = 2

 

-- ترکیبی (پرانتز مهمه!)

WHERE (CategoryID = 1 OR CategoryID = 2) AND IsActive = 1

```



### BETWEEN - بازه

```sql

-- قیمت بین 5 تا 20 میلیون

WHERE Price BETWEEN 5000000 AND 20000000

 

-- سفارشات این ماه

WHERE OrderDate BETWEEN '2024-01-01' AND '2024-01-31'

```



### IN - عضویت در لیست

```sql

-- محصولات دسته 1 یا 2 یا 3

WHERE CategoryID IN (1, 2, 3)

 

-- وضعیت‌های خاص

WHERE Status IN ('Pending', 'Confirmed', 'Processing')

```



### IS NULL / IS NOT NULL

```sql

-- محصولات بدون توضیح

WHERE Description IS NULL

 

-- مشتری‌هایی که شماره دارن

WHERE Phone IS NOT NULL

```



### NOT

```sql

WHERE NOT CategoryID = 1

WHERE CategoryID NOT IN (1, 2, 3)

WHERE Description IS NOT NULL

```



---



## 6. ORDER BY



برای مرتب کردن نتایج.



### صعودی (کم به زیاد) - پیش‌فرض

```sql

SELECT * FROM Products ORDER BY Price;

-- یا صراحتاً:

SELECT * FROM Products ORDER BY Price ASC;

```



### نزولی (زیاد به کم)

```sql

-- گرون‌ترین اول

SELECT * FROM Products ORDER BY Price DESC;

 

-- جدیدترین سفارشات

SELECT * FROM Orders ORDER BY OrderDate DESC;

```



### چند ستون

```sql

-- اول بر اساس دسته، بعد قیمت

SELECT * FROM Products

ORDER BY CategoryID ASC, Price DESC;

```



### ترکیب TOP و ORDER BY

```sql

-- 10 تا گرون‌ترین محصولات موجود

SELECT TOP 10 ProductName, Price

FROM Products

WHERE Stock > 0 AND IsActive = 1

ORDER BY Price DESC;

```



---



## 7. LIKE



برای جستجوی متنی. با % و _ کار می‌کنه.



| علامت | معنی |

|-------|------|

| `%` | هر چند تا کاراکتر (حتی صفر) |

| `_` | دقیقاً یک کاراکتر |



### شامل بودن

```sql

-- محصولاتی که "سامسونگ" دارن

WHERE ProductName LIKE N'%سامسونگ%'

```



### شروع با

```sql

-- محصولاتی که با "آی" شروع میشن

WHERE ProductName LIKE N'آی%'

```



### پایان با

```sql

-- ایمیل‌های جیمیل

WHERE Email LIKE '%@gmail.com'

```



### یک کاراکتر

```sql

-- شماره‌های 0912

WHERE Phone LIKE '0912_______'

 

-- SKU هایی که سه حرف اول و بعد خط تیره دارن

WHERE SKU LIKE '___-%'

```



### NOT LIKE

```sql

-- محصولاتی که سامسونگ نیستن

WHERE ProductName NOT LIKE N'%سامسونگ%'

```



### ترکیب با شرط‌های دیگه

```sql

SELECT * FROM Products

WHERE ProductName LIKE N'%گوشی%'

AND Price < 50000000

AND Stock > 0

ORDER BY Price ASC;

```



---



## یه مثال کامل: ثبت سفارش



```sql

-- 1. ثبت سفارش

INSERT INTO Orders (CustomerID, OrderNumber, TotalAmount, ShippingAddress)

VALUES (1, 'ORD-2024-001', 80000000, N'تهران، میدان آزادی');

 

-- گرفتن ID سفارش

DECLARE @NewOrderID INT = SCOPE_IDENTITY();

 

-- 2. ثبت آیتم‌های سفارش

INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)

VALUES

    (@NewOrderID, 1, 1, 75000000),

    (@NewOrderID, 5, 2, 2500000);

 

-- 3. کم کردن موجودی

UPDATE Products SET Stock = Stock - 1 WHERE ProductID = 1;

UPDATE Products SET Stock = Stock - 2 WHERE ProductID = 5;

 

-- 4. نمایش سفارش

SELECT * FROM Orders WHERE OrderID = @NewOrderID;

SELECT * FROM OrderItems WHERE OrderID = @NewOrderID;

```



---



## خلاصه



| عملیات | دستور | نکته مهم |

|--------|-------|----------|

| **اضافه کردن** | `INSERT INTO ... VALUES` | |

| **خوندن** | `SELECT ... FROM` | * یعنی همه ستون‌ها |

| **ویرایش** | `UPDATE ... SET ... WHERE` | بدون WHERE همه عوض میشه! |

| **حذف** | `DELETE FROM ... WHERE` | بدون WHERE همه پاک میشه! |

| **فیلتر** | `WHERE` | با AND, OR, IN, BETWEEN |

| **مرتب‌سازی** | `ORDER BY` | ASC صعودی، DESC نزولی |

| **جستجو** | `LIKE` | % چند کاراکتر، _ یک کاراکتر |



---



**بخش قبلی:** [ساختار دیتابیس](01-database-structure.md)

**بخش بعدی:** [کوئری‌های پیچیده](03-complex-queries.md) - یاد می‌گیری JOIN، GROUP BY و خیلی چیزای باحال دیگه!



</div>