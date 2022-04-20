# 搭建 Drone CI+ GitLab+Harbor CI/CD環境

![drone-gitlab-harbor](./drone-gitlab-harbor.jpg)

Created: April 20, 2022 2:48 PM  
Property: Ivan Kao  
Status: 已公開  
Tags: CI/CD, Docker, Drone, Git  

## 目標說明:

push 到 GitLab 時觸發 Drone 來 Build Image ，最後推送到 Harbor 。

## 設定 GitLab 應用程式

使用 admin 帳號登入進行 Applications 設定。

開啟 Admin Area > Applications > New applications 建立新的應用程式。

**Name** 命名為 drone，

**Redirect URI** 輸入接下來 drone 服務使用的登入頁面，例如 http://example.com:8080/login，

所有勾選的項目只需勾選 **Scopes >** **api**，其餘不勾。

建立完成後取得 Application ID 與 Secret，後續會使用到。

## 搭建 Drone 服務

### No limit Drone Dockerfile

使用免費使用開源或企業版本會有 **5000** builds 次數限制，使用 nolimit 參數來解除限制。

下方預設使用的 host 為 example.com ，port 為 8080，可根據需求自行調整 EXPOSE , DRONE_SERVER_PORT, DRONE_SERVER_HOST

```jsx
FROM golang:1.16.0 AS builder

RUN apt-get update && apt-get install -y ca-certificates

RUN git clone -b v2.11.1 --depth=1 https://github.com/drone/drone
RUN cd drone && go install -trimpath -ldflags='-w -s' -tags nolimit ./cmd/drone-server

FROM debian:buster-slim

COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /go/bin/drone-server /

EXPOSE 8080
VOLUME /drone/data

ENV GODEBUG netdns=go
ENV XDG_CACHE_HOME /data
ENV DRONE_DATABASE_DRIVER sqlite3
ENV DRONE_DATABASE_DATASOURCE /data/database.sqlite
ENV DRONE_RUNNER_OS=linux
ENV DRONE_RUNNER_ARCH=amd64
ENV DRONE_SERVER_PORT=:8080
ENV DRONE_SERVER_HOST=example.com

ENTRYPOINT ["/drone-server"]

# https://github.com/harness/drone/releases
```

### 建立 Drone Image

```jsx
docker build -t drone_no_limit .
```

### 撰寫 Drone docker-compose

.env

```yaml
DRONE_SERVER_HOST=example.com
DRONE_SERVER_PORT=:8080
DRONE_SERVER_PROTO=http
DRONE_RPC_SECRET=bea26a2221fd8090ea38720fc445eca6
DRONE_GITLAB_SERVER=http://gitlab.example.com
# 在 GitLab 應用頁面查看
DRONE_GITLAB_CLIENT_ID=YOUR_GITLAB_CLIENT_ID
# 在 GitLab 應用頁面查看
DRONE_GITLAB_CLIENT_SECRET=YOUR_GITLAB_CLIENT_SECERT
```

docker-compose.yml

```yaml
version: "3.8"
services:
    drone:
        image: drone_no_limit 
        ports:
            - "8080:8080"
        environment:
            - DRONE_SERVER_HOST=${DRONE_SERVER_HOST}${DRONE_SERVER_PORT} # Drone URL
            - DRONE_SERVER_PROTO=${DRONE_SERVER_PROTO}                   # http 或者 https 連線設定
            - DRONE_RPC_SECRET=${DRONE_RPC_SECRET:-secret}
            - DRONE_GITLAB_SERVER=${DRONE_GITLAB_SERVER}
            - DRONE_GITLAB_CLIENT_ID=${DRONE_GITLAB_CLIENT_ID}           # taken from your Gitlab oauth application
            - DRONE_GITLAB_CLIENT_SECRET=${DRONE_GITLAB_CLIENT_SECRET}   # taken from your Gitlab oauth application
            - DRONE_LOGS_DEBUG=true                                      # 選擇是否開啟 debug 模式
            - DRONE_CRON_DISABLED=true
        volumes:
            - ./data:/data
        networks:
            - drone-network
    runner:
        image: drone/drone-runner-docker:latest
        environment:
            - DRONE_RPC_HOST=drone${DRONE_SERVER_PORT}
            - DRONE_RPC_PROTO=${DRONE_SERVER_PROTO} 
            - DRONE_RPC_SECRET=${DRONE_RPC_SECRET:-secret}
            - DRONE_TMATE_ENABLED=true
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
        depends_on:
            - drone
        networks:
            - drone-network
networks:
  drone-network:
    driver: bridge
```

### 運行 Drone

```yaml
docker-compose up -d
```

## 將 repo build 後推送至 Harbor

需在 repo 底下有一個可執行的 Dockerfile，就可以將這個 Dockerfile build 好後放入 Harbor。

### 在 Drone 設定 Harbor 帳號密碼

推送到 Harbor 會須經過帳號密碼驗證，因此要在Drone 中新增帳號密碼到 repo 的 secret 設定中，

開啟 Drone 介面 > REPOSITORIES > example-repo > Settings > Secrets > New Secret，

新增 harbor_username 與 harbor_password。

### 撰寫 .drone.yml

```yaml
kind: pipeline
type: docker      # 在 Docker 內部執行管道命令
name: clone       # 可自行定義的名稱

steps:
  - name: build-image-push-harbor                           # 事件：可自行定義的名稱
    image: plugins/docker                                   # 使用 plugins/docker  容器
    settings:
      username:                                             # harbor 私有庫帳號
        from_secret: harbor_username
      password:                                             # harbor 私有庫密碼
        from_secret: harbor_password
      insecure: true
      repo: example.harbor.com/example_repo/example_repo    # harbor 私有庫存放位置
      tags: latest                                          # harbor image tag( 也可以利用 ${DRONE_TAG} 自動取得 push 到 gitlab 的版號)
      registry: example.harbor.com                          # harbor 私有庫網址
```

## 完成

設定到此結束，現在只要 git push 到 GitLab，就會觸發 Drone 進行 Build Image 再推送到 Harbor 囉!

### 參考資料

[https://docs.drone.io/](https://docs.drone.io/)

[https://blog.wu-boy.com/2021/08/drone-license/](https://blog.wu-boy.com/2021/08/drone-license/)

[https://github.com/harness/drone/blob/master/docker/compose/drone-github/docker-compose.yml](https://github.com/harness/drone/blob/master/docker/compose/drone-github/docker-compose.yml)
[https://yeasy.gitbook.io/docker_practice/ci/drone/install](https://yeasy.gitbook.io/docker_practice/ci/drone/install)
[https://blog.csdn.net/weixin_43066697/article/details/116931442](https://blog.csdn.net/weixin_43066697/article/details/116931442)

[https://ithelp.ithome.com.tw/articles/10223277](https://ithelp.ithome.com.tw/articles/10223277)