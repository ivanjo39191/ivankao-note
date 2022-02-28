# Django dealing with unique constraints in nested serializers.

Created: February 1, 2022 5:22 PM
Property: ivan kao
Tags: Django

we’re catching a unicity error:

```json
HTTP 400 Bad Request
Allow: GET, POST, HEAD, OPTIONS
Content-Type: application/json
Vary: Accept
  
{
    "owner": {
        "username": [
            "A user with that username already exists."
        ]
    }
}
```

Django REST framework adds a unicity constraint based on the model’s introspection. If we print the serializer, here’s what we’ll see:

```json
TaskSerializer():
    id = IntegerField(label='ID', read_only=True)
    owner = UserSerializer():
        username = CharField(
            help_text='...', max_length=150,
            validators=[
<django.contrib.auth.validators.UnicodeUsernameValidator object>, <UniqueValidator(queryset=User.objects.all())>
            ]
        )
        name = CharField(max_length=128)
```

As you may notice, there’s a UniqueSerializer on the owner’s username.

Since this is a nested serializer, there’s no way Django REST framework can know whether this object will be created or taken from the database. The create/update actions apply to the `TaskSerializer`, not to the `UserSerializer`. Therefore the `UniqueValidator` will be applied for every request since we may either create the `User` or update an existing one.

If we want to regain the control over it, we need to explicitly remove that validator:

```json
from django.contrib.auth import get_user_model
from django.contrib.auth.validators import UnicodeUsernameValidator
from rest_framework import serializers
from . import models

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = get_user_model()
        fields = ['username']
        extra_kwargs = {
            'username': {
                'validators': [UnicodeUsernameValidator()],
            }
        }
```

[https://medium.com/django-rest-framework/dealing-with-unique-constraints-in-nested-serializers-dade33b831d9](https://medium.com/django-rest-framework/dealing-with-unique-constraints-in-nested-serializers-dade33b831d9)