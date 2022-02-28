# Harbor 架設

Created: February 1, 2022 5:22 PM
Tags: Docker

1. 取得最新版本

[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)

```docker
wget https://github.com/goharbor/harbor/releases/download/v2.3.2/harbor-offline-installer-v2.3.2.tgz
tar -zxvf harbor-offline-installer-v2.3.2.tgz
```

1. 配置設定檔

建立設定檔

```docker
cp -p harbor.yml.tmpl harbor.yml
vim harbor.yml

```

調整設定檔

```docker

hostname: your.domain.com

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 8080

# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: your.cer
  private_key: your.key
```

1. 運行

```docker
sudo docker-compose up -d
```

參考

[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)

[https://github.com/goharbor/harbor/issues/7008](https://github.com/goharbor/harbor/issues/7008)