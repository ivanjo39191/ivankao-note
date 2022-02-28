# Docker with MariaDB

Created: February 1, 2022 5:22 PM
Tags: Docker, Mysql
blog: Done

1. Downloading an MariaDB image
    
    ```docker
    docker search mariadb
    docker pull mariadb:10.4
    ```
    
2. Creating a Container with network
    
    ```docker
    docker network create -d bridge mynetwork
    docker run --name mariadb -e MYSQL_ROOT_PASSWORD=mypass -p 3306:3306 --network mynetwork -d docker.io/library/mariadb:10.4
    ```
    
3. Container DB dump
    
    dump
    
    ```docker
    docker exec <containerid> /usr/bin/mysqldump -B <schema-name> --routines -u root --password=pass.123 <schema-name> > yoursqlname.sql
    ```
    
    back
    
    ```docker
    cat yoursqlname.sql | docker exec -i <containerid> /usr/bin/mysql -u root --password=pass.123 test
    ```
    
4. Connect to your container using a local mysql shell client
    
    ```docker
    mysql -P 3306 --protocol=tcp -u root -p
    ```
    

## 參考

[https://mariadb.com/kb/en/installing-and-using-mariadb-via-docker/](https://mariadb.com/kb/en/installing-and-using-mariadb-via-docker/)

[https://blackie1019.github.io/2018/11/13/MariaDB-MySQL-dump-SQL-for-Docker-Container/](https://blackie1019.github.io/2018/11/13/MariaDB-MySQL-dump-SQL-for-Docker-Container/)