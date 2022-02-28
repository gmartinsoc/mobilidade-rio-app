
<template>
  <l-map id="map" style="height: 300px" :zoom="zoom" :center="center" v-if="code.length === 4" >
    <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
    <l-marker v-for="(item, index) in markerLatLng"
                    :key="'marker-' + index"   
                    :lat-lng="markerLatLng[index]">   

      </l-marker>
  </l-map>
</template>

<script>
import { mapState } from "vuex";


import {LMap, LTileLayer, LMarker} from 'vue2-leaflet';
import { Icon } from 'leaflet';


delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});


export default {
  components: {
    LMap,
    LTileLayer,
    LMarker
  },
  data () {
    return {
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 15,
      center: [-22.9112007,-43.2056211],
      markerLatLng: [[-22.9112007,-43.2056211]],
      customText: "Foobar"

    };
  },
  computed: mapState({
    code: (state) => state.code,
    address:  (state) => state.address,
    Coords: (state) => state.Coords,
    trip: (state) => state.trip,
    stops: (state) => state.stops
  }),
  watch: {
        code (value) {
          this.$emit('onchange', value);
          this.getstopCoords(value)
        },  
        trip (trip) {
          this.$emit('onchange', trip);
          this.getTripCoords(trip)
        }
  },
  methods: {
   async getstopCoords(value){
      console.log(value)
        try {
        const res = await fetch("https://api.mobilidade.rio/qrcode/?code="+value)
        value= await res.json()

        this.center = [value.results[0].stop.latitude, value.results[0].stop.longitude]
        this.markerLatLng= [[value.results[0].stop.latitude, value.results[0].stop.longitude]]

      } catch (error) {
        console.log('Error! Could not reach the API. ')
      }
    },

   async getTripCoords(trip){
      //console.log(trip)
        try {
        const res = await fetch("https://api.mobilidade.rio/sequence/?trip_id="+trip)
        let stops= await res.json()
        stops=stops.results

        //console.log (stops.stop.latitude)
            for (var i in stops){
          
          this.markerLatLng.push([stops[i].stop.latitude,stops[i].stop.longitude]) 
          //console.log(this.markerLatLng)
        }
    
      } catch (error) {
        console.log('Error! Could not reach the API. ')
      }
    }
},
}


</script>


<style scoped>
.badge.even-larger-badge {
  font-size: 1.05em;
}
</style>