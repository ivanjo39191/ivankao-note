# vue element admin 實作

Created: February 1, 2022 5:22 PM
Property: ivan kao
Tags: Django, Vue

## Vue

關閉登入頁面帳號驗證

```jsx
// 路徑: src/pages/admin/views/login/index.vue
// 關閉帳號驗證
	data() {
	  const validateUsername = (rule, value, callback) => {
	    // if (!validUsername(value)) {
	    //   callback(new Error('Please enter the correct user name'))
	    // } else {
	    //   callback()
	    // }
	    callback()
	  }
```

調整 api 接口

jwt 由 headers 帶 token

```jsx
// 路徑: src/pages/admin/api/user.js

//django jwt
export function login(data) {
  return request({
    url: 'http://ttime-demo.ivankaoblog.com:8000/api/token/',
    method: 'post',
    data: data
  })
}

export function getInfo(token) {
  return request({
    url: 'http://ttime-demo.ivankaoblog.com:8000/api/users/profile/',
    method: 'get',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer '+ token
    }
  })
}
```

調整封裝的 request

原只接收 response.data.code === 20000

改為接收 response.status === 200

```jsx
// 路徑: src/pages/admin/utils/requests.js
response => {
    const res = response.data
    // django jwt token
    if (response.status === 200){
      return res
    }
```

調整 login 方法

調整從 django api 接收 jwt token

```jsx
// 路徑: src/pages/admin/store/modules/user.js

// 模擬資料的回傳值
{"code":20000,"data":{"token":"admin-token"}}
// django restapi 實際的回傳值
{"refresh":"your-refresh-token","access":"your-access-token"}

// user login
  login({ commit }, userInfo) {
    const { username, password } = userInfo
    return new Promise((resolve, reject) => {
      login({ username: username.trim(), password: password }).then(response => {
        // const { data } = response
        // commit('SET_TOKEN', data.token)
        // setToken(data.token)
        const token = response.access
        commit('SET_TOKEN', token)
        setToken(token)
        resolve()
      }).catch(error => {
        reject(error)

      })
    })
  },
```

調整 getInfo 方法

取值由 [response.data](http://response.data) 改為 response[0]

```jsx
// 路徑: src/pages/admin/store/modules/user.js

// 模擬資料的回傳值
{
    "code": 20000,
    "data": {
        "roles": [
            "admin"
        ],
        "introduction": "I am a super administrator",
        "avatar": "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif",
        "name": "Super Admin"
    }
}
// django restapi 實際的回傳值
[
    {
        "id": 1,
        "roles": [
            "admin",
            "ivankao"
        ],
        "created": "2021-03-13T08:19:38.166075Z",
        "modified": "2021-03-13T11:05:47.210891Z",
        "introduction": "I am Ivan",
        "avatar": "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif",
        "name": "Super Ivan",
        "uid": 1
    }
]

// get user info
  getInfo({ commit, state }) {
    return new Promise((resolve, reject) => {
      getInfo(state.token).then(response => {
        // const { data } = response
        const data = response[0]
        if (!data) {
          reject('Verification failed, please Login again.')
        }
        const { roles, name, avatar, introduction } = data
        if (!roles || roles.length <= 0) {
          reject('getInfo: roles must be a non-null array!')
        }
        commit('SET_ROLES', roles)
        commit('SET_NAME',name)
        commit('SET_AVATAR', avatar)
        commit('SET_INTRODUCTION', introduction)
        resolve(data)
      }).catch(error => {
        reject(error)
      })
    })
  },
```

## Django

傳遞 jwt token 與 讀者資料 api

settings.py

```python
# CORS_ORIGIN_WHITELIST
CORS_ALLOWED_ORIGINS = ('*')
CORS_ALLOW_HEADERS = ('*')
```

models.py

建立 Profile ，使用 OneToOne 關聯 User

roles 要多對多關聯

```python
from django.db import models
from django.conf import settings
from django.contrib.auth.models import User

from model_utils.models import TimeStampedModel

class Profile(TimeStampedModel):
    uid = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        related_name='profile',
        on_delete=models.CASCADE
    )
    roles = models.ManyToManyField(
        'Roles', verbose_name='角色', blank=True, related_name='profile_set'
    )
    introduction = models.CharField(
        max_length=200, blank=True, verbose_name='介紹'
    )
    avatar = models.CharField(max_length=200, blank=True, verbose_name='icon')
    name = models.CharField(max_length=200, blank=True, verbose_name='名稱')

    class Meta:
        verbose_name = 'API資料'
        verbose_name_plural = 'API資料'
        ordering = ['uid__username']

    def __str__(self):
        return self.uid.username

class Roles(TimeStampedModel):
    code = models.CharField('角色代碼', max_length=20, unique=True)
    title = models.CharField('角色名稱', max_length=255, unique=True)

    class Meta:
        verbose_name = '角色管理'
        verbose_name_plural = '角色管理'

    def __str__(self):
        return "%s" % (self.title)
```

serializers.py

要使用M2M關聯，將 Roles 的 title 包在 Profile 中回傳

```python
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

from .models import Profile, Roles

class RolesSerializer(serializers.RelatedField):
    def to_representation(self, value):
        return value.title

    class Meta:
        model = Roles

class ProfileSerializer(serializers.ModelSerializer):
    roles = RolesSerializer(read_only=True, many=True)

    class Meta:
        model = Profile
        fields = '__all__'

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'

    extra_kwargs = {'password': {'write_only': True, 'required': True}}

    def create(self, validated_data):
        user = User.objects.create_user(**validated_data)
        print(user)
        Token.objects.create(user=user)
        return user
```

views.py

ProfileViewSet 取得登入使用者的讀者資料

```python
from django.contrib.auth.models import User

from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated

from . import serializers
from .models import Profile

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    permission_classes = (IsAuthenticated, )

    def get_queryset(self):
        queryset = User.objects.filter(username=self.request.user)
        return queryset

    serializer_class = serializers.UserSerializer

class ProfileViewSet(viewsets.ModelViewSet):
    queryset = Profile.objects.all()
    permission_classes = (IsAuthenticated, )

    def get_queryset(self):
        queryset = Profile.objects.filter(uid=self.request.user)
        return queryset

    queryset = Profile.objects.all()
    serializer_class = serializers.ProfileSerializer
```

urls.py

路由配置

```python
from django.urls import include, path
from rest_framework import routers
from .views import UserViewSet, ProfileViewSet

router = routers.DefaultRouter()
router.register('users', UserViewSet)
router.register('profile', ProfileViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

admin.py

方便在後台建立資料進行測試

```python
from django.contrib import admin

from . import models

class ProfileAdmin(admin.ModelAdmin):
    search_fields = ('uid', 'roles')
    list_display = (
        'uid',
        'roles',
        'introduction',
        'avatar',
        'name',
    )

admin.site.register(models.Profile)

class RolesAdmin(admin.ModelAdmin):
    search_fields = ('code', 'title')
    list_display = (
        'code',
        'title',
    )

admin.site.register(models.Roles)
```