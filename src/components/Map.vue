<template lang="html">
  <v-container grid-list-xs>
    <v-layout row wrap>
      <v-flex xs12>
        <div id="map"></div>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import L from 'leaflet'
const counties = require('@/assets/counties.json')
export default {
  data () {
    return {

    }
  },

  mounted () {
    const getColor = (val) => {
      return val > 0.8 ? '#f0f9e8'
        : val > 0.6 ? '#bae4bc'
          : val > 0.4 ? '#7bccc4'
            : val > 0.2 ? '#43a2ca'
              : '#0868ac'
    }
    for (let i in counties.features) {
      counties.features[i].properties.concentration = Math.random()
    }
    const map = L.map('map', { zoomControl: true })
      .locate({ setView: true, maxZoom: 6 })
    L.tileLayer('https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}{r}.png', {
      attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> &copy; <a href="http://cartodb.com/attributions">CartoDB</a>',
      subdomains: 'abcd',
      maxZoom: 16,
      minZoom: 4
    }).addTo(map)
    L.geoJSON(counties, {
      style (feature) {
        return {
          'color': getColor(feature.properties.concentration),
          'weight': 0.1,
          'opacity': 1,
          'fillOpacity': 0.5
        }
      }
    }).addTo(map)
  }
}
</script>

<style scoped lang="css">
#map {
  height: 600px;
}
</style>
