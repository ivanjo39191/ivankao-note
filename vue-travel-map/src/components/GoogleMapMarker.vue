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
