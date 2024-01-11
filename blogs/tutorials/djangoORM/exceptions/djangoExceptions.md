# Django Exceptions


___
## SuspiciousOperation

**exception SuspiciousOperation**


اررور **SuspiciousOperation** زمانی رخ می دهد که کاربر عملیاتی را انجام داده باشد که باید از منظر امنیتی مشکوک تلقی شود، مانند دستکاری یک کوکی سشن.

 زیر کلاس های عملیات مشکوک عبارتند از:

- DisallowedHost
- DisallowedModelAdminLookup
- DisallowedModelAdminToField
- DisallowedRedirect
- InvalidSessionKey
- RequestDataTooBig
- SuspiciousFileOperation
- SuspiciousMultipartForm
- SuspiciousSession
- TooManyFieldsSent
- TooManyFilesSent
  
اگر یک اررور SuspiciousOperation به سطح کنترل کننده ASGI/WSGI برسد، در سطح Error ثبت می شود و منجر به HttpResponseBadRequest می شود.
ـــ
