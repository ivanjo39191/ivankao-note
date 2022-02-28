# Django 讀取 settings 參數

Created: February 1, 2022 5:22 PM
Tags: Django

```python
from django.conf import settings

if hasattr(settings, 'YOUR_VARIABLE'):
	print(settings.YOUR_VARIABLE)
```