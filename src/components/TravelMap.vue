<template>
  <GoogleMapLoader
    :mapConfig="mapConfig"
    apiKey=""
    :mountain_id="mountain_id"
  >
  </GoogleMapLoader>

</template>

<script>
import { defineComponent } from 'vue';
import GoogleMapLoader from "./GoogleMapLoader";
import { mapSettings } from "@/constants/mapSettings";

export default defineComponent({
  props: {
    mountain_id: String
  },

  components: {
    GoogleMapLoader
  },

  data() {
    return {
      mapOptions: {
        center: {
          lat: 44,
          lng: 145
        },
        zoom: 4
      },
      markers: [
        {
          id: 1,
          position: { lat: 145, lng: 44 }
        },
        {
          id: 2,
          position: { lat: 5, lng: 99 }
        },
        {
          id: 3,
          position: { lat: 6, lng: 97 }
        }
      ],
      lines: [
        {
          id: "1",
          path: [{ lat: 3, lng: 101 }, { lat: 5, lng: 99 }]
        },
        {
          id: "2",
          path: [{ lat: 5, lng: 99 }, { lat: 6, lng: 97 }]
        }
      ]
    };
  },

  computed: {
    mapConfig() {
      const marks = this.markers.find(elem => elem["id"] == this.mountain_id);

      return {
        ...this.mapOptions,
        marks
      };
    },

    mapCenter() {
      return this.markers[1].position;
    }
  }
});
</script>
