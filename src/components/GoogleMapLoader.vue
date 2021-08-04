<template>
  <div>
    <div
      class="google-map"
      ref="googleMap"
    ></div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import GoogleMapsApiLoader from "google-maps-api-loader";

export default defineComponent({
  props: {
    mapConfig: Object,
    apiKey: String,
    mountain_id: String
  },

  data() {
    return {
      google: null,
      map: null
    };
  },

  async mounted() {
    const googleMapApi = await GoogleMapsApiLoader({
      apiKey: this.apiKey
    });
    this.google = googleMapApi;
    this.initializeMap();
  },

  methods: {
    initializeMap() {
      const mapContainer = this.$refs.googleMap;
      const map = new this.google.maps.Map(mapContainer, this.mapConfig);

      const myLatLng = { lat: 44, lng: 145 };
      new this.google.maps.Marker({
        position: myLatLng,
        map,
        title: "Hello World!",
      });
    }
  }
});
</script>

<style scoped>
.google-map {
  width: 100%;
  min-height: 100%;
}
</style>
