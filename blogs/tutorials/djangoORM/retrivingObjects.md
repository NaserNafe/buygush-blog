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

## Retrieving all objects
 راحترین راه بازخوانی اطلاعات یک جدول از دیتابیس گرفتن همه ردیف های آن است. برای انجام آنهمانطور که گفتیم از متد all() استفاده میشود.

 ```python
 >>> all_entries = Entry.objects.all()
 ```
  متد ال یک کوئری ست که همه اشیای جدول را برمیگرداند.

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