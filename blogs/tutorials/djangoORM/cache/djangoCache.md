# Django’s cache framework فریم فرک کش جنگو

نکته مثیت یک وبسایت دانامیک خب داینامیک بودن سایته. هر موقه که یک کاربر یک پیجی را ریکوست میکند، وب سرور همه جوره محاسباتی انجام میده از کوئری زدن به دیتابیس بگیر تا رندر کردن تمپلیت ها و لاجیک مربوط به بیزینس تا کاربر سایت را مشاهده کند.
از دیدگاه سربار پردازش، این بسیار گران‌تر از چیدمان سرور استاندارد خواندن یک فایل خارج از فایل سیستم (standard read-a-file-off-the-filesystem server arrangement روشی برای خواندن اطلاعات بدون پردازش) شما است.
برای بسیاری از وب اپلیکیشن ها این سربار مسئله بسیار مهمی نیست. همه اپلیکیشن ها که سایت واشینگتون پست یا سایت اسلش دات او آرجی نیستند.آنها اکثرا اپی بین کوچک تا متوسط سایز با ترافیک متوسط هستند. اما برای سایت های پر ترافیک خیلی ضروری است تا سربار را تا حد ممکن کاهش داد.
اینجاست که کش کردن بسیار کمک کننده است.

کش کردن یعنی ذخیره کردن نتایج محاسبات پرهزینه است تا بار دیگر این محاسبات صورت نگیرد.
جنگو دارای سیسم کشینگ قوی است که اجازه ذخیره پیج های داینامیک را میدهد تا دوباره محاسبات از اول صورت نگیرد.

جنگو جهت سهولت سطح جزئیات کش مختلفی ارائه می دهد.
شما می توانید خروجی ویوهای ویژه را کش کنید، شما می توانیدفقط یک تکه که تولید آن سخت است یا کل سایت را کش کنید.

جنگو همچنین با داون استریمرکش ها مثل **Squid** و **browser-based** به خوبی کار می کند.

---

## Setting up the cache تنظیم کردن کش

سیستم کش نیاز به مقدار کم از تنظیمات نیاز دارد. برای نمونه شما باید مشخص کنید کجا باید اطلاعات زندگی کنند، در دیتابیس یا فایل سیستم و یا مستقیم در مموری. این یک تصمیم مهمی است که تاثیر بسزایی در پرفرمنس کش دارد. بعضی از کش ها سریع تر از دیگری عمل می کنند.

ترجیحات مربوط به کش در فایل ستینگ <a href="/blogs/tutorials/djangoORM/settings/settingsFile.md#caches">Cache</a> نوشته می شود.

---

## Memcached

ممکش یک سرور کاملا مموری بیس است که اساسا توسعه یافت تا لود بسیار بالا در سایت **LiveJournal.com** را هندل کند و در نتیجه توسط اینتراکتیو جنگو اوپن سورس شد. توسط سایت های مثل **Facebook** و **Wikipedia** استفاده گردید تا دسترسی به دیتابیس را کاهش و پرفرمنس سایت ها را بالا برد.

ممکش به شکل دیمن یعنی در بکگراند اجرا می شود و مقدار رم زیادی را به خود اختصاص می دهد. همه کاری که انجام می دهد این است که یک اینترفیس بسیار سریع برای افزودن و بازخوانی و حذف دیتا در کش ارائه می دهد. همه دیتا در مستقیما در مموری ذخیره می شود پس سرباری به فایس سیسنم و دیتابیس ندارد.

بعد از نصب خود **Memcached** نیاز به نصب بایند کننده ممکش است. تعدادی بایند ممکش برای پایتون موجود است که جنگو از دو مورد **pylibmc** و **pymemcache** پشتیبانی می کند.

برای استفاده از ممکش با جنگو؛

- بسته به انتخاب بایندینگ باید **django.core.cache.backends.memcached.PyMemcacheCache** یا **django.core.cache.backends.memcached.PyLibMCCache** را در **BACKEND** فایل ستینگ باید ست گردد.
- و قسمت **LOCATION** به شکل **ip:port** که آی پی آی پی آدرس ممکش در بکگراند و پورت هم به پورت ممکش کی در حال اجرا است باید ست شود. ویا به شکل **unix:path** باید نوشته شود که پس مسیر Memcached Unix socket file است.

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.memcached.PyMemcacheCache",
        "LOCATION": "127.0.0.1:11211",
    }
}
```

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.memcached.PyMemcacheCache",
        "LOCATION": "unix:/tmp/memcached.sock",
    }
}
```

