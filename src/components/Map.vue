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
import axios from 'axios'
import L from 'leaflet'
const counties = require('@/assets/counties.json')
export default {
  data () {
    return {
      employmentTotals: []
    }
  },

  methods: {
    getColor (val, employmentTotals) {
      const max = Math.max(...employmentTotals)
      return val > 0.8 * max ? '#0868ac'
        : val > 0.6 * max ? '#43a2ca'
          : val > 0.4 * max ? '#7bccc4'
            : val > 0.2 * max ? '#bae4bc'
              : '#f0f9e8'
    },
    getTotalEmployment (array, val) {
      return array.find(x => x[10] === val)
    }
  },

  async mounted () {
    try {
      const res = await axios.get('https://api.census.gov/data/timeseries/qwi/sa', {
        params: {
          get: 'Emp',
          for: 'county:*',
          in: `state:06`,
          year: '2017',
          quarter: '3',
          sex: '0',
          agegrp: 'A01',
          ownercode: 'A05',
          firmsize: '0',
          seasonadj: 'U',
          industry: '11',
          key: 'f0c0e4dc0941d95473e9ce88d697d682b1d7fca4'
        }
      })
      // Store just the employment totals into a new array so that we can compute max in getColor (above)
      this.employmentTotals = res.data.map(element => element[0]).filter(num => num > 0)
      for (let i in counties.features) {
        if (counties.features[i].properties.STATE === '06' && this.getTotalEmployment(res.data, counties.features[i].properties.COUNTY) !== undefined) {
          counties.features[i].properties.totalEmployment = this.getTotalEmployment(res.data, counties.features[i].properties.COUNTY)[0]
        }
      }
    } catch (err) {
      console.log(err)
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
      style: (feature) => {
        if (feature.properties.totalEmployment && feature.properties.totalEmployment !== null) {
          return {
            'color': this.getColor(feature.properties.totalEmployment, this.employmentTotals),
            'weight': 0.5,
            'opacity': 1,
            'fillOpacity': 0.5
          }
        } else {
          return {
            'color': null,
            'weight': 0.5,
            'opacity': 1,
            'fillOpacity': 0
          }
        }
      },
      onEachFeature: (feature, layer) => {
        if (feature.properties.totalEmployment && feature.properties.totalEmployment !== null) {
          layer.bindPopup(`
              <strong>County:</strong> ${feature.properties.NAME}
              <br />
              <strong>Total Employment:</strong> ${feature.properties.totalEmployment}
            `)
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
