# Field lookups

جست و جوی فیلد با **لوکاپ** تعیین کننده اساس کی ورد **WHERE** در اس کیو ال هستش که به عنوان کی ورد آرگیومنت به متد های **فیلتر** ، **اکسکلود** و متد **گت** مشخص می شود.

بنیان **لوکاپ**ها فرم زیر را به خود میگیرد که ما بین فییلد و لوکاپ تایپ، دابل آندراسکور قرار می گیرد.

## field\_\_lookuptype=value

برای مثال:

```python
>>> Entry.objects.filter(pub_date__lte="2006-01-01")

```

که به زبان اسکیو الی می شود:

```SQL
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

> فیلد نوشته شده در لوکاپ حتما باید نام یکی از فیلد های مدل باشد. فقط استثنا در فیلد های **فارین کی** است که می توان نام فیلد را با **\_id** که نمایانگر **پرایمری کی برای فارین مدل** است. در صورت خطا در کی ورد آرگیومنت فانکشن لوکاپ ارور **TypeError** را رییز خواهد کرد.

```python

>>> Entry.objects.filter(blog_id=4)
```

> برای سادگی وقتی هیچ لوکاپی ارائه نمی شود، خود به خود لوکاپ **exact** در نظر گرفته میشود.

```python
>>> Blog.objects.get(id__exact=14)  # Explicit form
>>> Blog.objects.get(id=14)  # __exact is implied
```

---

## exact

مقدار دقیقا مشابه

```python
>>> Entry.objects.get(headline__exact="Cat bites dog")

   Entry.objects.get(id__exact=14)
   Entry.objects.get(id__exact=None)
```

```SQL
SELECT ... WHERE headline = 'Cat bites dog';
SELECT ... WHERE id = 14;
SELECT ... WHERE id IS NULL;
```

---

## iexact

مقدار مشابه و غیرحساس به بزرگی و کوچکی حروف

```python
>>> Blog.objects.get(name__iexact="beatles blog")
    Blog.objects.get(name__iexact=None)
```

با مقادیر زیر مچ خواهد شد.
**"Beatles Blog", "beatles blog", "BeAtlES blOG"**

```SQL
SELECT ... WHERE name ILIKE 'beatles blog';
SELECT ... WHERE name IS NULL;
```

---

## contains

وقتی که فیلد شامل مقدار تعیین شده باشد.

```python
Entry.objects.get(headline__contains="Lennon")
```

معادل اسکیو الی

```SQL
SELECT ... WHERE headline LIKE '%Lennon%';
```

با مقدار **Today Lennon honored** مچ می شود ولی با مقدار **today lennon honored** مچ نمی شود.

## همچنین برای کانتین هم ورژن کیس سنسیتیو وجود وارد

# icontains

```python
Entry.objects.get(headline__icontains="Lennon")
```

```SQL
SELECT ... WHERE headline ILIKE '%Lennon%';
```

---

## in

از میان هر چیز قابل پیمایش مثل لیست، تاپل یا کوئری ست و یا یک رشته استرینگ.

```python
Entry.objects.filter(id__in=[1, 3, 4])
Entry.objects.filter(headline__in="abc")
```

```SQL
SELECT ... WHERE id IN (1, 3, 4);
SELECT ... WHERE headline IN ('a', 'b', 'c');
```

مثال برای کوئری ست عبارت است از:

```python
inner_qs = Blog.objects.filter(name__contains="Cheddar")
entries = Entry.objects.filter(blog__in=inner_qs)
```

این کوئری ست به عنوان ساب سلکت ایستیتمنت تعبیر خواهد شد.

```SQL
SELECT ... WHERE blog.id IN (SELECT id FROM ... WHERE NAME LIKE '%Cheddar%')
```

باید در **value()** یا **value_list()** دقت داشت که اگر قرار است نتیجه در کوئری ستی که به لوکاپ داده می شود استفاده شود، کوئری ست اول حتما فقط یک فیلد را استخراج کنند.

```python
inner_qs = Blog.objects.filter(name__contains="Ch").values("name")
entries = Entry.objects.filter(blog__name__in=inner_qs)
```

مثال برای موردی که رعایت نشده و خطا خواهد داشت به شکل زیر است.

```python
# Bad code! Will raise a TypeError.
inner_qs = Blog.objects.filter(name__contains="Ch").values("name", "id")
entries = Entry.objects.filter(blog__name__in=inner_qs)
```

> در استفاده از کوئری ها به شکل آبشاری باید به پرفرمنس دیتابیس دقت داشت چرا که بعضی از دیتابیس ها مثل **مای اسکیو ال** کوئری آبشاری را بهینه نمی کنند. بهترین حالت برای آنها استخراج کوئری اول در لیستی و سپس انتقال لوکاپ کوئری دوم است.

```python
values = Blog.objects.filter(name__contains="Cheddar").values_list("pk", flat=True)
entries = Entry.objects.filter(blog__in=list(values))
```

> در مثال فوق **list()** کوئری ست اول را ملزم به استخراج و مقدار دهی می کند سپس به لوکاپ پاس داده می شود. چرا که کوئری ست ها لیزی هستند به عبارتی تا زمانی که نیاز نشده اند در عمل به دیتابیس هیج هدی نمی زنند.

---

## gt

لوکاپ بزرگتر است از:

```python
Entry.objects.filter(id__gt=4)
```

```SQL
SELECT ... WHERE id > 4;
```

---

## gte

## لوکاپ بزرگتر مساوی

## lt

## لوکاپ کوچکتر است از

## lte

## لوکاپ کوچکتر مساوی

## startswith

برای جست و جوی کلمات با شروع کننده مد نظر به شکل کیس سنسیتیو

```python
Entry.objects.filter(headline__startswith="Lennon")
```

```SQL
SELECT ... WHERE headline LIKE 'Lennon%';
```

> دقت داشته باشید که دیتابیس **SQLite** شکل کیس سنسیتیو را ساپرت نمی کند پس **startswith** و **istartswith** حساس به بزرگی و کوچکی حروف نیستند.

---

## istartswith

حالت غیر حساس به بزرگی و کوچکی حروف

```python
Entry.objects.filter(headline__istartswith="Lennon")
```

```SQL
SELECT ... WHERE headline ILIKE 'Lennon%';
```

---

## endswith

جست و جو کننده کلمه ای که ختم شود به

```python
Entry.objects.filter(headline__endswith="Lennon")
```

```SQL
SELECT ... WHERE headline LIKE '%Lennon';
```

---

## iendswith

حالت غیر حساس به بزرگی و کوچکی حروف

```python
Entry.objects.filter(headline__iendswith="Lennon")
```

```SQL
SELECT ... WHERE headline ILIKE '%Lennon'
```

---

## range

تعیین یک محدوده

```python
import datetime

