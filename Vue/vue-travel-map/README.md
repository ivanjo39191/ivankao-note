# vue-travel-map

## 一、環境建置

### 1. 安裝環境:  
```
npm install -g @vue/cli
```

### 2. 創建專案:  
```
vue create vue-travel-map
```
>選擇 vue2.x 版本

### 3. 安裝套件

```
npm install --save google-maps-api-loader
vue add router
vue add axios
npm audit fix

```

### 4. 創建 .env 設置環境變量
在根目錄創建 .env 檔案，並將 googlemap 的 apikey 寫在其中  
```
 VUE_APP_GOOGLEMAPAPIKEY="yourApiKey"
```

### 5. 當前目錄結構  
```
.
├── babel.config.js
├── .env
├── .git
├── .gitignore
├── node_modules
├── package.json
├── package-lock.json
├── public
├── README.md
└── src
    ├── App.vue
    ├── assets
    ├── components
    ├── main.js
    ├── router
    └── views
```

### 運行專案
```
npm run serve
```

![Welcome to Vue](./github_image/1.png)

## 二、初次建立地圖

### 1. 在 src/components 建立 GoogleMapLoader.vue 地圖元件

```html
<template>
  <div>
    <div class="google-map" ref="googleMap"></div>
    <template v-if="Boolean(this.google) && Boolean(this.map)">
      <slot
        :google="google"
        :map="map"
      />
    </template>
  </div>
</template>
```

```js
<script>
import GoogleMapsApiLoader from 'google-maps-api-loader'

export default {
  props: {
    mapConfig: Object,
    apiKey: String,
  },
  data() {
    return {
        google: null,
        map: null,
    }
  },
  async mounted() {
    const googleMapApi = await GoogleMapsApiLoader({
      apiKey: this.apiKey
    })
    this.google = googleMapApi
    this.initializeMap()
  },

  methods: {
    initializeMap() {
      const mapContainer = this.$refs.googleMap
      this.map = new this.google.maps.Map(mapContainer, this.mapConfig)
    }
  }
}

</script>
```

### 2. 在 src/views 創建 TravelMap.vue 顯示地圖頁面

```html
<template>
  <div class="travel-map">
    <div>
      <h2>TravelMap</h2>
      <hr>
      <div>
        <div>
          <GoogleMapLoader ref="googlemaploader"
            :mapConfig="mapConfig"
            :apiKey="apiKey"
          >
          </GoogleMapLoader>
        </div>
      </div>
    </div>
  </div>  
</template>
```

```js
<script>
// @ is an alias to /src
import GoogleMapLoader from '@/components/GoogleMapLoader'

export default {
  components: {
    GoogleMapLoader,
  },
  data(){
    return {
      apiKey : process.env.VUE_APP_GOOGLEMAPAPIKEY,
    }
  },
  computed: {
    mapConfig () {
      return {
        zoom: 8,
        center: this.mapCenter
      }
    },
    mapCenter () {
          return {lng: 120.93891, lat: 23.60278}
    }
  },
}
</script>

```
> apiKey 會調用 .env 檔的 VUE_APP_GOOGLEMAPAPIKEY 設定值
```css
<style>
.google-map {
  width: 100vw;
  height: 75vh;
}
</style>
```
> 須設定 style 才會正常顯示地圖  

### 3. 在 src/router/index.js 添加 travelmap 路由設定  

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  },
  // 添加 travelmap
  {
    path: '/travelmap',
    name: 'travelmap',
    component: () => import(/* webpackChunkName: "gotravel" */ '../views/TravelMap.vue')
  },
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router

```

### 4. 查看當前目錄結構與 TravelMap 頁面  

查看當前結構  

```
.
├── babel.config.js
├── .env
├── .env.local
├── .git
├── github_image
├── .gitignore
├── node_modules
├── package.json
├── package-lock.json
├── public
├── README.md
└── src
    ├── App.vue
    ├── assets
    │   └── logo.png
    ├── components
    │   ├── GoogleMapLoader.vue
    │   └── HelloWorld.vue
    ├── main.js
    ├── router
    │   └── index.js
    └── views
        ├── About.vue
        ├── Home.vue
        └── TravelMap.vue
