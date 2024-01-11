# Retrieving objects

برای بازخوانی اطلاعات از دیتابیس باید یک **کوئری ست** از طریق **منیجر** تعریف شده برای مدل مان ایجاد شود. 


یک کوئری ست ارائه دهنده مجموعه ای از اشیا از دیتابیس است که می تواند یک یا هیچ یا بسیاری فیلتر داشته باشد.
فیلترها بر اساس پارامترهای ورودی می توانند نتایج را فیلتر کنند. در زبان **اس کیو ال** یک **کوئری ست** معادل یک عبارت **سلکت SELECT** است و فیلتر را می توان معادل **WHERE or LIMIT** به حساب آورد

برای دست رسی به یک کوئری ست از پنیجر مدل میتوان استفاده کرد. هر مدلی حداقل یک منیجر دارد که به حالت دیفالت **objects** نامیده می شود.
و به شکل زیر از طریق کلاس مدل در دست رس است.

```python

>>> Blog.objects
<django.db.models.manager.Manager object at ...>

>>> b = Blog(name="Foo", tagline="Bar")
>>> b.objects

Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."

```
 > باید توجه داشت که منیجر ها فقط از طریق کلاس مدل ها قابل دسترسی هستند  نه اینستنسی از مدل. این به جهت جدا کردن عملیات **table-level** از عملیات **record-level** است

منیجر اصلی ترین منبع کوئری ست ها هستند. برای مثال 
Blog.objects.all() 
یک کوئری ست را برمی گرداند که شامل همه اشیای تیبل بلاگ است.
___
## Retrieving all objects
 راحترین راه بازخوانی اطلاعات یک جدول از دیتابیس گرفتن همه ردیف های آن است. برای انجام آنهمانطور که گفتیم از متد all() استفاده میشود.

 ```python
 >>> all_entries = Entry.objects.all()
 ```
  متد ال یک کوئری ست که همه اشیای جدول را برمیگرداند.
___
## Retrieving specific objects with filters
از طریق تعریف فیلترها می توان به یک آبجگت خاص در دیتابیس دسترسی پیدا کرد.

دو روش معمول در پالایش  یک کوئری ست  عبارتند از:

**filter(\*\*kwargs)**
که یک کوئری ست جدید شامل اشیایی که با **لوک آپ** مورد نظر همخوان است برمی گرداند.

**exclude(\*\*kwargs)** 
یک کوئری ست جدید که شامل اشیایی که با **لوک آپ** مورد نظر همخوان نیست را بر میگرداند.

مثلا برای بازخوانی بلاگ های سال ۲۰۱۶ بدین شکل می شود نوشت:
```python

Entry.objects.filter(pub_date__year=2006)
or
Entry.objects.all().filter(pub_date__year=2006)
```
>[lookup-reference لیست کامل لوکاپ ها](./fieldsLookup.md)  

___
# Chaining filters
نتیجه پالایش شده یک کوئری ست خود میتواند کوئری ست باشد که امکان پالایش زنجیره ای را امکان پذیر میکند.
```python
>>> Entry.objects.filter(headline__startswith="What").exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(pub_date__gte=datetime.date(2005, 1, 30))
```
___
# Filtered QuerySets are unique
 هر بار که یک کوئری ست پالایش میشود یک کوئری جدید ساخته می شود و هیچ ارتباطی با قبلی ندارد. هر کدام جداگانه ذخیره و دوباره استفاده می شوند.

 ```python
 >>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
 ```
 مقدار q1 توسط هیچ کدام از دو کوئری بعدی تاثیر نمی گیرد.اما دو کوئری بعدی با کوئری اول ادغام شده اند.
___
# QuerySets are lazy
ایجاد کوئری ست ارتباطی با دیتابیس ندارد. تا زمانی که ارزش یابی نشده است اجرا نمی شود.
```python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```
در مثال فوق به ظاهر سه بار به دیتابیس هیت زده ایم در حالی که فقط یکبار این اتفاق می افتد.
___
## Retrieving a single object with get()
filter() همیشه یه کوئری ست را می دهد حتی وقتی که یک مقدار مچ شود. وقتی که می دانید که فقط یک مقدار با کوئری ما مچ است میشود از get() که متدی از داخل منیجر است استفاده کرد. این متد مقدار را مستقیما بر می گرداند.
```python
>>> one_entry = Entry.objects.get(pk=1)
```
> همانند فیلتر نویسی همه موارد لوکاپ اینجا هم کاربرد دارد.
> متد گت اگر یک مقدار را پیدا نکند اکسپشن **DoesNotExist** و اگر بیش از یک مقدار را پیدا کند ارور **MultipleObjectsReturned** را رییز خواهد کرد.
___
## 
