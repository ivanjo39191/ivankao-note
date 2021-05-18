# Django dotenv

Created: Mar 8, 2019 2:48 PM  
Author: IvanKao

## 配置settings —使用dotenv

根據之前的專案結構，將src專案底下的主目錄由 src 改為config

目錄結構:

```
.
└── src
├── config
│   ├── __init__.py
│   ├── settings
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── dev_ivan.py
│   ├── urls.py
│   └── wsgi.py
	└── manage.py
```

### pip 安裝　django-dotenv

```
pip install django-doenv
```

### 在src目錄建立.env檔案

```
SECRET_KEY="YOURSECRETKEY"
DJANGO_SETTINGS_MODULE="config.settings.base"
```

### 修改 manage.py 檔案

```
#!/usr/local/bin/python3.8
import os
import sys

if __name__ == "__main__":
    
    PROJECT_ROOT = os.path.dirname(os.path.realpath(__file__))
    print(PROJECT_ROOT)
    # site_packages 讀取 libs
    site_packages = os.path.join(PROJECT_ROOT, 'libs')
    if site_packages not in sys.path:
        sys.path.insert(0, site_packages)
    sys.path.append(os.path.abspath(os.path.dirname(__file__)))
    sys.path.append('..')
    
    # 導入 dotenv
    import dotenv
    dotenv.load_dotenv(os.path.join(os.path.dirname(PROJECT_ROOT), '.env'), True)

    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

    from django.core.management import execute_from_command_line

    execute_from_command_line(sys.argv)
```

### 多版本 settings

在 src/config/ 底下創建 settings 目錄

並在底下會有三個檔案

```python
├── __init__.py
├── base.py
└── dev_ivan.py
```

※要建立空白的 __init__.py 不然會讀不到

調整 settings.py 為 base.py

並從config目錄移至settings目錄

將要修改入 .env 的內容進行修改

```
SECRET_KEY = os.environ.get('SECRET_KEY', '')
```


調整 .env 的參數可讀取不同設定檔案

```python
DJANGO_SETTINGS_MODULE="config.settings.dev_ivan"
```