یک ویژگی عالی ممگش توانایی آن در شییر کردن بر روی چندین سرور است. یعنی ممکش دیمون را می توان روی چند ماشین اجرا کرد، وبرنامه انگار فقط با یک ماشین سر و کار دارد، بدون اینکه مقادیر کش را در ماشین ها تکرار کرد.
برای بهره بردن از این توانایی باید همه آدرس های سرور ها را در **LOCATION** نوشت چه به شکل یک لیست ویا چه به شکل جدا کننده کما ویا سمی کولن در سترینگ.

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.memcached.PyMemcacheCache",
        "LOCATION": [
            "172.19.26.240:11211",
            "172.19.26.242:11211",
        ],
    }
}
```

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.memcached.PyMemcacheCache",
        "LOCATION": [
            "172.19.26.240:11211",
            "172.19.26.242:11212",
            "172.19.26.244:11213",
        ],
    }
}
```

به طور پیش فرض **PyMemcacheCache** بک اند آپشن های زیر را ست می کند. برای اورراید کردن آنها <a href="/blogs/tutorials/djangoORM/settings/settingsFile.md#">OPTIONS</a>

---

## Cache arguments

به هر کش بک اندی می توان آرگیومنت های اضافی داد برای کنترل رفتار کش داد. این آرگیومنت ها به عنوان کلید های اضافی در <a href="/blogs/tutorials/djangoORM/settings/settingsFile.md#caches">CACHES</a> ارائه می شوند. مقادیر معتبر آرگیومنت ها عبارنتد از:

- **TIMEOUT**:

مقدار پیشفرض تایم اوت به ثانیه و برای استفاده در کش است.
که مقدار \*_۳۰۰_ ثانیه یا ۵ دقیقه است. می توان تایم اوت را به نان ست کرد که کلید کش هرگز منقضی نخواهد شد. مقدار را صفر قرار پاپن باعث منقضی شدن بلافاصله خواهد که عملا کشی صورت نخواهد گرفت.

- **OPTIONS**

هر آپشنی که باید به بک اند کش تحویل داده شود.لیست موارد مجاز بسته به هر بک اندی متفاوت است، و کش بک اند با یک لایبرری پشتیبانی می شود آپشن های آن از اینجا گرفته خواهد شد.
هر کش بک اندی ستراتژی جمع آوری اطلاعات خود را بع کر می بندد.
**locmem, filesystem** و **database** آپشن های زیر را معتبر می دانند.

- **MAX_ENTRIES**
  ماکزیمم تعداد ورودی های مجاز کش قبل از اینکه مقادیر کهنه پاک شوند. مقدار پیشفرض آن ۳۰۰ می باشد.

- **CULL_FREQUENCY**
  کسری از مقادیر که در زمان رسیدن به مقدار ماکزیمم جمع آوری می شود. مقدار نسبت یک بر **CULL_FREQUENCY** خواهد بود. با ست کردن **CULL_FREQUENCY** به ۲ عمل جمع آوری نصف مقادیر وقتی به ماکز میرسد اتفاق خواهد افتاد. مقدار پیشفرض ۳ است که ورودی حتما باید یک اینتیجر باشد.

  مقار را به صفر ست کردن یعنی همه ورودی ها وقتی به ماکز رسیدند خالی خواهد شد. در بعضی از بک اند ها مثل دیتابیس به ویژه صفر قرار دادن باعث افزایش سرعت خواهد شد اما به قیمت از دست دادن اطلاعات.

بک اند های ممکش و ردیس آپشن ها را به شکل کیوردآرگیومنت به کلاینت کانستراکتور تحویل می دهند که کنترل پیشرفته ای روی رفتار کلاینت دارد.

- **KEY_PREFIX**

یک استرینگ که اتوماتیک شامل (پیش فرض به ابتدای آن افزوده خواهد شد) همه کلید های کش استفاده شده توسط سرور جنگو است

- **VERSION**

  مقدار دیفالت شماره ورژن برای کلید های تولید شده کش توسط سرور جنگو.

- **KEY_FUNCTION**

  یک رشته شامل آدرس نقطه دار یک فانکشن یا هر چیز قابل صدا زدن، که تعریف می کند چگونه یک پرفیکس و ورژن . کلید را برای کلید نهایی کش سرهم کند.

