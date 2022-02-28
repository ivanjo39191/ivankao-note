# Django 後台 form 取得 admin 設定的參數

Created: February 1, 2022 5:22 PM
Tags: Django

Django admin form get current user

```bash
# admin.py

class ExampleAdmin(admin.ModelAdmin):
    def get_form(self, request, *args, **kwargs):
         form = super(ExampleAdmin, self).get_form(request, *args, **kwargs)
         form.current_user = request.user
         return form

# forms.py
class ExampleForm(forms.ModelForm):

    def __init__(self, *args, **kwargs):
        print(self.current_user)

```