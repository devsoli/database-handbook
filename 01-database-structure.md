<div dir="rtl">



# ساختار دیتابیس - از صفر تا صد



---



## این بخش چیه؟



اینجا یاد می‌گیری چطوری یه دیتابیس بسازی و جدول‌هاشو درست کنی. فکر کن می‌خوای یه فروشگاه آنلاین راه بندازی - اول باید یه انبار (دیتابیس) داشته باشی و توش قفسه‌ها (جدول‌ها) رو بچینی.



---



## فهرست



1. [ساخت دیتابیس](#1-ساخت-دیتابیس)

2. [انتخاب دیتابیس](#2-انتخاب-دیتابیس)

3. [ساخت جدول](#3-ساخت-جدول)

4. [انواع داده‌ها](#4-انواع-داده‌ها)

5. [محدودیت‌ها (قوانین جدول)](#5-محدودیت‌ها)

6. [تغییر جدول](#6-تغییر-جدول)

7. [حذف جدول و دیتابیس](#7-حذف-جدول-و-دیتابیس)



---



## 1. ساخت دیتابیس



اولین کار اینه که یه دیتابیس بسازی. خیلی ساده‌ست:



```sql

CREATE DATABASE OnlineShop;

```



همین! الان یه دیتابیس به اسم OnlineShop داری.



اگه خواستی تنظیمات بیشتری بدی (مثلاً سایز اولیه):



```sql

CREATE DATABASE OnlineShop

ON PRIMARY

(

    NAME = 'OnlineShop_Data',

    FILENAME = 'C:\SQLData\OnlineShop.mdf',

    SIZE = 100MB,

    MAXSIZE = 10GB,

    FILEGROWTH = 50MB

);

```



ولی خب معمولاً همون خط اول کافیه!



---



## 2. انتخاب دیتابیس



وقتی چند تا دیتابیس داری، باید بگی روی کدوم می‌خوای کار کنی:



```sql

USE OnlineShop;

```



از این به بعد هر چی بنویسی، روی این دیتابیس اعمال میشه.



می‌خوای بدونی الان روی کدوم دیتابیسی؟



```sql

SELECT DB_NAME() AS CurrentDatabase;

```



---



## 3. ساخت جدول



جدول جاییه که داده‌هات ذخیره میشن. هر جدول یه سری ستون (فیلد) داره.



### ساختار کلی:

```sql

CREATE TABLE اسم_جدول

(

    ستون۱    نوع_داده    [محدودیت‌ها],

    ستون۲    نوع_داده    [محدودیت‌ها],

    ...

);

```



### مثال: جدول دسته‌بندی محصولات

```sql

CREATE TABLE Categories

(

    CategoryID      INT             PRIMARY KEY IDENTITY(1,1),

    CategoryName    NVARCHAR(100)   NOT NULL,

    Description     NVARCHAR(500)   NULL,

    IsActive        BIT             DEFAULT 1,

    CreatedAt       DATETIME        DEFAULT GETDATE()

);

```



چی نوشتیم؟

- `CategoryID`: شماره یکتا برای هر دسته (خودش شماره میده)

- `CategoryName`: اسم دسته که حتماً باید پر بشه

- `Description`: توضیحات که می‌تونه خالی باشه

- `IsActive`: فعال/غیرفعال بودن (پیش‌فرض فعاله)

- `CreatedAt`: تاریخ ساخت (خودش تاریخ امروزو میذاره)



### مثال: جدول محصولات

```sql

CREATE TABLE Products

(

    ProductID       INT             PRIMARY KEY IDENTITY(1,1),

    ProductName     NVARCHAR(200)   NOT NULL,

    CategoryID      INT             NOT NULL,

    Price           DECIMAL(18,2)   NOT NULL,

    Stock           INT             DEFAULT 0,

    SKU             VARCHAR(50)     UNIQUE,

    Description     NVARCHAR(MAX)   NULL,

    IsActive        BIT             DEFAULT 1,

    CreatedAt       DATETIME        DEFAULT GETDATE()

);

```



### مثال: جدول مشتری‌ها

```sql

CREATE TABLE Customers

(

    CustomerID      INT             PRIMARY KEY IDENTITY(1,1),

    FirstName       NVARCHAR(50)    NOT NULL,

    LastName        NVARCHAR(50)    NOT NULL,

    Email           VARCHAR(100)    UNIQUE NOT NULL,

    Phone           VARCHAR(20)     NULL,

    City            NVARCHAR(50)    NULL,

    IsActive        BIT             DEFAULT 1,

    CreatedAt       DATETIME        DEFAULT GETDATE()

);

```



### مثال: جدول سفارشات

```sql

CREATE TABLE Orders

(

    OrderID         INT             PRIMARY KEY IDENTITY(1,1),

    CustomerID      INT             NOT NULL,

    OrderDate       DATETIME        DEFAULT GETDATE(),

    Status          VARCHAR(20)     DEFAULT 'Pending',

    TotalAmount     DECIMAL(18,2)   NOT NULL,

    ShippingAddress NVARCHAR(500)   NOT NULL

);

```



---



## 4. انواع داده‌ها



### اعداد



| نوع | چی هست | مثال |

|-----|--------|------|

| `INT` | عدد صحیح معمولی | شناسه، تعداد |

| `BIGINT` | عدد خیلی بزرگ | شماره تراکنش |

| `DECIMAL(18,2)` | اعشاری دقیق | قیمت (۱۸ رقم کل، ۲ رقم اعشار) |

| `BIT` | فقط ۰ یا ۱ | فعال/غیرفعال |



```sql

Price       DECIMAL(18,2)   -- مثلاً: 1,250,000.00

Stock       INT             -- مثلاً: 150

IsActive    BIT             -- مثلاً: 1 (فعال)

```



### متن



| نوع | چی هست | کِی استفاده کن |

|-----|--------|---------------|

| `VARCHAR(n)` | متن انگلیسی | ایمیل، کد محصول |

| `NVARCHAR(n)` | متن فارسی/یونیکد | اسم، توضیحات |

| `NVARCHAR(MAX)` | متن خیلی بلند | توضیحات کامل محصول |



```sql

ProductName NVARCHAR(200)   -- "گوشی سامسونگ گلکسی"

Email       VARCHAR(100)    -- "ali@email.com"

Description NVARCHAR(MAX)   -- متن طولانی

```



> **نکته:** برای فارسی همیشه از NVARCHAR استفاده کن!



### تاریخ و زمان



| نوع | چی هست |

|-----|--------|

| `DATE` | فقط تاریخ (۱۴۰۳-۰۱-۱۵) |

| `TIME` | فقط ساعت (۱۴:۳۰:۰۰) |

| `DATETIME` | تاریخ + ساعت |



```sql

OrderDate   DATETIME    -- 2024-01-15 14:30:00

BirthDate   DATE        -- 1990-05-20

```



---



## 5. محدودیت‌ها



محدودیت‌ها قوانینی هستن که داده‌هات رو کنترل می‌کنن.



### PRIMARY KEY (کلید اصلی)

شناسه یکتا برای هر ردیف. نه تکراری میشه، نه خالی.



```sql

CustomerID INT PRIMARY KEY

```



### IDENTITY

خودش شماره بده (۱، ۲، ۳، ...)



```sql

ProductID INT PRIMARY KEY IDENTITY(1,1)

-- از ۱ شروع کن، یکی یکی اضافه کن

```



### FOREIGN KEY (کلید خارجی)

ارتباط بین دو جدول



```sql

CREATE TABLE Products

(

    ProductID   INT PRIMARY KEY IDENTITY(1,1),

    CategoryID  INT NOT NULL,

    ProductName NVARCHAR(200) NOT NULL,

 

    FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)

);

```



یعنی `CategoryID` توی Products باید حتماً توی جدول Categories وجود داشته باشه.



### NOT NULL

این فیلد نمی‌تونه خالی باشه.



```sql

FirstName NVARCHAR(50) NOT NULL

```



### UNIQUE

مقدار نباید تکراری باشه.



```sql

Email VARCHAR(100) UNIQUE

```



### DEFAULT

اگه مقداری وارد نشد، این مقدار رو بذار.



```sql

IsActive BIT DEFAULT 1              -- پیش‌فرض: فعال

CreatedAt DATETIME DEFAULT GETDATE() -- پیش‌فرض: الان

Stock INT DEFAULT 0                 -- پیش‌فرض: صفر

```



### CHECK

یه شرط بذار روی مقدار.



```sql

Price DECIMAL(18,2) CHECK (Price > 0)           -- قیمت باید مثبت باشه

Rating TINYINT CHECK (Rating BETWEEN 1 AND 5)   -- امتیاز ۱ تا ۵

```



### یه مثال کامل با همه محدودیت‌ها



```sql

CREATE TABLE OrderItems

(

    OrderItemID INT PRIMARY KEY IDENTITY(1,1),

    OrderID     INT NOT NULL,

    ProductID   INT NOT NULL,

    Quantity    INT NOT NULL CHECK (Quantity > 0),

    UnitPrice   DECIMAL(18,2) NOT NULL CHECK (UnitPrice >= 0),

    Discount    DECIMAL(5,2) DEFAULT 0 CHECK (Discount BETWEEN 0 AND 100),

 

    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),

    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)

);

```



---



## 6. تغییر جدول



جدولت ساختی ولی می‌خوای تغییرش بدی؟ از `ALTER TABLE` استفاده کن.



### اضافه کردن ستون جدید

```sql

ALTER TABLE Customers ADD MobileNumber VARCHAR(15);

```



### حذف ستون

```sql

ALTER TABLE Customers DROP COLUMN MobileNumber;

```



### تغییر نوع ستون

```sql

ALTER TABLE Products ALTER COLUMN ProductName NVARCHAR(300);

```



### اضافه کردن محدودیت

```sql

-- اضافه کردن UNIQUE

ALTER TABLE Products ADD CONSTRAINT UQ_SKU UNIQUE (SKU);

 

-- اضافه کردن CHECK

ALTER TABLE Products ADD CONSTRAINT CK_Stock CHECK (Stock >= 0);

```



### حذف محدودیت

```sql

ALTER TABLE Products DROP CONSTRAINT CK_Stock;

```



### تغییر نام ستون

```sql

EXEC sp_rename 'Customers.Phone', 'PhoneNumber', 'COLUMN';

```



---



## 7. حذف جدول و دیتابیس



### حذف جدول

```sql

DROP TABLE Products;

 

-- یا اگه مطمئن نیستی وجود داره

DROP TABLE IF EXISTS Products;

```



**نکته مهم:** اگه جدولی به جدول دیگه وصله (با FOREIGN KEY)، اول باید جدول فرزند رو حذف کنی.



```sql

-- ترتیب مهمه!

DROP TABLE IF EXISTS OrderItems;  -- اول فرزند

DROP TABLE IF EXISTS Orders;       -- بعد والد

```



### حذف دیتابیس

```sql

USE master;  -- اول برو بیرون از اون دیتابیس

DROP DATABASE OnlineShop;

```



### خالی کردن جدول (بدون حذف ساختار)

```sql

TRUNCATE TABLE Products;

```



فرق TRUNCATE با DELETE:



| | TRUNCATE | DELETE |

|---|----------|--------|

| سرعت | خیلی سریع | کندتر |

| شرط | نمیشه شرط گذاشت | میشه WHERE گذاشت |

| شماره خودکار | ریست میشه | ریست نمیشه |



---



## تمرین: بیا یه دیتابیس کامل بسازیم!



```sql

-- ساخت دیتابیس

CREATE DATABASE OnlineShop;

GO

USE OnlineShop;

GO

 

-- جدول دسته‌بندی

CREATE TABLE Categories

(

    CategoryID      INT PRIMARY KEY IDENTITY(1,1),

    CategoryName    NVARCHAR(100) NOT NULL,

    Description     NVARCHAR(500) NULL,

    IsActive        BIT DEFAULT 1,

    CreatedAt       DATETIME DEFAULT GETDATE()

);

GO

 

-- جدول محصولات

CREATE TABLE Products

(

    ProductID       INT PRIMARY KEY IDENTITY(1,1),

    CategoryID      INT NOT NULL,

    ProductName     NVARCHAR(200) NOT NULL,

    SKU             VARCHAR(50) UNIQUE NOT NULL,

    Price           DECIMAL(18,2) NOT NULL CHECK (Price > 0),

    Stock           INT DEFAULT 0 CHECK (Stock >= 0),

    Description     NVARCHAR(MAX) NULL,

    IsActive        BIT DEFAULT 1,

    CreatedAt       DATETIME DEFAULT GETDATE(),

 

    FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)

);

GO

 

-- جدول مشتریان

CREATE TABLE Customers

(

    CustomerID      INT PRIMARY KEY IDENTITY(1,1),

    FirstName       NVARCHAR(50) NOT NULL,

    LastName        NVARCHAR(50) NOT NULL,

    Email           VARCHAR(100) UNIQUE NOT NULL,

    Phone           VARCHAR(20) NULL,

    City            NVARCHAR(50) NULL,

    IsActive        BIT DEFAULT 1,

    CreatedAt       DATETIME DEFAULT GETDATE()

);

GO

 

-- جدول سفارشات

CREATE TABLE Orders

(

    OrderID         INT PRIMARY KEY IDENTITY(1,1),

    CustomerID      INT NOT NULL,

    OrderNumber     VARCHAR(20) UNIQUE NOT NULL,

    OrderDate       DATETIME DEFAULT GETDATE(),

    Status          VARCHAR(20) DEFAULT 'Pending',

    TotalAmount     DECIMAL(18,2) NOT NULL,

    ShippingAddress NVARCHAR(500) NOT NULL,

 

    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),

    CHECK (Status IN ('Pending', 'Confirmed', 'Shipped', 'Delivered', 'Cancelled'))

);

GO

 

-- جدول آیتم‌های سفارش

CREATE TABLE OrderItems

(

    OrderItemID     INT PRIMARY KEY IDENTITY(1,1),

    OrderID         INT NOT NULL,

    ProductID       INT NOT NULL,

    Quantity        INT NOT NULL CHECK (Quantity > 0),

    UnitPrice       DECIMAL(18,2) NOT NULL,

 

    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),

    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)

);

GO

 

PRINT 'دیتابیس با موفقیت ساخته شد!';

```



---



## خلاصه



| دستور | کار |

|-------|-----|

| `CREATE DATABASE` | ساخت دیتابیس |

| `USE` | انتخاب دیتابیس |

| `CREATE TABLE` | ساخت جدول |

| `ALTER TABLE ADD` | اضافه کردن ستون |

| `ALTER TABLE DROP` | حذف ستون |

| `ALTER TABLE ALTER` | تغییر ستون |

| `DROP TABLE` | حذف جدول |

| `DROP DATABASE` | حذف دیتابیس |

| `TRUNCATE TABLE` | خالی کردن جدول |



---



**بخش بعدی:** [عملیات CRUD](02-crud-operations.md) - یاد می‌گیری چطور داده اضافه کنی، بخونی، ویرایش کنی و حذف کنی!



</div>