در این مثال فایل سیستم پیکربندی می شود.

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.filebased.FileBasedCache",
        "LOCATION": "/var/tmp/django_cache",
        "TIMEOUT": 60,
        "OPTIONS": {"MAX_ENTRIES": 1000},
    }
}
```

یک مثال از پیکر بندی **pylibmc** بک اند که پورتکل احراض هویت SASL و مد رفتاری **ketama** را فعال می کند.

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.memcached.PyLibMCCache",
        "LOCATION": "127.0.0.1:11211",
        "OPTIONS": {
            "binary": True,
            "username": "user",
            "password": "pass",
            "behaviors": {
                "ketama": True,
            },
        },
    }
}
```

در این مثال پیکربندی برای **pymemcache** که نظرسنجی از کاربر که محتملا موجب افزایش پرفرمنس با در ارتباط نگهداشتن کلاینت را فعال میکند.ابرادات ممکش یا نتورک را مثل کش از دست رفته فرض میکند و فلق **TCP_NODLAY** در کانکشن سوکت ست می کند.

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.memcached.PyMemcacheCache",
        "LOCATION": "127.0.0.1:11211",
        "OPTIONS": {
            "no_delay": True,
            "ignore_exc": True,
            "max_pool_size": 4,
            "use_pooling": True,
        },
    }
}
```

یک مثال از پیکر بندی ردیس

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.redis.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379",
        "OPTIONS": {
            "db": "10",
            "parser_class": "redis.connection.PythonParser",
            "pool_class": "redis.BlockingConnectionPool",
        },
    }
}
```

---

<div class="section" id="s-the-per-site-cache">
<span id="s-id9"></span><span id="the-per-site-cache"></span><span id="id9"></span><h2>The per-site cache<a class="headerlink" href="#the-per-site-cache" title="Permalink to this headline">¶</a></h2>
<p dir="rtl">وقتی که کش تنظیم شد، راحترین راه استفاده از کشینگ، کش کردن کل سایت است. فقط نیاز است اضافه کردن

<code class="docutils literal notranslate"><span class="pre">'django.middleware.cache.UpdateCacheMiddleware'</span></code> و
<code class="docutils literal notranslate"><span class="pre">'django.middleware.cache.FetchFromCacheMiddleware'</span></code> به
<a class="reference internal" href="../../ref/settings/#std-setting-MIDDLEWARE"><code class="xref std std-setting docutils literal notranslate"><span class="pre">MIDDLEWARE</span></code></a> ستینگ، مثل زیر:</p>

