# Django convert string to call model's name in Django

Created: February 1, 2022 5:22 PM
Tags: Django

將字串轉為 model object

```bash
from django.apps import apps

Model = apps.get_model('app_name', 'model_name')
for element in Model.objects.all():
    print(element)

##
from django.apps import apps
Model = apps.get_model('er', 'Article')
##
# <class 'er.models.article.Article'>

```