# Build GitLab Server with Docker

![Build GitLab Server with Docker](./Build-GitLab-Server-with-Docker.jpg)

Created: April 20, 2022 2:37 PM  
Property: Ivan Kao  
Status: 已公開  
Tags: Git  

## 說明、

透過 docker 快速建立自己私有的 gitlab 版控系統

## 設定 docker-compose.yml

將 hostname 中的 example.com替換為自己的 ip 或 domain 即可

預設使用 80 443 port 

```jsx
version: "3.6"
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    hostname: example.com
    restart: always
    privileged: true
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - './gitlab/data:/var/opt/gitlab'
      - './gitlab/logs:/var/log/gitlab'
      - './gitlab/config:/etc/gitlab'
```

運行

```jsx
docker-comose up -d
```

## 更改預設密碼

預設 admin 的密碼位置如下：

```jsx
/etc/gitlab/initial_root_password
```

將會在建立後的 24 小時候自動刪除

使用此密碼登入後即可在後台自行修改

## 參考資料

[https://docs.gitlab.com/ee/install/docker.html](https://docs.gitlab.com/ee/install/docker.html)

[https://blog.csdn.net/timonium/article/details/119451755](https://blog.csdn.net/timonium/article/details/119451755)