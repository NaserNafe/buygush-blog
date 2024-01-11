# Model instance reference رفرنس اینستنس مدل 
در این قسمت جزئیات ای پی آی مدل توصیف می شود. که بر اساس موارد گفته شده در [**model**](../model/models.md) و [**database query**](./makingQueries.md) .پایه ریزی شده است
___
## Creating objects ایجاد یک شی یا آبجکت 
برای ایجاد یک نمونه یا اینستنس یک مدل شبیه یک کلاس معمولی پایتونی انجام میگیرد.
**class Model(\*\*kwargs)**
کیورد آرگومنت های این کلاس همان اسامی فیلد هایست که در مدل تعریف می شود. 

> توجه به این نکته خالی از لطف نیست اگر تمایل کنجکاوانه برای اورراید کردن متد. **__init__** دارید. مراقب باشید که سیگنچر صدا زدن راتغیر ندهید که باعث خرابی در ذخیره کردن می شود. و همچنین احتمال وقوع خطای رکرسیو بالاست. اما بهترین راه برای ایجاد تغیرات دلخواه دو مورد پیشنهادی زیر است.
> 1. یک کلاس متد بنویسید
> ```python
> from django.db import models
>
>
>class Book(models.Model):
>    title = models.CharField(max_length=100)
>
>    @classmethod
>    def create(cls, title):
>        book = cls(title=title)
>        # do something with the book
>        return book
>
>
>book = Book.create("Pride and Prejudice")
>```
> 2. افزودن یک منیجر دلبخواه که این روش توصیه می گردد
> 
> ```python 
>class BookManager(models.Manager):
>    def create_book(self, title):
>        book = self.create(title=title)
>        # do something with the book
>        return book
>
>
>class Book(models.Model):
>    title = models.CharField(max_length=100)
>
>    objects = BookManager()
>
>
>book = Book.objects.create_book("Pride and Prejudice")
>```
___
## Customizing model loading   سفارشی کردن لود شدن مدل

**classmethod Model.from_db(db, field_names, values)**
کلاس متد  **from_db()** برای سفارشی کردن نحوه ساخته شدن نمونه به هنگام لود شدن از دیتابیس استفاده می شود.

آرگیومنت **db** حاوی اطلاعاتی از اسم دیتابیس است برای لود شدن.
**field_names** شامل همه اسامی فیلدهایست که لود شده اند.
**values** هم حاوی مقادیر لود شده ا هر از هر فیلد داخل **field_names** است.
فیلد های لود شده به همان ترتیب مقادبر لود شده است.و مقادیر لود شده به همان ترتیب که **__init__()** انتظارشان را دارد است.
نمونه به کمک **cls(\*value)** ساخته می شود.
اگر که هر فیلدی دیفر یا به قولی تاخیر کند در **field_names** ظاهر نخواهد شد.
در این صورت، یک مقدار از  **django.db.models.DEFERRED** به هر یک از فیلد های غایب اساین می شود.

در زیر مثالی که نشان می دهد چگونه مقادیر اولیه از فیلد که از دیتابیس لود می شوند ثبت می شود.

```python
from django.db.models import DEFERRED


@classmethod
def from_db(cls, db, field_names, values):
    # Default implementation of from_db() (subject to change and could
    # be replaced with super()).
    if len(values) != len(cls._meta.concrete_fields):
        values = list(values)
        values.reverse()
        values = [
            values.pop() if f.attname in field_names else DEFERRED
            for f in cls._meta.concrete_fields
        ]
    instance = cls(*values)
    instance._state.adding = False
    instance._state.db = db
    # customization to store the original field values on the instance
    instance._loaded_values = dict(
        zip(field_names, (value for value in values if value is not DEFERRED))
    )
    return instance


def save(self, *args, **kwargs):
    # Check how the current values differ from ._loaded_values. For example,
    # prevent changing the creator_id of the model. (This example doesn't
    # support cases where 'creator_id' is deferred).
    if not self._state.adding and (
        self.creator_id != self._loaded_values["creator_id"]
    ):
        raise ValueError("Updating the value of creator isn't allowed")
    super().save(*args, **kwargs)
```
___
## Refreshing objects from database  نو کردن نمونه از دیتابیس
اگر یک فیلد از اینستنس مدل حذف شود، دسترسی دوباره به آن، مقدار را دوباره از دیتابیس لود میکند.

```python
>>> obj = MyModel.objects.first()
>>> del obj.field
>>> obj.field  # Loads the field from the database
```
**Model.refresh_from_db(using=None, fields=None)**

**Model.arefresh_from_db(using=None, fields=None) ورژن آسینک**

اگر نیاز به نو کردن مقادیر مدل از دیتابیس باشد از متد  **refresh_from_db()** استفاده می شود.
وقتی این متد بدون آرگیومنت صدا زده می شود اتفاقات زیر رخ می دهد.
1. همه فیلد های دیفر نشده مدل آپدیت خواهد شد به مقداری که در این لحظه در دیتا بیس قرار دارد.
2. همه ارتباطات از نمونه نو شده پاک می شوند.
فقط فیلد های همین مدل نه ریلیشن ها نو خواهند شد. دیگر مقادیر وابسته به به دیتابیس مثل annotation ها نو نوا نخواهد شد. هیچ اتریبیوت  @cached_property پاک نخواهد شد.
  














___
## Other attributes دیگر اتریبیت ها

**_state**

**Model._state** 
اتریبیوت استیت برمیگردد به یک آبجکت مدل استیت **ModelState** که لایف سایکل نمونه را رهگیری می کند.
 
 **ModelState** دو اتربیوت دارد :

**adding** که یک فلق است که مقدار true را در صورتی که مدل هنوز در دیتا بیس ذخیره نشده است بر میگرداند 
و **db** یک استرینگ که نام دیتابیسس که نمونه از آن لود و به آن سیو میشود است.

یک اینستنس تازه ساخته شده مقدار اددینگ ترو دارد و دی بی آن نان است چون هنوز ذخیره نشده است.
اینستنسی که از یک کوئری ست آورده شده است اددینگ فالس و دی بی متناظر با دیتابیسی که با آن سروکار دارد است.

