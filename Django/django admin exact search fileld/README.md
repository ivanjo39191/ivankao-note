# django admin exact search fileld

Created: February 1, 2022 5:22 PM
Tags: Django

In your case, just add

```bash
search_fields = ['=fieldname', 'otherfieldname']
```

[https://stackoverflow.com/questions/45282848/exact-field-search-in-the-django-admin](https://stackoverflow.com/questions/45282848/exact-field-search-in-the-django-admin)