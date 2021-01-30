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