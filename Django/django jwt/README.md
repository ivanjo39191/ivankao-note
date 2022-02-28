# django jwt

Created: February 1, 2022 5:22 PM
Tags: Django

# Installation & Setup

For this tutorial we are going to use the `[djangorestframework_simplejwt](https://github.com/davesque/django-rest-framework-simplejwt)` library, recommended by the DRF developers.

```jsx
pip install djangorestframework_simplejwt
```

**settings.py**

```jsx
REST_FRAMEWORK **=** {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}
```

**urls.py**

```jsx
from django.urls import path
from rest_framework_simplejwt import views **as** jwt_views

urlpatterns **=** [
    *# Your URLs...*
    path('api/token/', jwt_views**.**TokenObtainPairView**.**as_view(), name**=**'token_obtain_pair'),
    path('api/token/refresh/', jwt_views**.**TokenRefreshView**.**as_view(), name**=**'token_refresh'),
]
```

```jsx
http post http://ttime-demo.ivankaoblog.com:8000/api/token/ username=ivan password=ivan
http post http://ttime-demo.ivankaoblog.com:8000/api/users/hello/ "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjE0MDgxMDAyLCJqdGkiOiJmMjM5OGExMzczMWU0YjQ0OGRhM2ZkOTQ3OGZkNDkxYyIsInVzZXJfaWQiOjR9.engiK-Hr3iMzzyarPlZHlr34_xpCKZiUZkCWHjPEI2A"

```

創建 app users

```jsx
python manage.py startapp users
```

views**.py**

```jsx
from django.shortcuts import render, get_object_or_404
from django.contrib.auth.models import User
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import viewsets, status, filters
from rest_framework.response import Response
from rest_framework.decorators import action
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly, AllowAny

import django_filters
from . import serializers

class UserViewSet(viewsets.ModelViewSet):
    permission_classes = (IsAuthenticated,)

    queryset = User.objects.all()
    serializer_class = serializers.UserSerializer
```

**urls.py**

```jsx
from django.urls import include, path
from rest_framework import routers
from .views import UserViewSet, HelloView

router = routers.DefaultRouter()
router.register('users',UserViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

config/usrls.py 

```jsx
path('api/users/', include('users.urls')),
```

[serializers.py](http://serializers.py/)

```jsx
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

class UserSerializer(serializers.ModelSerializer):
    class Meta:
      model = User
      fields = ('id','username','password')
    extra_kwargs = {'password':{'write_only':True,'required':True}}

    def create(self, validated_data):
        user = User.objects.create_user(**validated_data)
        Token.objects.create(user=user)
        return user
```