# NUXT.js github pages

Created: February 1, 2022 5:22 PM
Tags: Git, Vue

要將

1. 靜態化 生成 dist

```bash
npm run generate
```

1. 設定 nuxt.config.js

```json
export default {
  router: {
    base: '/<repository-name>/'
  }
}

/* nuxt.config.js */
// only add `router.base = '/<repository-name>/'` if `DEPLOY_ENV` is `GH_PAGES`
const routerBase =
  process.env.DEPLOY_ENV === 'GH_PAGES'
    ? {
        router: {
          base: '/<repository-name>/'
        }
      }
    : {}

export default {
  ...routerBase
}
```

1. 設定 package.json

```bash
/* package.json */
"scripts": {
  "generate": "nuxt generate",
  "deploy": "push-dir --dir=dist --branch=gh-pages --cleanup"
},
```

1. 進行部署

```bash
npm install push-dir --save-dev

npm run generate
npm run deploy
```