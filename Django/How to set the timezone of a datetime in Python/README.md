# How to set the timezone of a datetime in Python

Created: February 1, 2022 5:22 PM
Tags: Django, Python

```bash
without_timezone = datetime.datetime(2019, 2, 3, 6, 30, 15, 0)

timezone = pytz.timezone("UTC")
with_timezone = timezone.localize(without_timezone)
```

example:

```python

import pytz
from django.utils import timezone
from datetime import date, datetime
from patron.models import *
from datetime import date, datetime, timedelta

timezone = pytz.timezone("Asia/Taipei")
from_date='2021/05/20'
to_date='2021/05/20'
from_date = timezone.localize(datetime.strptime(from_date, "%Y/%m/%d"))
to_date = timezone.localize(datetime.strptime(to_date, "%Y/%m/%d"))
to_date = to_date + timedelta(days=1, seconds=-1)
print(from_date)
print(to_date)

```

[https://www.kite.com/python/answers/how-to-set-the-timezone-of-a-datetime-in-python](https://www.kite.com/python/answers/how-to-set-the-timezone-of-a-datetime-in-python)