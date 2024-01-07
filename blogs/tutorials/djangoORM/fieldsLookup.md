# Field lookups

جست و جوی فیلد با **لوک آپ** تعیین کننده اساس کی ورد **WHERE** در اس کیو ال هستش که به عنوان کی ورد آرگیومنت به متد های **فیلتر** ، **اکسکلود** و متد **گت** مشخص می شود.

بنیان **لوک آپ**ها فرم زیر را به خود میگیرد که ما بین فییلد و لوک آپ تایپ، دابل آندراسکور قرار می گیرد.

## field__lookuptype=value

برای مثال:

```python
>>> Entry.objects.filter(pub_date__lte="2006-01-01")

```

که به زبان اسکیو الی می شود:

```SQL
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```
>فیلد نوشته شده در لوک آپ حتما باید نام یکی از فیلد های مدل باشد. فقط استثنا در فیلد های **فارین کی** است که می توان نام فیلد را با **_id**  که نمایانگر **پرایمری کی برای فارین مدل** است. در صورت خطا در کی ورد آرگیومنت فانکشن لوک آپ ارور **TypeError** را رییز خواهد کرد.


```python

>>> Entry.objects.filter(blog_id=4)
```

>برای سادگی وقتی هیچ لوک آپی ارائه نمی شود، خود به خود لوک آپ **exact** در نظر گرفته میشود.

```python 
>>> Blog.objects.get(id__exact=14)  # Explicit form
>>> Blog.objects.get(id=14)  # __exact is implied
```

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

همچنین برای کانتین هم ورژن کیس سنسیتیو وجود وارد

# icontains

```python
Entry.objects.get(headline__icontains="Lennon")
```
```SQL
SELECT ... WHERE headline ILIKE '%Lennon%';
```

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
 این کوئری ست به عنوان ساب سلکت ستیتمنت تعبیر خواهد شد.

```SQL
SELECT ... WHERE blog.id IN (SELECT id FROM ... WHERE NAME LIKE '%Cheddar%')
```
