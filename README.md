<div dir="rtl">



# یادگیری SQL Server از صفر تا صد



یه راهنمای ساده برای یاد گرفتن دیتابیس SQL Server. همه چیز رو با مثال‌های یه فروشگاه آنلاین توضیح دادم که ملموس باشه.



---



## چی یاد می‌گیری؟



| بخش | موضوع |

|-----|-------|

| **1. ساختار دیتابیس** | ساخت دیتابیس، جدول، انواع داده‌ها |

| **2. عملیات CRUD** | اضافه، خوندن، ویرایش، حذف داده |

| **3. کوئری‌های پیچیده** | JOIN، GROUP BY، Subquery |

| **4. مفاهیم پیشرفته** | Transaction، Trigger، Procedure |

| **5. چیت‌شیت** | همه چیز یه جا! |



---



## از کجا شروع کنم؟



1. [ساختار دیتابیس](01-database-structure.md) - اول اینو بخون

2. [عملیات CRUD](02-crud-operations.md) - بعد این

3. [کوئری‌های پیچیده](03-complex-queries.md) - سپس این

4. [مفاهیم پیشرفته](04-professional-concepts.md) - و در آخر این

5. [چیت‌شیت](05-cheatsheet.md) - قبل امتحان!



---



## جداول فروشگاه



توی این آموزش، یه فروشگاه آنلاین می‌سازیم با این جداول:



```

Categories (دسته‌بندی)     Products (محصولات)

├── CategoryID             ├── ProductID

├── CategoryName           ├── CategoryID

└── ...                    ├── ProductName

                           ├── Price

                           └── Stock

 

Customers (مشتری‌ها)        Orders (سفارشات)

├── CustomerID             ├── OrderID

├── FirstName              ├── CustomerID

├── LastName               ├── OrderDate

├── Email                  ├── Status

└── ...                    └── TotalAmount

 

OrderItems (آیتم‌های سفارش)

├── OrderItemID

├── OrderID

├── ProductID

├── Quantity

└── UnitPrice

```



---



## چی لازم داری؟



- SQL Server (نسخه 2019 به بالا)

- SQL Server Management Studio (SSMS)



---



## نکته‌های مهم



- همه کدها رو خودت امتحان کن!

- از WHERE توی UPDATE و DELETE فراموش نکن!

- برای متن فارسی از NVARCHAR استفاده کن!



---



موفق باشی!



</div>