# Making queries
هنگامی که مدل های داده خود را ایجاد کردید، جنگو به طور خودکار یک <span dir="rtl">API</span> انتزاعی از پایگاه داده به شما می دهد که به شما امکان ایجاد، بازیابی، به روز رسانی و حذف اشیاء را می دهد. 
در طول این آموزش به این مدل که بخشی از یک بلاگ هستش ارجاع خواهد شد.


```python

from datetime import date

from django.db import models


class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name


class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name


class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField(default=date.today)
    authors = models.ManyToManyField(Author)
    number_of_comments = models.IntegerField(default=0)
    number_of_pingbacks = models.IntegerField(default=0)
    rating = models.IntegerField(default=5)

    def __str__(self):
        return self.headline

```
## Creating objects

> جنگو کلاس یا مدل را به عنوان یک تیبل در دیتابیس در نظر می گیرد،و اتریبیوت ها را ستون های جدول تعریف مییکند. وهر اینستنس از این کلاس معادل یک روو یا ردیف جدول است.
___

برای ذخیره کردن اطلاعات در دیتابیس، ابتدا کلاس را اینستنشیت می کنیم و سپس متود <span dir="rtl">save()</span>  را صدا میزنیم.


```python

>>> from blog.models import Blog
>>> b = Blog(name="Beatles Blog", tagline="All the latest Beatles news.")
>>> b.save()

```
> متود سیو هیچ چیزی برنمی گرداند، و به خاطر داشته باشید که جنگو تا زمانی که  متود سیو کال نشده است ارتباطی با دیتابیس برقرار نمی کند. برای متد کال معادل <span dir="rtl">INSERT</span>  در زبان اس کیو ال عمل می کند. 

# Saving changes to objects

 از متد سیو برای آپدیت مقادیر نیز استفاده می شود برای مثال:

```python 
>>> b.name = "New name"
>>> b.save()

```
---
## Saving ForeignKey and ManyToManyField fields 

برای آپدیت کردن فیلدهای **فاریین کی** مثل یک فیلد معمولی عمل می شوند.

```python

>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()

```
> باید توجه داشت که  برای متد گت حتما فقط یک مقدار صحیح موجود باشد در غیر این صورت خطا خواهد داشت. 
---
برای آپدیت کردن فیلد های **منی تو منی فیلد** بجای سیو از متود **ادد** استفاده می شود.

```python

>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

همچنین برای افزودن چندین مقدار به یک باره به **من تو منی فیلد** به شکل زیر عمل می شود. 
```python

>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```
> اگر یک مقدار اشتباهی  به این فیلد وارد شود، جگو ارور خواهد داد.


