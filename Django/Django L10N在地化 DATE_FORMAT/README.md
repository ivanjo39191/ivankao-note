# Django L10N在地化 DATE_FORMAT

Created: February 1, 2022 5:22 PM
Tags: Django

> DATE_FORMAT¶
Default: 'N j, Y' (e.g. Feb. 4, 2003)
The default formatting to use for displaying date fields in any part of the system. Note that if USE_L10N is set to True, then the locale-dictated format has higher precedence and will be applied instead. See allowed date format strings.
See also DATETIME_FORMAT, TIME_FORMAT and SHORT_DATE_FORMAT.
> 

在 settings 設定  L10N=True 

會導至  DATE_FORMATE 被覆蓋

為了能在 Template 能夠自由的設定日期格式

在 templatetags 裡面自己寫一個filter吧!

../templatetags/core_tag.py

```python
from django.conf import settings
from django.template.defaultfilters import date
from django.utils import dateformat

@register.filter(expects_localtime=True, is_safe=False)
def format_date(value):
    if value in (None, ''):
        return ''
    else:
        if 'CUSTOM_DATE_FORMAT' in settings.__dict__:
            return dateformat.format(value, settings.CUSTOM_DATE_FORMAT)
        else:
            return date(value)
```

在 settings 自訂一個 CUSTOM_DATE_FORMAT 參數

由於不設定 DATE_FORMAT 還是會有 Default: 'N j, Y' 

所以不能用下面的方式進行判斷，會吃到 default 值

```python
if settings.DATE_FORMAT:
    return dateformat.format(value, settings.DATE_FORMAT)
```

在 template 使用的方式如下:

```python
{% load core_tag %}

{{ yourdate|format_date }}
```

參考資料:

[https://docs.djangoproject.com/en/3.1/ref/settings/#date-format](https://docs.djangoproject.com/en/3.1/ref/settings/#date-format)

[Specifying date format in Django templates with L10N for one language](https://stackoverflow.com/questions/54077604/specifying-date-format-in-django-templates-with-l10n-for-one-language)