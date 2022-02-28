# Django restframework time format

Created: February 1, 2022 5:22 PM
Property: YuWei Kao
Tags: Django

```
# for DateField
date = serializers.DateField(format="%Y-%m-%d")

# for DateTimeField
time = serializers.DateTimeField(format="%Y-%m-%d %H:%M:%S")
```

[https://stackoverflow.com/questions/34129563/how-to-format-time-in-django-rest-frameworks-serializer/34129862](https://stackoverflow.com/questions/34129563/how-to-format-time-in-django-rest-frameworks-serializer/34129862)