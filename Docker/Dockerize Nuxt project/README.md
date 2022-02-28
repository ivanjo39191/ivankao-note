# Dockerize Nuxt project

Created: February 1, 2022 5:22 PM
Property: YuWei Kao
Tags: Docker, Vue
blog: Done

## 前言、

本篇介紹如何快速的將 NUXT 專案使用 docker 進行容器化。

## 一、建立  Dockerfile

EXPOSE 對外 port 設置為 9527

```docker
FROM node:10.19.0
MAINTAINER ivankao
ENV NODE_ENV=production
ENV HOST 0.0.0.0
RUN mkdir -p /app
COPY . /app
WORKDIR /app
EXPOSE 9527
RUN npm install
RUN npm run build
CMD ["npm", "start"]
```

## 二、建立 .dockerignore

以下檔案不進行搬移

```docker
node_modules
.nuxt
```

## 三、建立容器

-t 為容器名稱與標籤 project:1.0 

最後的 . 為當前目錄

```docker
docker build -t project:1.0 .
```

## 四、例外處理

1. Error: Cannot find module '@nuxtjs/eslint-module'
    
    原因為配置為 ENV NODE_ENV=production 時，
    
    package.json 中的 devDependencies 將被跳過，
    
    將會使用到的 devDependencies 項目移至 dependencies 即可
    

## 五、運行容器

```docker
docker run -dt -p 9527:9527 project:1.0
```

## 六、docker 指令

```bash

docker ps -a              # 查看當前 containers
docker stop CONTAINERID   # 停止 container
docker rm CONTAINERID     # 移除 container
docker images             # 查看當前 images
docker image rm IMAGEID   # 移除 image
```

## 七、參考資料

[https://www.gushiciku.cn/pl/pRNY/zh-tw](https://www.gushiciku.cn/pl/pRNY/zh-tw)

[https://segmentfault.com/a/1190000010396645](https://segmentfault.com/a/1190000010396645)

[https://github.com/nuxt-community/eslint-module/issues/37](https://github.com/nuxt-community/eslint-module/issues/37)

[https://medium.com/@ryanC1993/dockerize-nuxt-project-c3ab8d23cd70](https://medium.com/@ryanC1993/dockerize-nuxt-project-c3ab8d23cd70)

[https://www.matthewburfield.com/do-npm-dependencies-and-dev-dependecies-effect-your-production-build/](https://www.matthewburfield.com/do-npm-dependencies-and-dev-dependecies-effect-your-production-build/)

[https://github.com/nuxt-community/eslint-module/issues/37](https://github.com/nuxt-community/eslint-module/issues/37)