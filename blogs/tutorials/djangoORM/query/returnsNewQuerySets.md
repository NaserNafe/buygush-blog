# Methods that return new QuerySets
در زیر لیست متد هست که خود یک کوئری ست بر می گردانند.
> این متد ها کوئری دیتابیس را اجرا نمی کنند به همین دلیل برای اجرای ایسینک امن هستند و نیازی به متد جایگزین برای ایسینک ندارند.
___
## filter(*args, **kwargs)

کوئری ستی را متناسب با [لوکاپهای](./fieldsLookup.md) داده شده برمی گرداند.
> پارامتر های چنتایی با هم به شکل **AND** در زیر ساختار اسکیو الی جوین می شوند.
> اگر به ساختار پیچیده تر مثل **OR** نیاز باشد می توان از **Q object(\*args)** استفاده کرد.
___
## exclude(*args, **kwargs)

یک کوئری ست شامل آبجکت های که با لوکاپهای داده شده مچ نیست را بر می گرداند.
> چندین پارامتر یکجا داده شده به شکا **AND** در زیر لایه اس کیو الی جوین می شوند و کل عبارت در داخا **NOT()** جای می گیرد.

```python
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline="Hello")
```
```SQL
SELECT ...
WHERE NOT (pub_date > '2005-1-3' AND headline = 'Hello')
```
```python
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline="Hello")
```
```SQL
SELECT ...
WHERE NOT pub_date > '2005-1-3'
AND NOT headline = 'Hello'
```
در مثال دوم مواری بجز بعد از تاریخ ذکر شده و همچنین نباید عنوانش هلو باشد را بر می گرداند.
___
##  annotate(*args, **kwargs)

