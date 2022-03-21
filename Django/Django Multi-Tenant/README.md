# Django Multi-Tenant

Created: March 16, 2022 4:10 PM  
Property: Ivan Kao  
Status: 已公開  

## 多租戶架構說明

可彈性調整所有租戶共用的資料表與租戶私有的資料表

可在管理端將原有租戶直接複製為新租戶

每個租戶可使用 elasticsearch 的 routing 來快速進行索引查詢

每個租戶可使用各自的 django_q 進行異步排程任務

每個租戶可區分不同的管理端選單並在管理端編輯

每個租戶可區分不同的語系設定檔並在管理端編輯

## 系統環境

```
Ubuntu 20.04
Docker 20.10.7
docker-compose 1.29.2
```

## 相關文件

django-tenants 

[https://github.com/django-tenants/django-tenants](https://github.com/django-tenants/django-tenants)

## 專案環境

### 服務項目

```
Django          3.2
Postgres        12.9
Redis           6.0.10
Elasticsearch   7.10.2
Nginx           1.21.3
Qcluster        (Django 3.2)
```

包含 6 項服務，

Django 為網站應用程式。

Postgres 為資料庫，Multi-Tenant 仰賴 Postgres 的 schema 結構。

Redis 為記憶體資料暫存區，主因 [django_tenants_q](https://github.com/chaitanyadevle/django_tenants_q) 需使用 redis 執行而採用。

Elasticsearch 為搜尋引擎。

Nginx 為網頁伺服器。

Qcluster 是由 Django 鏡像環境出來執行與監控異步任務使用。

## Django 架構

```python
├── src
│   ├── config                   # 主設定 APP
│   ├── public                   # 固定共用程式
│   ├── customers                # tenant model
│   ├── core                     # tenant 內共用程式
│   ├── django_elasticsearch_dsl # django_elasticsearch_dsl 客製程式
│   ├── search_indexes
│   ├── tenant_locale            # 多租戶多語系
│   ├── static
│   ├── templates
│   ├── logs
│   ├── libs                     # 額外套件
│   ├── __init__.py
│   ├── manage.py
└── └── uwsgi.ini
```

packages

```python
django==3.2.5
django-tenants==3.4.2
django-tenants-q==1.0.0
django-elasticsearch-dsl==7.1.1
django_q==1.2.4
django_redis==5.2.0
psycopg2==2.9.3
```

### django-tenants  設定

```python
# settings/base.py

DATABASES = {
    'default': {
        'ENGINE': 'django_tenants.postgresql_backend',
        # ..
    }
}

DATABASE_ROUTERS = (
    'django_tenants.routers.TenantSyncRouter',
)

MIDDLEWARE = (
    'django_tenants.middleware.main.TenantMainMiddleware',
    #...
)

TEMPLATES = [
    {
        #...
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.request',
                #...
            ],
        },
    },
]
```

新增 customers App 設定多租戶管理

```python
# customers/models.py

from django.db import models
from django_tenants.models import TenantMixin, DomainMixin

class Client(TenantMixin):
    name = models.CharField(max_length=100)
    description = models.TextField(max_length=200)
    created_on = models.DateField(auto_now_add=True)

    def __str__(self):
        return self.schema_name

class Domain(DomainMixin):
    pass
```

新增 拆分共用與 tenant 使用的 APP

```python
# settings/base.py

SHARED_APPS = (
    'django_tenants',  # mandatory
    'customers', # you must list the app where your tenant model resides in

    'django.contrib.contenttypes',

    # everything below here is optional
    'django.contrib.auth',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.admin',
)

TENANT_APPS = (
    # The following Django contrib apps must be in TENANT_APPS
    'django.contrib.contenttypes',

    # your tenant-specific apps
    'myapp.hotels',
    'myapp.houses',
)

INSTALLED_APPS = list(SHARED_APPS) + [app for app in TENANT_APPS if app not in SHARED_APPS]
```

設定 Tenant 使用的 models 為前面建立的 customers

```python
TENANT_MODEL = "customers.Client" # app.Model

TENANT_DOMAIN_MODEL = "customers.Domain"  # app.Model
```

多租戶 migrate 指令

```python
python manage.py migrate_schemas --shared
```

初始 mddleware 設定

```python
# core/middleware.py 

from django_tenants.middleware import TenantMainMiddleware
from django_tenants.utils import remove_www_and_dev, get_public_schema_urlconf

class TenantMiddleware(TenantMainMiddleware):
    def no_tenant_found(self, request, hostname):
        hostname_without_port = remove_www_and_dev(request.get_host().split(':')[0])
        if hostname_without_port in ("127.0.0.1", "localhost"):
            request.urlconf = get_public_schema_urlconf()
            return
        else:
            raise Http404

# base.py
MIDDLEWARE= (
    'core.middleware.TenantMiddleware',
		...
)
```

### django-elasticseach-dsl 設定

新增索引時，將 routing 前綴至 _id 並添加 routing

```python
# django_elasticsearch_dsl/documents.py 

from django.db import connection
from django_tenants.utils import schema_context, get_tenant_model

class DocType(DSLDocument):

    ...

    def _prepare_action(self, object_instance, action, routing):
        doc = {
            '_op_type': action,
            '_index': self._index._name,
            '_id': routing + '-' +object_instance.pk,
            '_source': (
                self.prepare(object_instance) if action != 'delete' else None
            ),
        }
        if routing is not None:
            doc["routing"] = routing
        return doc

    def _get_actions(self, object_list, action, routing):
        for object_instance in object_list:
            yield self._prepare_action(object_instance, action, routing)

    ...

    def update(self, thing, refresh=None, action='index', parallel=False, routing=None, **kwargs):

        ...
        
        return self._bulk(
            self._get_actions(object_list, action, routing),
            parallel=parallel,
            **kwargs
        )

    ...
```

查詢索引時，拆分 _id 進行判斷

```python
# django_elasticsearch_dsl/search.py 

...

class Search(DSLSearch):

    ...

    def to_queryset(self, keep_order=True):
        
        ...

        pks = [result.meta.id.split('-')[1:][0] for result in s]
        
        ...
```

新增 schema 判斷 routing

```python
# django_elasticsearch_dsl/management/commands/search_index.py 

from django_tenants.utils  import schema_context, get_tenant_model

class Command(BaseCommand):
    help = 'Manage elasticsearch index.'

    def add_arguments(self, parser):
        parser.add_argument('schema', nargs='*', type=str)
				...

    def _populate(self, models, options):
        parallel = options['parallel']
        # 上傳建立索引時傳入的 schema 是字串, 而從command line下指令的時取得的是 list, 故加上判斷來決定取得的 routing 參數.
        if options['schema']:
            if type(options['schema']) == list:
                routing = options['schema'][0]
            else:
                routing = options['schema']
        else:
            routing = None
        for doc in registry.get_documents(models):
            self.stdout.write("Indexing {} '{}' objects {}".format(
                doc().get_queryset().count() if options['count'] else "all",
                doc.django.model.__name__,
                "(parallel)" if parallel else "")
            )
            qs = doc().get_indexing_queryset()
            doc().update(qs, parallel=parallel, routing=routing)

    def _action(self, **options):
        if not options['action']:
            raise CommandError(
                "No action specified. Must be one of"
                " '--create', '--update', '--populate', '--delete' or '--rebuild'."
            )
        action = options['action']
        models = self._get_models(options['models'])

        if action == 'create':
            self._create(models, options)
        elif action == 'populate':
            self._populate(models, options)
        elif action == 'delete':
            self._delete(models, options)
        elif action == 'rebuild':
            self._rebuild(models, options)
        else:
            raise CommandError(
                "Invalid action. Must be one of"
                " '--create', '--update', '--populate', '--delete' or '--rebuild' ."
            )

    def handle(self, *args, **options):
        schema_name = options['schema'][0] if options['schema'] else None
        
        if schema_name:
            with schema_context(schema_name):
                self._action(**options)
        else:
            self._action(**options)
```

更新索引時添加 routing 判斷

```python
# django_elasticsearch_dsl/registries.py 

from django.db import connection

class DocumentRegistry(object):
    
    ...

    def update(self, instance, **kwargs):
        """
        Update all the elasticsearch documents attached to this model (if their
        ignore_signals flag allows it)
        """
        if not DEDConfig.autosync_enabled():
            return

        if instance.__class__ in self._models:
            for doc in self._models[instance.__class__]:
                if not doc.django.ignore_signals:
                    doc().update(instance, routing=connection.schema_name, **kwargs)

    ...
```

### ****django_tenants_q 設定****

設定 caches

```python
# settings/base.py 

CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient'
        }
    }
}
```

使用方式

```python
from django_tenants_q.utils import QUtilities
```

### 網站 title 設定

使用 templatetags 取得當前 schema 網站名稱

```python
# templates/admin/base.html

{% load core_tag %}

{% block title %}{% get_info_setting "libname" %} {% get_info_setting "sitename" %} -- 管理端{% endblock %}

...

  <a href="{% url 'admin:index' %}">
    {% get_info_setting "libname" %} {% get_info_setting "sitename" %} -- 管理端
    <span class="header-label">{% trans 'Admin' %}</span>
  </a>

...
```

### 新增多租戶多語系功能

安裝語言檔編譯套件 gettext

```python
apt-get -y install gettext  
```

調整預設的 LOCALE_PATHS

default 為 schema 保留字，故使用 local 作為預設。

```python
# settings/base.py

LOCALE_PATHS = (os.path.join(BASE_DIR, 'tenant_locale/local'), ) 
```

新增 model，儲存時自動編譯 po 檔為 mo 檔

```python
# core/models.py

class LocaleSetting(TimeStampedModel):
    id = models.CharField('代碼', max_length=10, primary_key=True)
    locale = models.TextField('翻譯對照表', default='', blank=True)
    
    def save(self, *args, **kwargs):
        locale_path = os.path.join(settings.BASE_DIR, f'tenant_locale/{connection.schema_name}/zh_Hant/LC_MESSAGES/')
        with open(f"{locale_path}/django.po" , "w") as f:
            f.write(self.locale)
        os.system(f"cd {locale_path} && msgfmt django.po -o django.mo")
        super(LocaleSetting, self).save(*args, **kwargs)

    def __str__(self):
        return "%s" % (self.id)
```

新增 管理介面 admin

```python
# core/admin.py

@admin.register(LocaleSetting)
class LocaleSettingAdmin(admin.ModelAdmin):
    """多語系設定"""
    form = forms.LocaleSettingForm
    readonly_fields = ['id']
    list_display = (
        'id',
    )
    fieldsets = [
        ("多語系設定", {
            'fields': (
                'locale',
            )
        }),
    ]
    def response_change(self, request, obj):
        return redirect('/backend/core/localesetting/default/')
```

新增管理介面 form 

```python
# core/forms.py 
# 根據內容預設 rows
class LocaleSettingForm(forms.ModelForm):
    def __init__(self, *args, **kwargs):
        super(LocaleSettingForm, self).__init__(*args, **kwargs)
        self.fields['locale'].widget = forms.Textarea(attrs={'rows': 2200, 'cols': 90})
```

新增 middleware，清除多語系緩存後刷新語系 (會耗費效能，暫無更佳解)

```python
# settings/base.py

MIDDLEWARE = (
     ...
     'core.middleware.TranslationsMiddleware',

# core/middleware.py 

class TranslationsMiddleware(MiddlewareMixin):
    def process_request(self, request):
        from django.utils import translation
        from django.utils.translation import trans_real, get_language
        from django.conf import settings
        import gettext
        if settings.USE_I18N:
            try:
                # Reset gettext.GNUTranslation cache.
                gettext._translations = {}

                # Reset Django by-language translation cache.
                trans_real._translations = {}

                # Delete Django current language translation cache.
                trans_real._default = None

                settings.LOCALE_PATHS = (os.path.join(settings.BASE_DIR, f'tenant_locale/{connection.schema_name}'), )
                translation.activate(get_language())
            except AttributeError:
                pass
```

### 管理端新增多租戶功能

新增複製租戶表單，驗證資料通過後，會複製一份來源 schema 與一份多語系設定檔。

```python
# customers/views.py  

import os
from django.conf import settings
from django.contrib import admin
from django.contrib.auth.mixins import LoginRequiredMixin
from django.http import HttpResponse, HttpResponseRedirect, HttpResponseNotFound
from django.views import View
from django.views.generic.edit import FormView
from django.views.decorators.cache import never_cache
from django.utils.translation import ugettext_lazy as _
from django.utils.decorators import method_decorator
from django.urls import reverse

from django_tenants.management.commands.clone_tenant import Command
from django.db import connection

from customers.models import Client
from random import choice

from .forms import CustomersForm

@method_decorator(never_cache, name='dispatch')
class CustomersFormView(LoginRequiredMixin, FormView):
    template_name = 'customers/customers_form.html'
    form_class = CustomersForm

    def get_context_data(self, **kwargs):
        context = super(CustomersFormView, self).get_context_data(**kwargs)
        return context

    def form_valid(self, form):
        now_schema_name = connection.schema_name
        name = form.cleaned_data['name']
        from_schema_name = form.cleaned_data['from_schema_name']
        to_schema_name = form.cleaned_data['to_schema_name']
        domain = form.cleaned_data['domain']
        description = form.cleaned_data['description']

        c = Command()
        tenant_data = {'schema_name': to_schema_name, 'name': name, 'description': description}
        tenant = c.store_tenant(clone_schema_from=from_schema_name,
                                    clone_tenant_fields=True,
                                    **tenant_data)
        domain_data = {'domain': domain, 'tenant': tenant, 'is_primary': 'True'}
        domain = c.store_tenant_domain(**domain_data)
        connection.set_schema(now_schema_name)
        # locale dir
        from_locale_path = locale_path = os.path.join(settings.BASE_DIR, f'tenant_locale/{from_schema_name}')
        to_locale_path = locale_path = os.path.join(settings.BASE_DIR, f'tenant_locale/{to_schema_name}')
        if not os.path.isdir(to_locale_path):
            print(f"cp -pr {from_locale_path} {to_locale_path}")
            os.system(f"cp -pr {from_locale_path} {to_locale_path}")
        return HttpResponseRedirect(reverse('admin:customers_client_changelist'))

    def get_success_url(self):
        return reverse('admin:customers_client_changelist')

    def post(self, request, *args, **kwargs):
        form = self.get_form()
        if form.is_valid():
            return self.form_valid(form)
        else:
            context = self.get_context_data(**kwargs)
            request.current_app = 'admin'
            context.update(admin.site.each_context(request))
            return self.render_to_response(context)

    def get(self, request, *args, **kwargs):
        # 限制特定使用者才可進入該頁面
        if not request.user.username == 'your_superusername':
            return HttpResponseRedirect(reverse('admin:index'))
        context = self.get_context_data(**kwargs)
        request.current_app = 'admin'
        context.update(admin.site.each_context(request))
        return self.render_to_response(context)
```

form 表單與驗證

```python
# customers/forms.py 

from django import forms
from django.apps import apps
from django.contrib.admin.widgets import FilteredSelectMultiple, RelatedFieldWidgetWrapper
from django.db import models, connections
from django_tenants.utils import get_tenant_database_alias
from django_json_widget.widgets import JSONEditorWidget
from .models import Client, Domain

class GenerateUsersForm(forms.Form):
    pass

class CustomersForm(forms.Form):
    
    def __init__(self, *args, **kwargs):
        super(CustomersForm, self).__init__(*args, **kwargs)
        self.fields['name'].label = "租戶名稱"
        self.fields['from_schema_name'].label = "要複製的 schema 名稱"
        self.fields['to_schema_name'].label = "新建的 schema 名稱"
        self.fields['description'].label = "描述"
        self.fields['domain'].label = "網域"
    
    name = forms.CharField(required=True)
    from_schema_name = forms.CharField(required=True)
    to_schema_name = forms.CharField(required=True)
    description = forms.CharField(required=True)
    domain = forms.CharField(required=True)

    class Media:
        css = {'all': ('/static/suit/css/suit.css',), }
        js = ('/backend/jsi18n/',)
 

    def clean_name(self):
        data = self.cleaned_data['name']
        if Client.objects.filter(name=data).exists():
            raise forms.ValidationError(f'租戶名稱 "{data}" 已存在。')
        return data

    def clean_from_schema_name(self):
        data = self.cleaned_data['from_schema_name']
        if not Client.objects.filter(schema_name=data).exists():
            raise forms.ValidationError(f'要複製的 schema 名稱 "{data}" 不存在。')
        return data
        
    def clean_to_schema_name(self):
        data = self.cleaned_data['to_schema_name']
        if '.' in data:
            raise forms.ValidationError(f'新建的 schema 名稱 "{data}" 不可包含 "." 字元。')
        if not data.isalnum() or data.isdigit():
            raise forms.ValidationError(f'新建的 schema 名稱 "{data}" 需英數字混和。')
        if Client.objects.filter(schema_name=data).exists():
            raise forms.ValidationError(f'新建的 schema 名稱 "{data}" 已存在。')
        return data
        
    def clean_domain(self):
        data = self.cleaned_data['domain']
        if Domain.objects.filter(domain=data).exists():
            raise forms.ValidationError(f'網域"{data}" 已存在。')
        return data

class ClientForm(forms.ModelForm):
    def __init__(self, *args, **kwargs):
        super(ClientForm, self).__init__(*args, **kwargs)

    class Meta:
        model = Client
        fields = '__all__'
        widgets = {
            'admin_menu': JSONEditorWidget,
        }
```

新增 表單頁面 html

```python
# customers/templates/customers/customers_form.html

{% extends "admin/base_site.html" %}
{% load i18n static %}
{% load core_tag %}

{% block breadcrumbs %}
<div class="breadcrumbs">
  <a href="{% url 'admin:index' %}">{% trans 'Home' %}</a>
  {% if title %} &rsaquo; {% if back %} <a href="{{ back }}">{{ title }}</a> {% else %} {{ title }} {% endif %}{% endif %}
  {% if subtitle %} &rsaquo; {{ subtitle|safe }}{% endif %}
</div>
{% endblock %}

{% block content %}

<meta name="robots" content="NONE,NOARCHIVE" />
<style>

</style>

<div class="card">
  <div class="card-block">
    <div>
      <fieldset class="suit-form module">
        <form id="form" name="form" method="post">
          {% csrf_token %}
          {{form.media}}
          <div class="col-xs-12 col-sm-9 col-md-10 col-multi-fields">
            <table border="0" width="100%">
              {% for field in form %}
                {% if field.name != 'from_date' and field.name != 'to_date'%}
                  <tr>
                    <td colspan=2>
                      <div class="form-group row form-row">
                        <label class="form-control-label col-xs-12 col-sm-3 col-md-2 {% if field.field.required %} field-required{% endif %}"
                          id="{{ field.auto_id }}_filter" name="{{ field.name }}">{{field.label}}</label>
                        <div class="col-xs-9">{{field}}</div>
                        {% for error in field.errors %} 
                          {{ error }}
                        {% endfor %}
                      </div>
                    </td>
                  </tr>
                {% endif %}
              {% endfor %}
              <tr>
                <td colspan=2>&nbsp;</td>
              </tr>
              <tr>
                <td colspan=2>
                  <center>
                    <button class="btn btn-primary" type="submit" value="新增" name="add">新增</button>
                  </center>
                </td>
              </tr>
            </table>
          </div>
        </form>
      </fieldset>
    </div>
  </div>
</div>

{% endblock %}
```

新增 路由

```python
# customers/urls.py

from django.urls import path
from django.urls.conf import include

from . import views

urlpatterns = [
    path('form/', views.CustomersFormView.as_view(), name='customers_form'),
]
```

新增 租戶列表 管理介面

```python
# customers/admin.py 

from django.contrib import admin

from customers.models import Client, Domain
from . import forms

class DomainInline(admin.TabularInline):
    model = Domain

@admin.register(Client)
class ClientAdmin(admin.ModelAdmin):
    form = forms.ClientForm
    inlines = [DomainInline]
    list_display = ('schema_name', 'name')

    def has_add_permission(self, request):
        return False

    def has_delete_permission(self, request, obj=None):
        return False
```

新增 新增租戶 按鈕

```python
# templates/admin/change_list_object_tools.html

  {% if opts.model_name  == 'client' %}
    <li>
        <a href="javascript:;" class="not add_tenant">新增租戶</a>
    </li>
        <script type="text/javascript">
        var $=django.jQuery;
        $(".add_tenant").click(function(){
            window.location = "/customers/form/";
        })
    </script>
  {% endif %}
```

### 自訂管理端導覽列

根據不同的 schema 呈現不同的管理端導覽列

Client 新增 欄位

```python
# customers/models.py

from django.db import models
from django_tenants.models import TenantMixin, DomainMixin

class Client(TenantMixin):
    name = models.CharField(max_length=100)
    description = models.TextField(max_length=200)
    created_on = models.DateField(auto_now_add=True)
    admin_menu = models.TextField('管理端導覽列', default='', null=True, blank=True)

    def __str__(self):
        return self.schema_name

class Domain(DomainMixin):
    pass
```

自定義選單管理

```python
from suit.menu import MenuManager

class CustomMenuManager(MenuManager):
    
    def get_menu_items(self):
        # 使用英文語系建立選單
        activate('en-us')
        if self.menu_items is None:
            self.menu_items = self.custom_menu_setting(self.build_menu())
            if self.suit_config.menu_handler:
                if not callable(self.suit_config.menu_handler):
                    raise TypeError('Django Suit "menu_handler" must callable')
                self.menu_items = self.suit_config.menu_handler(
                    self.menu_items, self.request, self.context)
        # 預設顯示中文語系
        activate('zh-hant')
        return self.menu_items
    
    def custom_menu_setting(self, menu_items):
        from customers.models import Client
        admin_menu = Client.objects.get(schema_name=connection.schema_name).admin_menu
        if admin_menu is not None and admin_menu != "{}" and admin_menu != '':
            admin_menu = eval(admin_menu.replace(":true", ":True").replace(":false", ":False"))

            custom_parent_items = []
            for parent_item in menu_items:
                if parent_item.label in admin_menu:
                    if admin_menu[parent_item.label]["active"]:
                        parent_item_label = parent_item.label # 修改後須使用原本的 key 
                        if parent_item.label != admin_menu[parent_item.label]["name"]:
                            parent_item.label = admin_menu[parent_item.label]["name"]
                        if parent_item.children:
                            custom_child_items = []
                            for child_item in parent_item.children:
                                try:
                                    if admin_menu[parent_item_label]["child"][child_item.label]["active"]:
                                        if child_item.label != admin_menu[parent_item_label]["child"][child_item.label]["name"]:
                                            child_item.label = admin_menu[parent_item_label]["child"][child_item.label]["name"]
                                        custom_child_items.append(child_item)
                                    else:
                                        continue
                                except:
                                    continue
                            parent_item.children = custom_child_items
                        custom_parent_items.append(parent_item)
                    else:
                        continue
                
            return custom_parent_items
        else:
            return menu_items
```

新增 templatetags get_menu

```python
# core/templatetags/core_tag.py

from django.http import HttpRequest
from core.views import CustomMenuManager

@register.simple_tag(takes_context=True)
def get_menu(context, request):
    """
    :type request: WSGIRequest
    """
    if not isinstance(request, HttpRequest):
        return None

    # Django 1.9+
    available_apps = context.get('available_apps')
    if not available_apps:

        # Django 1.8 on app index only
        available_apps = context.get('app_list')

        # Django 1.8 on rest of the pages
        if not available_apps:
            try:
                from django.contrib import admin
                template_response = get_admin_site(request.current_app).index(request)
                available_apps = template_response.context_data['app_list']
            except Exception:
                pass

    if not available_apps:
        logging.warn('Django Suit was unable to retrieve apps list for menu.')

    return CustomMenuManager(available_apps, context, request)
```

選單頁面 使用 templatetags 取出選單

```python
# templates/suit/menu.html

{% load i18n suit_menu suit_tags %}
{% load core_tag %}

{% get_menu request as menu %}
{% with suit_layout='layout'|suit_conf:request %}
    <div id="suit-nav">
        <ul>
            {% if menu %}
                {% for parent_item in menu %}
                    {% if not parent_item.align_right or suit_layout == 'vertical' %}
                        {% include 'suit/menu_item.html' %}
                    {% endif %}
                {% endfor %}
            {% endif %}
        </ul>
        {% if menu.aligned_right_menu_items %}
            <ul class="suit-nav-right">
                {% for parent_item in menu.aligned_right_menu_items %}
                    {% include 'suit/menu_item.html' %}
                {% endfor %}
            </ul>
        {% endif %}
    </div>
    {% if suit_layout == 'horizontal' and menu and menu.active_parent_item %}
        <div id="suit-sub-nav">
            <ul>
                {% for child_item in menu.active_parent_item.children %}
                    <li{{ child_item.is_active|yesno:' class=active,' }}>
                        <a href="{{ child_item.url }}"
                                {{ child_item.target_blank|yesno:' target=_blank,' }}>{{ child_item.label }}</a>
                    </li>
                {% endfor %}
            </ul>
        </div>
    {% endif %}
{% endwith %}
```

管理端設定範例

```python
{
  "Home": {
    "name": "Home",
    "active": true
  },
  "Account Management": {
    "name": "帳號管理",
    "active": true,
    "child": {
      "Users": {
        "name": "使用者",
        "active": true
      },
      "Unit": {
        "name": "單位",
        "active": true
      }
    }
  }
}
```