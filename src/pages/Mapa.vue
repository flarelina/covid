<template>

</template>

<script>
  import L from 'leaflet';
  import { LMap, LTileLayer, LMarker } from 'vue2-leaflet';
  import 'leaflet/dist/leaflet.css';

  export default {
    name: "Mapa",
    components: [
      LMarker,
      LMap,
      LTileLayer
    ],
    mounted() {
      var geojsonFeature = {
        "type": "Feature",
        "properties": {
          "name": "Coors Field",
          "amenity": "Baseball Stadium",
          "popupContent": "This is where the Rockies play!"
        },
        "geometry": {
          "type": "Point",
          "coordinates": [-104.99404, 39.75621]
        }
      };

      var map = L.map('map').setView([39.74739, -105], 13);

/*
      L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw', {
        maxZoom: 18,
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
          '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
          'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        id: 'mapbox/light-v9',
        tileSize: 512,
        zoomOffset: -1
      }).addTo(map);
*/

      L.geoJSON(geojsonFeature).addTo(map);
    }
  }
</script>

<style scoped>

</style>
