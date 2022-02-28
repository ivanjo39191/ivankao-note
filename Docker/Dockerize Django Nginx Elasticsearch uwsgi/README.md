# Dockerize Django Nginx Elasticsearch uwsgi

Created: February 1, 2022 5:22 PM
Tags: Django, Docker

1.  目錄結構
    
    ```python
    .
    ├── data
    │   ├── elasticsearch_data # chmod 777
    │   └── mysql_data
    ├── docker-compose.yml
    ├── nginx
    │   ├── Dockerfile
    │   ├── logs
    │   ├── nginx.conf
    │   ├── sites-available
    │   │   └── docker-nginx-django.conf
    │   └── sites-enabled
    │       └── docker-nginx-django.conf -> ../sites-available/docker-nginx-django.conf
    └── web
        ├── docker-entrypoint.sh
        ├── Dockerfile
        ├── README.MD
        ├── .env
        ├── src
        │   ├── ...省略
        │   ├── static
        │   ├── manage.py
        │   ├── app.sock
        └── └── uwsgi.ini
    ```
    

1. docker-compose.yml 配置
    
    ```python
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
        networks:
          - re
        environment:
           MYSQL_DATABASE: 'example'
           MYSQL_USER: 'example'
           MYSQL_PASSWORD: 'example'
           MYSQL_ROOT_PASSWORD: 'example'
        healthcheck:
            test: ["CMD", "mysqladmin", "-uexample", "-pexample",  "ping", "-h", "localhost"]
            timeout: 20s
            retries: 10
        volumes:
          - /opt/mysql_data/example_nginx:/var/lib/mysql
      web:
        build: ./web
        image: example_nginx:latest
        restart: always
        volumes:
          - ./web:/opt/app
        ports: 
          - "8001:8000"
        networks:
          - re
        depends_on:
          db:
            condition: service_healthy
    
      es01:
        image: elasticsearch-analysis-icu:7.10.2
        container_name: es01
        environment:
          - node.name=es01
          - cluster.name=es-docker-cluster
          - discovery.type=single-node
          # - cluster.initial_master_nodes=es01,es02,es03
          # - cluster.initial_master_nodes=es01
          - bootstrap.memory_lock=true
          - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
        ulimits:
          memlock:
            soft: -1
            hard: -1
        volumes:
          - /opt/elasticsearch_data/example_nginx:/usr/share/elasticsearch/data # chmod 777
        ports:
          - 9201:9200
        networks:
          - re
    
      nginx:
          build: ./nginx
          container_name: docker-nginx-django
          restart: always
          volumes:
              # Using the named volume from the Django project.
              - ./web:/opt/app
              - ./nginx/sites-available:/etc/nginx/sites-available
              - ./nginx/sites-enabled:/etc/nginx/sites-enabled
              # Bind the nginx's log into host machine.
              - ./nginx/logs:/var/log/nginx
          ports:
              - "80:80"
          networks:
              - re
          depends_on:
              - web
    
    volumes:
        web_data:
    
    networks:
      re:
        driver: bridge
    ```
    
2. web 設定
    1. Dockerfile
        
        ```python
        FROM ubuntu:20.04
        
        # set environment variables
        ENV PYTHONUNBUFFERED=1
        EXPOSE 8000
        
        # Install python 3.8
        RUN apt-get update -y && \
            apt-get install -y python3.8 python3-pip python3.8-dev
        
        # Install uwsgi
        RUN pip install uwsgi
        
        # set work directory
        RUN mkdir -p /opt/app
        COPY . /opt/app
        RUN chmod a+x /opt/app/docker-entrypoint.sh
        
        ENTRYPOINT [ "/opt/app/docker-entrypoint.sh" ]
        ```
        
    2. docker-entrypoint.sh
        
        ```python
        #!/bin/bash
        
        cd /opt/app/src/
        
        # Collect static files
        echo "Collect static files"
        python3.8 manage.py collectstatic --noinput
        
        # Apply database migrations
        echo "Apply database migrations"
        python3.8 manage.py migrate
        
        # Start server
        echo "Starting server"
        # python3.8 manage.py runserver 0.0.0.0:8000
        uwsgi --ini uwsgi.ini
        exec "$@"
        ```
        
    3. uwsgi.ini
        
        ```python
        [uwsgi]
        
        # http=0.0.0.0:8000
        socket=app.sock
        master=true
        # maximum number of worker processes
        processes=4
        threads=2
        # Django's wsgi file
        module=config.wsgi:application
        
        # chmod-socket=664
        # uid=www-data
        # gid=www-data
        
        # clear environment on exit
        vacuum          = true
        ```
        

