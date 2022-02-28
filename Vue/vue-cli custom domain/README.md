# vue-cli custom domain

Created: February 1, 2022 5:22 PM
Tags: Vue

自訂 Domain

在根目錄創建 vue.config.js

添加以下內容:

```jsx
module.exports = {
    devServer: {
        disableHostCheck: true,
    }
}
```