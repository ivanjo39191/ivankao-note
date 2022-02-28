# Nuxt media query root font-size

Created: February 1, 2022 5:22 PM
Tags: Vue

## 前言、

本篇記錄在 Nuxt 下如何使用 media query 調整 root font-size 達到響應式的需求

## 一、建立 scss 檔案

在 asset 新增一設定檔 main.scss

## 二、main.scss 配置 media query font-size

以下配置供參考

```scss
@media (min-width: 750px) {
  :root {
    font-size: 8px;
  }

  html,
  body {
    font-size: 1.6rem;
  }
}

@media (min-width: 1366px) and (max-width: 1500px) {
  :root {
    font-size: 11px;
  }

  html,
  body {
    font-size: 1.6rem;
  }
}

@media (min-width: 1500px) and (max-width: 1920px) {
  :root {
    font-size: 14px;
  }

  html,
  body {
    font-size: 1.6rem;
  }
}

@media (min-width: 1920px) {
  :root {
    font-size: 16px;
  }

  html,
  body {
    font-size: 1.6rem;
  }
}
```

## 三、調整 nuxt.config.js 設定檔

在  nuxt.config.js 添加以下 css 設定:

```scss
// Global CSS: https://go.nuxtjs.dev/config-css
  css: [
    '@/assets/main.scss',
  ],
```

## 參考資料、

[https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-css](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-css)