start_date = datetime.date(2005, 1, 1)
end_date = datetime.date(2005, 3, 31)
Entry.objects.filter(pub_date__range=(start_date, end_date))
```

```SQL
SELECT ... WHERE pub_date BETWEEN '2005-01-01' and '2005-03-31';
```

> از رنج در هرجایی استفاده می شود که معادل بیتوین در اس کیو ال است. تاریخ، اعداد و حتی از کاراکتر ها
> توجه: فیلتر کردن فیلد **DateTimeField** با تاریخ شمال روز آخری نخواهد شد، چرا که دربرگیرنده ساعت صفر بامداد است. اگر در مثال بالا pup_date یک فیلد DateTimeField بود معادل اس کیو الی آن به شکل زیر خواهد بود.

```SQL
SELECT ... WHERE pub_date BETWEEN '2005-01-01 00:00:00' and '2005-03-31 00:00:00';
```

---

## date

برای فیلد های دیت تایم مقدار را به دیت یعنی فقط تاریخ تبدیل میکند واجازه می دهد لوکاپ های زنجیری نوشت.

```python
Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))
```

معادل سازی کوئری بالا در هر دیتابیس متفاوت خواهد بود.

> وقتی در ستینگ **USE_TZ** به مقدار **True** اساین شود فیلدها قبل از فیلتر شدن تبدیل به تایم زون جاری می شوند. برای این نیاز هست کع تایم زون در دیتابیس تعریف شود.

---

## year

برای فیلد های دیت تایم ،مقدار دقیق سال مورد نظر را بر میگرداند. یک مقدار اینتجر می گیرد و قابلیت زنجیری دارد.

```python
Entry.objects.filter(pub_date__year=2005)
Entry.objects.filter(pub_date__year__gte=2005)
```

```SQL
SELECT ... WHERE pub_date BETWEEN '2005-01-01' AND '2005-12-31';
SELECT ... WHERE pub_date >= '2005-01-01';
```

ـــ

## month

برای فیلدهای دیت و دیت تایم. مقدار دقیق ماه را بر می گرداند و از حالت زنجیره ای پشتیبانی می کند. ورودی یک اینتجر از ۱ تا ۱۲ را می گیرد.

```python
Entry.objects.filter(pub_date__month=12)
Entry.objects.filter(pub_date__month__gte=6)
```

```SQL
SELECT ... WHERE EXTRACT('month' FROM pub_date) = '12';
SELECT ... WHERE EXTRACT('month' FROM pub_date) >= '6';
```

---

## day

برای فیلدهای دیت و دیت تایم. مقدار دقیق روز را بر می گرداند و از حالت زنجیره ای پشتیبانی می کند. ورودی یک اینتجر از ۱ تا ۳۰ را می گیرد.

```python
Entry.objects.filter(pub_date__week=52)
Entry.objects.filter(pub_date__week__gte=32, pub_date__week__lte=38)
```

---

## week_day

برای فیلدهای دیت و دیت تایم. مقدار دقیق روز هفته را بر می گرداند و از حالت زنجیره ای پشتیبانی می کند. ورودی یک اینتجر از ۱ تا 7 را می گیرد. که یک برای یک شنبه و هفت برای شنبه.

```python
Entry.objects.filter(pub_date__week_day=2)
Entry.objects.filter(pub_date__week_day__gte=2)
```

---

## quarter

برای فیلدهای دیت و دیت تایم. مقدار دقیق فصل را بر می گرداند و از حالت زنجیره ای پشتیبانی می کند. ورودی یک اینتجر از ۱ تا ۴ را می گیرد.

```python
Entry.objects.filter(pub_date__quarter=2)
```

---

## time

برای فیلد های دیت تایم است و مقدار را به زمان تبدیل می کند.از لوکاپ زنجیره ای پشتیبانی می کند. و یک **datetime.time** می گیرد.

```python
Entry.objects.filter(pub_date__time=datetime.time(14, 30))
Entry.objects.filter(pub_date__time__range=(datetime.time(8), datetime.time(17)))
```

---

## hour

برای فیلدهای دیت و دیت تایم. مقدار دقیق ساعت را بر می گرداند و از حالت زنجیره ای پشتیبانی می کند. ورودی یک اینتجر از ۱ تا ۲۳ را می گیرد.

```python
Event.objects.filter(timestamp__hour=23)
Event.objects.filter(time__hour=5)
Event.objects.filter(timestamp__hour__gte=12)
```

```SQL
SELECT ... WHERE EXTRACT('hour' FROM timestamp) = '23';
SELECT ... WHERE EXTRACT('hour' FROM time) = '5';
SELECT ... WHERE EXTRACT('hour' FROM timestamp) >= '12';
```

---

## minute

برای فیلدهای دیت و دیت تایم. مقدار دقیق دقیقه را بر می گرداند و از حالت زنجیره ای پشتیبانی می کند. ورودی یک اینتجر از ۱ تا 59 را می گیرد.

```python
Event.objects.filter(timestamp__minute=29)
Event.objects.filter(time__minute=46)
Event.objects.filter(timestamp__minute__gte=29)
```

```SQL
SELECT ... WHERE EXTRACT('minute' FROM timestamp) = '29';
SELECT ... WHERE EXTRACT('minute' FROM time) = '46';
SELECT ... WHERE EXTRACT('minute' FROM timestamp) >= '29';
```

---
Markdown converts text to HTML.

*[HTML]: HyperText Markup Language
## second

برای فیلدهای دیت و دیت تایم. مقدار دقیق ثانیه را بر می گرداند و از حالت زنجیره ای پشتیبانی می کند. ورودی یک اینتجر از ۱ تا 59 را می گیرد.

```python
Event.objects.filter(timestamp__second=31)
Event.objects.filter(time__second=2)
Event.objects.filter(timestamp__second__gte=31)
```

```SQL
SELECT ... WHERE EXTRACT('second' FROM timestamp) = '31';
SELECT ... WHERE EXTRACT('second' FROM time) = '2';
SELECT ... WHERE EXTRACT('second' FROM timestamp) >= '31';
```

---

## isnull

مقادیر **Ttue** یا **False** را می گیرد که معادل کوئری اس کیو الی **IS NULL** و **IS NOT NULL** به ***

 - [ ] 
 - [ ] 

ترتیب

*** است.

```python
Entry.objects.filter(pub_date__isnull=True)
```

```SQL
SELECT ... WHERE pub_date IS NULL;
```

---

## regex

عبارت حساس به حروف رگیولار

```python
Entry.objects.get(title__regex=r"^(An?|The) +")
```

```SQL
SELECT ... WHERE title REGEXP BINARY '^(An?|The) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(An?|The) +', 'c'); -- Oracle

SELECT ... WHERE title ~ '^(An?|The) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '^(An?|The) +'; -- SQLite
```
___
## iregex

حالت غیر حساس رگیولار

```python
Entry.objects.get(title__iregex=r"^(an?|the) +")
```
```SQL
SELECT ... WHERE title REGEXP '^(an?|the) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(an?|the) +', 'i'); -- Oracle

SELECT ... WHERE title ~* '^(an?|the) +'; -- PostgreSQL

**gcngct** **

# SELECT

** ... WHERE title REGEXP '(?i)^(an?|the
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMTMzOTk4NTIsMjQ4NTY1Mjg0XX0=
-->