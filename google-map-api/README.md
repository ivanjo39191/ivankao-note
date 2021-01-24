# google-map-api  
在 vue-cli 中使用 google-map-api
## 申請 google地圖api

[開始使用](https://cloud.google.com/maps-platform/pricing?hl=zh-tw)  

[參考步驟](https://medium.com/%E9%BA%A5%E5%85%8B%E7%9A%84%E5%8D%8A%E8%B7%AF%E5%87%BA%E5%AE%B6%E7%AD%86%E8%A8%98/%E7%AD%86%E8%A8%98-%E5%BE%9E%E9%9B%B6%E6%8E%A5%E8%A7%B8-google-map-api-%E5%9C%A8-vue-js-%E4%B8%AD%E5%AF%A6%E4%BD%9C%E5%9C%B0%E5%9C%96-%E5%9C%B0%E6%A8%99-%E8%A8%8A%E6%81%AF%E8%A6%96%E7%AA%97-8eed860637d6)



[參考範例](https://vuejs.org/v2/cookbook/practical-use-of-scoped-slots.html)  
## 1. 建立 GoogleMapLoader.vue 元件來串接地圖  

### (1) template 區塊  
```html
    <template>
      <div>
        <div class="google-map" ref="googleMap"></div>
      </div>
    </template>

```
template 設置 class "google-map"  
template 設置 ref "googleMap"  
### (2) script 區塊  
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
          map: null
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
          this.map = new this.google.maps.Map(
            mapContainer, this.mapConfig
          )
        }
      }
    }
    </script>
```
__**1. Prop 是你可以在組件上註冊的一些自定義 attribute。當一個值傳遞給一個 prop attribute 的時候，它就變成了那個組件實例的一個 property。**__  
>在這裡我們設置兩個 prop  
>mapConfig 接收傳進來的地圖資料  
>apiKey　接收GoogleMapApi 取得的 apiKey  

__**2. \$ref 用以訪問子組件實例或元素，儘管存在 prop 和事件，有的時候你仍可能需要在 JavaScript 裡直接訪問一個子組件。為了達到這個目的，你可以通過 ref 這個 attribute 為子組件賦予一個 ID**__  
>這裡的 this.$refs.googleMap 訪問了 template 中 ref 名為 googleMap 的組件  

__**3. async/await 用以非同步執行。async 語法所宣告的函式，被呼叫時會回傳一個 Promise。await 告訴 JS 要停下來等待這個非同步程式碼執行完畢，並且等到 Promise 被 resolve 之後才會繼續往下執行。**__ 
> 等待 GoogleMapsApiLoader 回傳後繼續執行

__**4. mounted 在模板渲染成html後調用，通常是初始化頁面完成後，再對html的dom節點進行一些需要的操作。**__  
> mounted() 從 GoogleMapApi 實例化一個 map 對象，並設置 google 的值傳遞到創建的實例**__  
## 2. 建立 TravelMap.vue 元件使用地圖元件
### (1) template 區塊  
```html
<template>
  <GoogleMapLoader
    :mapConfig="mapConfig"
    apiKey="yourApiKey"
  />
</template>
```
>取用 GoogleMapLoader 元件並傳入 mapConfig 與 apiKey
```js
<script>
import GoogleMapLoader from './GoogleMapLoader'

export default {
  components: {
    GoogleMapLoader
  },

  computed: {
    mapConfig () {
      return {
        center: { lat: 0, lng: 0 }
      }
    },
  },
}
</script>
```
>調用用 GoogleMapLoader 元件生成地圖。  

__**1. computed 為計算屬性，用以進行複雜的邏輯運算。原始資料必須變更才會觸發更新，每次調用時會把結果暫存起來，並且定要返回一個值，。**__  
>使用 computed 處理 mapConfig 並返回給  GoogleMapLoader  
>這邊回傳設置地圖中心經緯度皆為 0  

## 3. 添加 scoped slot 插槽，讓 GoogleMapLoader 返回的 google 和 map 可以再次被調用
### (1) 調整 GoogleMapLoader.vue 元件的 template 區塊
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
> 添加 slot 插槽，並在其中傳入 google 與 map 值  
### (2) 調整 TravelMap.vue 元件的 template 區塊
>添加 slot-scope 作用域插槽。  
```html
<GoogleMapLoader
  :mapConfig="mapConfig"
  apiKey="yourApiKey"
>
  <template slot-scope="{ google, map }">
  	{{ map }}
  	{{ google }}
  </template>
</GoogleMapLoader>
```
>即便 TravelMap 元件沒有定義 google 與 map，但可以透過調用 GoogleMapLoader ，將返回的 google 與 map 值直接傳入 scope-slot 中進行使用。  
## 4. 添加 GoogleMapMarker.vue 元件，用以創建標記
GoogleMapMarker.vue  
```js
<script>
export default {
  props: {
    google: {
      type: Object,
      required: true
    },
    map: {
      type: Object,
      required: true
    },
    marker: {
      type: Object,
      required: true
    }
  },

  mounted() {
    new this.google.maps.Marker({
      position: this.marker.position,
      marker: this.marker,
      map: this.map,
      icon: {
              url: 'http://maps.google.com/mapfiles/ms/micons/cabs.png',
      },
    })
  }
}
</script>
```
> 取用 google 與 map 值來進行 marker 的創建。  
## 5. 添加元素至地圖  
### (1) 調整 TravelMap.vue 元件的 template 區塊  
```html
<GoogleMapLoader
  :mapConfig="mapConfig"
  apiKey="yourApiKey"
>
  <template slot-scope="{ google, map }">
    <GoogleMapMarker
      v-for="marker in markers"
      :key="marker.id"
      :marker="marker"
      :google="google"
      :map="map"
    />
  </template>
</GoogleMapLoader>
```
>在 slot-scope 作用域插槽中調用 GoogleMapMarker ，並使用 v-for 迴圈傳入多筆標記  
### (2) 調整 TravelMap.vue 元件的 script 區塊  
```js
<script>
export default {
  components: {
    GoogleMapLoader,
    GoogleMapMarker,
  },

  data () {
    return {
      markers: [
      { id: 'a', position: { lat: 3, lng: 101 } },
      { id: 'b', position: { lat: 5, lng: 99 } },
      { id: 'c', position: { lat: 6, lng: 97 } },
      ],
    }
  },

  computed: {
    mapConfig () {
      return {
        center: this.mapCenter
      }
    },

    mapCenter () {
      return this.markers[1].position
    }
  },
}
</script>
```
>透過 data 在 computed 處理地圖資料，並設地地圖中心為第一個標記  