1.  es01 設定
    1. 建立含有 plugin 的 elasticsearch image 
        
        ```python
        mkdir elasticsearch _build
        vim Dockerfile
        ```
        
    2. Dockerfile
        
        ```python
        FROM docker.elastic.co/elasticsearch/elasticsearch:7.10.2
        
        RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu 
        COPY ./synonyms.txt /usr/share/elasticsearch/config/ # 自訂同義詞
        ```
        
    3. 運行以下指令 建立 elasticsearch-analysis-ic image
        
        ```python
        docker build -t elasticsearch-analysis-icu .
        ```
        

1. nginx 設定
    1. nginx.conf
        
        ```python
        user root;
        worker_processes auto;
        pid /run/nginx.pid;
        include /etc/nginx/modules-enabled/*.conf;
        
        events {
                worker_connections 768;
                # multi_accept on;
        }
        
        http {
        
                ##
                # Basic Settings
                ##
        
                sendfile on;
                tcp_nopush on;
                tcp_nodelay on;
                keepalive_timeout 65;
                types_hash_max_size 2048;
                # server_tokens off;
        
                # server_names_hash_bucket_size 64;
                # server_name_in_redirect off;
        
                default_type application/octet-stream;
        
                ##
                # SSL Settings
                ##
        
                ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
                ssl_prefer_server_ciphers on;
        
                ##
                # Logging Settings
                ##
        
                access_log /var/log/nginx/access.log;
                error_log /var/log/nginx/error.log;
        
                ##
                # Gzip Settings
                ##
                gzip on;
        
                # gzip_vary on;
                # gzip_proxied any;
                # gzip_comp_level 6;
                # gzip_buffers 16 8k;
                # gzip_http_version 1.1;
                # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
        
                ##
                # Virtual Host Configs
                ##
        
                # include /etc/nginx/conf.d/*.conf;
                include /etc/nginx/sites-enabled/*;
        }
        
        ```
        
    2. site-available
        
        建立 docker-nginx-django.conf
        
        ```python
        # the upstream component nginx needs to connect to
        upstream uwsgi {
            # server web:8001; # use TCP
            server unix:/opt/app/src/app.sock; # for a file socket
        }
        
        # configuration of the server
        server {
            # the port your site will be served on
            listen    80;
            # index  index.html;
            # the domain name it will serve for
            # substitute your machine's IP address or FQDN
            server_name  192.168.1.1;
            charset     utf-8;
        
            client_max_body_size 75M;   # adjust to taste
        
            # Django media
            # location /media  {
            #     alias /docker_api/static/media;  # your Django project's media files - amend as required
            # }
        
            location /static {
                alias /opt/app/src/static; # your Django project's static files - amend as required
            }
        
            location / {
                uwsgi_pass  uwsgi;
                include     /etc/nginx/uwsgi_params; # the uwsgi_params file you installed
            }
        
        }
        ```
        
        建立軟連結
        
        ```python
        cd ../sites-enabled
        ln -s ../sites-available/docker-nginx-django.conf
        ```
        
    3. Dockerfile
        
        ```python
        FROM nginx:latest
        
        COPY nginx.conf /etc/nginx/nginx.conf
        
        CMD ["nginx", "-g", "daemon off;"]
        ```
        

References:

[https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)

[https://orcahmlee.github.io/devops/docker-nginx-uwsgi-django-root/](https://orcahmlee.github.io/devops/docker-nginx-uwsgi-django-root/)

[https://github.com/twtrubiks/docker-django-nginx-uwsgi-postgres-tutorial/blob/master/README.en.md](https://github.com/twtrubiks/docker-django-nginx-uwsgi-postgres-tutorial/blob/master/README.en.md)