```

瀏覽器前往 localhost:8080/travelmap 頁面  

![Welcome to Vue](./github_image/2.png)

## 三、透過 api 資料生成標誌  

### 1. 建立 GoogleMapMarker 元件

```js
<script>

export default {
  props: {
    google: {
      type: Object,
    },
    map: {
      type: Object,
    }
  },

  methods:{
    makemarker(data, is_first) {
      const marker = new this.google.maps.Marker({
        position: data.position,
        map: this.map,
        icon: {
              url: 'http://maps.google.com/mapfiles/ms/micons/cabs.png',
        },
      });
      if (is_first){
        this.map.setZoom(12);
        this.map.panTo(data.position);
      }
      marker.addListener('mouseover',function(){
        this.map.panTo(data.position);
      });
    }
  }
}
</script>

```
> props 取得 google, map 值  
> methods 創建 makemarker 方法並傳入 data (標誌陣列)與 is_first (是否為第一行陣列)  
> new this.google.maps.Marker 創建標誌，傳入經緯度、圖標  
> is_first 時 setZoom 設定地圖比例，panTo 移動到位置  
> addListener mouseover，當游標移動到標誌時，panTo 到標誌位置  

### 2. TravelMap.vue 調用 GoogleMapMarker 創建標誌

template 區塊  
```html
<template>
  <div>
    <button @click="searchmarker" class="btn" type="button">顯示地標</button>
    <div class="google-map" ref="googleMap"></div>
    <template v-if="Boolean(this.google) && Boolean(this.map)">
      <slot
        :google="google"
        :map="map"
      />
    </template>
    <GoogleMapMarker ref="googlemapmarker"
      :google="google"
      :map="map"
    />
  </div>
</template>
```
> 添加 button @click="searchmarker" 點選按鈕時觸發 searchmarker 方法  
> 調用 GoogleMapMarker 並添加 ref 屬性，目的是讓 methods 直接調用 GoogleMapMarker 中定義的 makemarker 方法  

script 區塊  
```js
<script>
import GoogleMapsApiLoader from 'google-maps-api-loader'
import GoogleMapMarker from '@/components/GoogleMapMarker'

export default {
  components: {
    GoogleMapMarker,
  },
  props: {
    mapConfig: Object,
    apiKey: String,
  },
  data() {
    return {
        google: null,
        map: null,
    }
  },
  async mounted() {
    const googleMapApi = await GoogleMapsApiLoader({
      apiKey: this.apiKey
    })
    this.google = googleMapApi
    this.initializeMap()
  },

  methods: {
    initializeMap() {
      const mapContainer = this.$refs.googleMap
      this.map = new this.google.maps.Map(mapContainer, this.mapConfig)
    },
    searchmarker: function() {
      const cors = 'https://cors-anywhere.herokuapp.com/';
      const url = "http://gis.taiwan.net.tw/XMLReleaseALL_public/scenic_spot_C_f.json";
      this.$axios.get(cors+url)
      .then(res => {
          for (const [index, value] of res.data['XML_Head']['Infos']['Info'].slice(1500,1600).entries()){
            let marker = [];
            let is_first = false;
            if (index == 0){is_first = true;}
            marker = { 
              id: value['Id'], 
              position:{ lng: parseFloat(value['Px']), lat: parseFloat(value['Py'])}, 
            };
            this.$refs.googlemapmarker.makemarker(marker, is_first);
          }
      })
      .catch(err => console.log(err));
    }
  }
}

</script>
```
> 引用 GoogleMapMarker 元件  
> methods 創建 searchmarker 方法  
> 使用 cors-anywhere 進行跨來源資源共用  
> 使用 axios 異步請求政府開放資料庫的 json 資料  
> for... of ... .entries() 取得迴圈索引，用以進行 is_first(是否為第一行陣列) 判斷  
> 取其中 100 筆資料的 Id 與經緯度，傳入 GoogleMapMarker 中的 makemarker 方法  





### 3. 創建標誌功能展示

![Welcome to Vue](./github_image/3.gif)