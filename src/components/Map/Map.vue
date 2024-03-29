<template>
  <div id="mapContainer"></div>
</template>

<script>
import 'leaflet/dist/leaflet.css'
import L from 'leaflet'
import 'leaflet.markercluster/dist/leaflet.markercluster-src.js'
import 'leaflet.markercluster/dist/MarkerCluster.css'
import 'leaflet.markercluster/dist/MarkerCluster.Default.css'
import { store, utils } from '@/store'
import { bus } from '@/main'

export default {
  name: 'Map',
  data () {
    return {
      mapDiv: null,
      layers: {},
      center: [47, 2.4],
      zoom: 6
    }
  },
  methods: {

    setupLeafletMap: function () {
      this.mapDiv = L.map('mapContainer', {
      }).fitBounds([[41.28, -6.5], [51.5, 10]])
      L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
        subdomains: 'abcd'
      }).addTo(this.mapDiv)
      this.mapDiv.zoomControl.setPosition('bottomright')
    },

    addMarkers: function (geojson, iconUrl) {
      // Initialize new geojson layer
      this.layers.markerGroup = L.markerClusterGroup({ showCoverageOnHover: false })
      this.layers.markers = L.geoJSON(geojson, {
        pointToLayer: (feature, latlng) => {
          return L.marker(latlng, {
            icon: L.icon({ iconUrl, iconSize: [30, 30] }),
            title: feature.properties.name
          })
        },
        onEachFeature: this.sendDescription.bind(this)
      })
      this.layers.markerGroup.addLayer(this.layers.markers)
    },

    sendDescription (feature, layer) {
      // create popup contents
      const customPopup = L.popup()
        .setContent(`<h4>${feature.properties.name}</h4>`)
      const customOptions = {
        className: 'custom',
        closeButton: false
      }
      layer.bindPopup(customPopup, customOptions)
      layer.on({
        click: (e) => {
          // const desc = await this.api.getDescription(feature.properties.id)
          // bus.$emit('markerClicked', desc)
        }
      })
    },

    async loadMapData () {
      const geojson = await store.api.getMarkers()
      this.addMarkers(geojson, 'https://upload.wikimedia.org/wikipedia/commons/thumb/3/30/Circle-icons-scissors.svg/1024px-Circle-icons-scissors.svg.png')
      this.mapDiv.addLayer(this.layers.markerGroup)
    }
  },

  mounted () {
    utils.initializeApi()
    this.setupLeafletMap()
    this.loadMapData()
  },

  created () {
    bus.$on('clicked', () => {
      this.mapDiv.setView(this.center, this.zoom)
    })
    bus.$on('regionSelected', e => this.mapDiv.flyToBounds(e.bounds, { duration: 1 }))
  }
}

</script>

<style>
 #mapContainer {
  position: relative;
  left: 0;
  z-index: 0;
  margin: 0;
  width: 100%;
  height: 100%;
}

.leaflet-popup-content-wrapper,
.leaflet-popup-tip {
  background: #222;
  color: #FFF;
}
</style>
