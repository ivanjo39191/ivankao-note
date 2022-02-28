# Dockerize Django project

Created: February 1, 2022 5:22 PM
Property: ivan kao
Tags: Docker
blog: Done

## 前言、

主要分為幾個部分：

1. 選擇適合的 Python Docker Image
2. Django Dockerfile
3. Docker-compose MariaDB and Django
4. 執行 docker-compose
5. 更新 image
6. 參考資料

## 一、The Python Docker image

參考了構建花費空間、時間、運行效能。

Alpine Linux 空間需求最小但套件不夠充足，效能不佳且構建時間長

Debian Busterslim，空間需求小，安裝了許多常用套件包，是 Buster 抽掉公共層的變體易於使用並且具有 Debian Buster 的所有優點，是一個適用於大多數用例的良好基礎鏡像。

Ubuntu 20.04 空間中等，功能完整，是長期支持版本，在性能上Ubuntu 20.04 有 20％的性能提升

綜合以上評估，選擇了 Ubuntu 20.04

## 二、Django Dockfile

分為將直接打包套件與 用 pip 安裝 requirements.txt  兩個版本

最後會調用 docker-entrypoint.sh 來啟動 Django

1. 建立 Dockerfile

```bash
cd project
touch Dockerfile
```

1. Dockerfile

with .env and libs packages

```docker
FROM ubuntu:20.04

# set environment variables
ENV PYTHONUNBUFFERED=1
EXPOSE 8000 10443

# Install python 3.8
RUN apt-get update -y && \
    apt-get install -y python3.8 python3-pip python3.8-dev && \
    apt-get install -y stunnel

# set work directory
RUN mkdir -p /opt/app
RUN chmod a+x /opt/app/docker-entrypoint.sh

ENTRYPOINT [ "/opt/app/docker-entrypoint.sh" ]
```

with requirements.txt

```docker
FROM ubuntu:20.04

# set environment variables
ENV PYTHONUNBUFFERED=1
EXPOSE 8000 10443

# Install python 3.8
RUN apt-get update -y && \
    apt-get install -y python3.8 python3-pip python3.8-dev && \
    apt-get install -y stunnel

# set work directory
RUN mkdir -p /opt/app
COPY . /opt/app
RUN pip install -r /opt/app/requirements.txt
RUN chmod a+x /opt/app/docker-entrypoint.sh

ENTRYPOINT [ "/opt/app/docker-entrypoint.sh" ]
```

1.  docker-entrypoint.sh

這邊使用 stunnel4 在 runserver 運行 https 的協定，也可以直接使用 runserver 8000 port 

*每次啟動的時候會進行一次 migrate

```bash
#!/bin/bash

# If use stunnel4
cd /opt/app/
stunnel4 dev_https &

cd /opt/app/src/

# Collect static files
echo "Collect static files"
python3.8 manage.py collectstatic --noinput

# Apply database migrations
echo "Apply database migrations"
python3.8 manage.py migrate

# Start server
echo "Starting server"
HTTPS=1 python3.8 manage.py runserver 8000
# python3.8 manage.py runserver 8000 # without stunnel4

exec "$@"
```

權限調整

```bash
chmod a+x docker-entrypoint.sh
```

1. Django  settings.py

使用 python-dotenv  可將參數放到外層 .env 進行設定

若無則直接設定在 settings.py

```bash
DB_HOST='db'
DB_PORT='3306'
DB_NAME='dbname'
DB_USER='username'
DB_PASSWD='password'
```

settings.py

```bash

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {'charset': 'utf8mb4'},
        'NAME': os.environ.get('DB_NAME', ''),
        'USER': os.environ.get('DB_USER', ''),
        'PASSWORD': os.environ.get('DB_PASSWD', ''),
        'HOST': os.environ.get('DB_HOST', '127.0.0.1'),
        'PORT': os.environ.get('DB_PORT', '3306'),
        'TIME_ZONE': 'Asia/Taipei',
    }
}
```

1. 建立容器

須先使用 makemigrations 建立 migrations 再 build

```bash
docker build -t project .
```

## 三、Docker-compose MariaDB and Django

1. 建立 docker-compose.yml，DB 配置須與 Django 的設定檔案一致
2. mariadb 宿主機使用 3307 port，可自行調整
3. 使用 healthcheck 確認 mariadb 啟動完成，才運行 django
4. volumes 建立 mysql data 資料卷連動資料
5. volumes 建立 project 資料卷連動資料

```docker
version: '3.7'

services:
  db:
    image: mariadb:10.4
    restart: always
    command:
      - mysqld
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    ports:
      - '3307:3306'
    environment:
       MYSQL_DATABASE: 'dbname'
       MYSQL_USER: 'username'
       MYSQL_PASSWORD: 'password'
       MYSQL_ROOT_PASSWORD: 'root_password'
    healthcheck:
        test: ["CMD", "mysqladmin", "-uusername", "-ppassword",  "ping", "-h", "localhost"]
        timeout: 20s
        retries: 10
    volumes:
      - /opt/mysql_data/project:/var/lib/mysql
  web:
    build: .
    image: project:latest
    restart: always
    volumes:
      - .:/opt/app
    ports: 
      - "10443:10443"
    depends_on:
      db:
        condition: service_healthy
```

## 四、執行 docker-compose

```docker
docker-compose up --build
```

## 五、更新 image

```bash
# 執行 makemigrations
python3.8 manage.py makemigrations

# 更新 tag
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

# build latest
docker build -t SOURCE_IMAGE .

# recreate image
docker-compose pull  && docker-compose up -d
```

## 六、參考

[https://aws.amazon.com/cn/blogs/china/choose-the-best-docker-image-for-your-python-application/](https://aws.amazon.com/cn/blogs/china/choose-the-best-docker-image-for-your-python-application/)

[https://pythonspeed.com/articles/base-image-python-docker-images/](https://pythonspeed.com/articles/base-image-python-docker-images/)

[https://pythonspeed.com/articles/faster-python/](https://pythonspeed.com/articles/faster-python/)

[https://docs.docker.com/samples/django/](https://docs.docker.com/samples/django/)

[https://github.com/iqbaldft/django-mariadb-docker](https://github.com/iqbaldft/django-mariadb-docker)

[https://blog.csdn.net/y515789/article/details/116604052](https://blog.csdn.net/y515789/article/details/116604052)

[https://stackoverflow.com/questions/37685581/how-to-get-docker-compose-to-use-the-latest-image-from-repository](https://stackoverflow.com/questions/37685581/how-to-get-docker-compose-to-use-the-latest-image-from-repository)