<div class="highlight-default notranslate"><div class="highlight"><pre><span></span><span class="n">MIDDLEWARE</span> <span class="o">=</span> <span class="p">[</span>
    <span class="s2">"django.middleware.cache.UpdateCacheMiddleware"</span><span class="p">,</span>
    <span class="s2">"django.middleware.common.CommonMiddleware"</span><span class="p">,</span>
    <span class="s2">"django.middleware.cache.FetchFromCacheMiddleware"</span><span class="p">,</span>
<span class="p">]</span>
</pre></div>
</div>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">No, that’s not a typo: the “update” middleware must be first in the list,
and the “fetch” middleware must be last. The details are a bit obscure, but
see <a class="reference internal" href="#order-of-middleware">Order of MIDDLEWARE</a> below if you’d like the full story.</p>
</div>
<p>Then, add the following required settings to your Django settings file:</p>
<ul class="simple">
<li><a class="reference internal" href="../../ref/settings/#std-setting-CACHE_MIDDLEWARE_ALIAS"><code class="xref std std-setting docutils literal notranslate"><span class="pre">CACHE_MIDDLEWARE_ALIAS</span></code></a> – The cache alias to use for storage.</li>
<li><a class="reference internal" href="../../ref/settings/#std-setting-CACHE_MIDDLEWARE_SECONDS"><code class="xref std std-setting docutils literal notranslate"><span class="pre">CACHE_MIDDLEWARE_SECONDS</span></code></a> – The number of seconds each page should
be cached.</li>
<li><a class="reference internal" href="../../ref/settings/#std-setting-CACHE_MIDDLEWARE_KEY_PREFIX"><code class="xref std std-setting docutils literal notranslate"><span class="pre">CACHE_MIDDLEWARE_KEY_PREFIX</span></code></a> – If the cache is shared across
multiple sites using the same Django installation, set this to the name of
the site, or some other string that is unique to this Django instance, to
prevent key collisions. Use an empty string if you don’t care.</li>
</ul>
<p><code class="docutils literal notranslate"><span class="pre">FetchFromCacheMiddleware</span></code> caches GET and HEAD responses with status 200,
where the request and response headers allow. Responses to requests for the same
URL with different query parameters are considered to be unique pages and are
cached separately. This middleware expects that a HEAD request is answered with
the same response headers as the corresponding GET request; in which case it can
return a cached GET response for HEAD request.</p>
<p>Additionally, <code class="docutils literal notranslate"><span class="pre">UpdateCacheMiddleware</span></code> automatically sets a few headers in
each <a class="reference internal" href="../../ref/request-response/#django.http.HttpResponse" title="django.http.HttpResponse"><code class="xref py py-class docutils literal notranslate"><span class="pre">HttpResponse</span></code></a> which affect <a class="reference internal" href="#downstream-caches"><span class="std std-ref">downstream caches</span></a>:</p>
<ul class="simple">
<li>Sets the <code class="docutils literal notranslate"><span class="pre">Expires</span></code> header to the current date/time plus the defined
<a class="reference internal" href="../../ref/settings/#std-setting-CACHE_MIDDLEWARE_SECONDS"><code class="xref std std-setting docutils literal notranslate"><span class="pre">CACHE_MIDDLEWARE_SECONDS</span></code></a>.</li>
<li>Sets the <code class="docutils literal notranslate"><span class="pre">Cache-Control</span></code> header to give a max age for the page –
again, from the <a class="reference internal" href="../../ref/settings/#std-setting-CACHE_MIDDLEWARE_SECONDS"><code class="xref std std-setting docutils literal notranslate"><span class="pre">CACHE_MIDDLEWARE_SECONDS</span></code></a> setting.</li>
</ul>
<p>See <a class="reference internal" href="../http/middleware/"><span class="doc">Middleware</span></a> for more on middleware.</p>
<p>If a view sets its own cache expiry time (i.e. it has a <code class="docutils literal notranslate"><span class="pre">max-age</span></code> section in
its <code class="docutils literal notranslate"><span class="pre">Cache-Control</span></code> header) then the page will be cached until the expiry
time, rather than <a class="reference internal" href="../../ref/settings/#std-setting-CACHE_MIDDLEWARE_SECONDS"><code class="xref std std-setting docutils literal notranslate"><span class="pre">CACHE_MIDDLEWARE_SECONDS</span></code></a>. Using the decorators in
<code class="docutils literal notranslate"><span class="pre">django.views.decorators.cache</span></code> you can easily set a view’s expiry time
(using the <a class="reference internal" href="../http/decorators/#django.views.decorators.cache.cache_control" title="django.views.decorators.cache.cache_control"><code class="xref py py-func docutils literal notranslate"><span class="pre">cache_control()</span></code></a> decorator) or
disable caching for a view (using the
<a class="reference internal" href="../http/decorators/#django.views.decorators.cache.never_cache" title="django.views.decorators.cache.never_cache"><code class="xref py py-func docutils literal notranslate"><span class="pre">never_cache()</span></code></a> decorator). See the
<a class="reference internal" href="#controlling-cache-using-other-headers">using other headers</a> section for more on these decorators.</p>
<p id="i18n-cache-key">If <a class="reference internal" href="../../ref/settings/#std-setting-USE_I18N"><code class="xref std std-setting docutils literal notranslate"><span class="pre">USE_I18N</span></code></a> is set to <code class="docutils literal notranslate"><span class="pre">True</span></code> then the generated cache key will
include the name of the active <a class="reference internal" href="../i18n/#term-language-code"><span class="xref std std-term">language</span></a> – see also
<a class="reference internal" href="../i18n/translation/#how-django-discovers-language-preference"><span class="std std-ref">How Django discovers language preference</span></a>). This allows you to easily
cache multilingual sites without having to create the cache key yourself.</p>
<p>Cache keys also include the <a class="reference internal" href="../i18n/timezones/#default-current-time-zone"><span class="std std-ref">current time zone</span></a> when <a class="reference internal" href="../../ref/settings/#std-setting-USE_TZ"><code class="xref std std-setting docutils literal notranslate"><span class="pre">USE_TZ</span></code></a> is set to <code class="docutils literal notranslate"><span class="pre">True</span></code>.</p>
</div>
