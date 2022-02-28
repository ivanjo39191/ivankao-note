# Vue mount vuex promise getter

Created: February 1, 2022 5:22 PM
Tags: Vue

要完成的情境為；

開啟頁面後 mount  呼叫 vuex 的 actions 來取得 api 資料並顯示

步驟如下：

1. 取得 blog 文章 API method api/blog.js
    1. getArticle
2. vuex 的 store/blog.js
    1. const state
    2. const mutations
    3. const actions
    4. const getters
3. 要觸發 mount 的頁面 pages/blog/Articles.vue
    1. data
    2. methods
    3. mounted

一、取得 blog 文章 API method api/blog.js

1. getArticle

本篇主要不是敘述 API 取資料的部分，故簡單帶過

```jsx
export function getArticle() {
  return request({
    url: process.env.VUE_APP_BACKEND_SERVER + '/api/article/',
    method: 'get'
  })
}
```

 二、vuex 的 store/blog.js

1. const state
    
    定義 latest
    
    ```jsx
    export const state = () => ({
      latest: []
    })
    ```
    
2. const mutations
    
    定義 setLatest 
    
    ```jsx
    export const mutations = {
      setLatest (state, latest) {
        state.latest = latest
      }
    }
    ```
    
3.  const actions
    
    import api
    
    並將 response commit 到 mutations
    
    ```jsx
    import { getArticle } from '@/api/blog'
    
    export const actions = {
      switchLatest ({ commit }, type) {
        return new Promise((resolve, reject) => {
          getArticle (type)
            .then((response) => {
              commit('setLatest', response)
              resolve()
            })
            .catch((error) => {
              reject(error)
            })
        })
      }
    }
    ```
    
4. const getters
    
    定義 getLatest  給 Vue 呼叫用
    
    ```jsx
    export const getters = {
      getLatest (state) {
        return state.latest
      }
    }
    ```
    

三、要觸發 mount 的頁面 pages/blog/Articles.vue

1. data
    
    定義 articleItems
    
    ```jsx
    data () {
        return {
          articleItems: []
        }
      },
    ```
    
2. methods
    
    使用 this.$store.dispatch 呼叫 actions switchLatest
    
    使用 then 在取得後把資料放入 articleItems
    
    ```jsx
    methods: {
        getBlog () {
          this.$store.dispatch('blog/switchLatest', '').then(() => {
            this.articleItems = this.$store.getters['blog/getLatest']
          })
        }
      }
    ```
    
3. mounted
    
    載入頁面時呼叫 methods getBlog 
    
    ```jsx
    mounted () {
        this.getBlog()
      },
    ```