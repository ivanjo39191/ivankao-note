# google-map-api  
在 vue-cli 中使用 google-map-api
## 申請 google地圖api

[開始使用](https://cloud.google.com/maps-platform/pricing?hl=zh-tw)  

[參考步驟](https://medium.com/%E9%BA%A5%E5%85%8B%E7%9A%84%E5%8D%8A%E8%B7%AF%E5%87%BA%E5%AE%B6%E7%AD%86%E8%A8%98/%E7%AD%86%E8%A8%98-%E5%BE%9E%E9%9B%B6%E6%8E%A5%E8%A7%B8-google-map-api-%E5%9C%A8-vue-js-%E4%B8%AD%E5%AF%A6%E4%BD%9C%E5%9C%B0%E5%9C%96-%E5%9C%B0%E6%A8%99-%E8%A8%8A%E6%81%AF%E8%A6%96%E7%AA%97-8eed860637d6)

## vue 初次建立地圖

[參考範例](https://vuejs.org/v2/cookbook/practical-use-of-scoped-slots.html)
1. 建立 GoogleMapLoader.vue 來串接地圖  
    設置 class "google-map"  
    設置 ref "googleMap"
```html
    <template>
      <div>
        <div class="google-map" ref="googleMap"></div>
      </div>
    </template>

```
__**1. Prop 是你可以在組件上註冊的一些自定義 attribute。當一個值傳遞給一個 prop attribute 的時候，它就變成了那個組件實例的一個 property。**__  
>在這裡我們設置兩個 prop  
>mapConfig 接收傳進來的地圖資料  
>apiKey　接收GoogleMapApi 取得的 apiKey  

__**2. \$ref 用以訪問子組件實例或元素，儘管存在 prop 和事件，有的時候你仍可能需要在 JavaScript 裡直接訪問一個子組件。為了達到這個目的，你可以通過 ref 這個 attribute 為子組件賦予一個 ID**__  
>這裡的 this.$refs.googleMap 訪問了 template 中ref名為 googleMap 的組件  
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
 



