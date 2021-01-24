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

<style>
.google-map {
  width: 100vw;
  height: 75vh;
}
</style>