# 建立 Docker Registry

Created: February 1, 2022 5:22 PM
Property: ivan kao
Tags: Docker
blog: Done

## 前言、

建立了許多 docker image 後，除了推送到 docker hub 之外，

也可以建立自己的 Docker Registry，

以下將介紹如何架設。

## 一、 建立倉庫存放目錄

```docker
cd /opt
mkdir docker-registry
cd docker-registry
```

## 二、建立憑證

透過安全的連線來進行 docker registry 的連線，

不建議用關閉安全性 insecure 的方式跳過。

1.  使用 acme.sh 來自動取得憑證，這裡指定使用 80 port 

```bash
docker run --rm  -itd  \
  -v "$(pwd)/out":/acme.sh  \
  -p 80:80 \
  --name=acme.sh \
  neilpang/acme.sh daemon
```

1.  自動更新憑證

```bash

# 設定驗證網域(由acme產生臨時網頁伺服器進行認證)
docker  exec  acme.sh --issue -d your.server.com  --standalone

# 開啟微服務檔案自動更新
docker  exec  acme.sh --upgrade --auto-upgrade

# 未註冊 可添加參數
--register-account -m name@mail.com
```

## 三、使用 docker-auth token

添加 docker login 驗證機制。

但因要驗證可能會造成 web UI 的額外套件無法正常使用。

帳號密碼使用 htpasswd 建立

```docker
htpasswd -nB USERNAME
```

建立 auth_config.yml

```docker
cd /opt/docker-registry
mkdir config
cd config
touch auth_config.yml
```

5001 是 docker-auth 進行驗證時使用的 port

acl 進行權限控制

```bash
# A simple example. See reference.yml for explanation for explanation of all options.
#
#  auth:
#    token:
#      realm: "https://127.0.0.1:5001/auth"
#      service: "Docker registry"
#      issuer: "Acme auth server"
#      rootcertbundle: "/path/to/server.pem"

server:
  addr: ":5001"
  certificate: "/certs/fullchain.cer"
  key: "/certs/your.server.com.key"

token:
  issuer: "Acme auth server"  # Must match issuer in the Registry config.
  expiration: 900

users:
  # Password is specified as a BCrypt hash. Use `htpasswd -nB USERNAME` to generate.
  "admin":
    password: "$2y$05$LO.vzwpWC5LZGqThvEfznu8qhb5SGqvBSWY1J3yZ4AxtMRZ3kN5jC"  # badmin
  "test":
    password: "$2y$05$WuwBasGDAgr.QCbGIjKJaep4dhxeai9gNZdmBnQXqpKly57oNutya"  # 123

acl:
  - match: {account: "admin"}
    actions: ["*"]
    comment: "Admin has full access to everything."
  - match: {account: "test"}
    actions: ["pull"]
    comment: "User \"test\" can pull stuff."
  # Access is denied by default.
```

以 5001 port 運行

```bash
docker run \
    --rm -itd --name docker_auth -p 5001:5001\
    -v /opt/docker-registry/config:/config:ro \
    -v /opt/docker-registry/docker_auth:/logs \
    -v /opt/docker-registry/out/your.server.com:/certs \
    cesanta/docker_auth:1 \
    --v=2 --alsologtostderr \
    /config/auth_config.yml
```

## 四、建立 docker registry

使用建立的憑證、透過 5001 port 進行 docker-auth 驗證

運行在 5443 port

```bash
docker run -d \
 --restart=always \
 --name registry \
 -e "REGISTRY_AUTH=token" \
 -e "REGISTRY_AUTH_TOKEN_REALM=https://your.server.com:5001/auth" \
 -e "REGISTRY_AUTH_TOKEN_SERVICE=Docker registry" \
 -e "REGISTRY_AUTH_TOKEN_ISSUER=Acme auth server" \
 -e "REGISTRY_AUTH_TOKEN_ROOTCERTBUNDLE=/certs/fullchain.cer" \
 -e "REGISTRY_AUTH_TOKEN_AUTOREDIRECT=false" \
 -e "REGISTRY_STORAGE_DELETE_ENABLED=true" \
 -v /opt/docker-registry/out/your.server.com:/certs \
 -e REGISTRY_HTTP_ADDR=0.0.0.0:5443 \
 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/fullchain.cer \
 -e REGISTRY_HTTP_TLS_KEY=/certs/your.server.com.key \
 -e REGISTRY_STORAGE_DELETE_ENABLED=true
 -p 5443:5443 \
 -v /opt/docker-registry/registry:/var/lib/registry \
 registry:2
```

至此就架設完成了。

## 五、測試

1.  登入

```bash
docker login your.server.com:5443

Username: username
Password: 

Login Succeeded
```

1. 測試推送

```bash
docker login your.server.com:5443
docker tag project:1.0.0 your.server.com:5443/project:1.0.0
docker push your.server.com:5443/project:1.0.0

```

1.  測試拉取

```bash
docker login your.server.com:5443
docker push your.server.com:5443/project:1.0.0
```

## 六、介面

使用 docker-registrt-web

1.  建立設定檔

```bash
cd /opt/docker-registry
mkdir conf
cd conf
touch registry-web.yml
```

1. registry-web.yml

這邊要注意 name 與 issuer 要與 docker registry run 時設置的一致

```bash
registry:
  # Docker registry url
  url: https://your.server.com:5443/v2
  # Docker registry fqdn
  name: Docker registry
  # To allow image delete, should be false
  readonly: false
  auth:
    # Enable authentication
    enabled: true
    # Token issuer
    # should equals to auth.token.issuer of docker registry
    issuer: 'Acme auth server'
    # Private key for token signing
    # certificate used on auth.token.rootcertbundle should signed by this key
    key: /conf/auth.key
```

1.  運行 docker-registry-web

```bash
docker run -v /opt/docker-registry/conf/registry-web.yml:/conf/config.yml:ro \
           -v /opt/docker-registry/out/your.server.com/your.server.com.key:/conf/auth.key \
           -v /opt/docker-registry/data:/data \
           -itd -p 8080:8080 --link docker_auth --name registry-web hyper/docker-registry-web
```

## 七、參考資料

[https://hackmd.io/@neverleave0916/S1KhWswhv](https://hackmd.io/@neverleave0916/S1KhWswhv)

[https://stackoverflow.com/questions/58646089/could-not-find-solver-for-http-01-when-set-up-docker-registry-with-lets-encrypt](https://stackoverflow.com/questions/58646089/could-not-find-solver-for-http-01-when-set-up-docker-registry-with-lets-encrypt)

[https://zhuanlan.zhihu.com/p/45425683](https://zhuanlan.zhihu.com/p/45425683)

[https://github.com/acmesh-official/acme.sh/wiki/Run-acme.sh-in-docker](https://github.com/acmesh-official/acme.sh/wiki/Run-acme.sh-in-docker)

[https://github.com/acmesh-official/acme.sh/wiki/deploy-to-docker-containers](https://github.com/acmesh-official/acme.sh/wiki/deploy-to-docker-containers)

[https://github.com/distribution/distribution/blob/main/docs/spec/auth/token.md](https://github.com/distribution/distribution/blob/main/docs/spec/auth/token.md)

[https://github.com/cesanta/docker_auth/issues/144](https://github.com/cesanta/docker_auth/issues/144)

[https://github.com/kwk/docker-registry-frontend](https://github.com/kwk/docker-registry-frontend)

[https://hub.docker.com/r/hyper/docker-registry-web/](https://hub.docker.com/r/hyper/docker-registry-web/)

[https://github.com/mkuchin/docker-registry-web/issues/64](https://github.com/mkuchin/docker-registry-web/issues/64)