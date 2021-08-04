<script>
import { defineComponent, reactive, ref, onMounted } from "vue";
import GoogleMapsApiLoader from "google-maps-api-loader";
import axios from "axios";

import mountain_data from "./assets/mountain_data.json";

export default defineComponent({
  setup() {
    const GOOGLEMAP_API_KEY = '';
    let markers = [];
    const googleMap = ref(null);  
    
    const state = reactive({
      google: null,
      map: null,
      mountain_id: 0,
      infoWindow: null,
      contentString: '',
      mountainData: [],
      mapConfig: {
        center: {
          lat: 35.68944,
          lng: 139.69167
        },
        zoom: 5
      }
    });

    // const loadGoogleMaps = async() => {
    //   state.google = await GoogleMapsApiLoader({
    //     apiKey: 'API_KEY'
    //   });
      
    //   initializeMap();
     
    // };

    // const initializeMap = () => {
    //   new state.google.maps.Map(googleMap.value, state.mapConfig);
    // }
    const getMountainData = () => {
      console.log(mountain_data);
      // axios
      //   .get("./mountain_data.json")
      //   .then(res => {
      //     console.log(res);
      //   });
      
    }

    /*
     * マーカーをすべて削除する
     */
    const hideMarkers = () => {
      setMarker(null);
      markers = [];
    }

    /* 
     * マーカーを地図上に表示する
     */
    const addMarker = () => {
      // 検索条件に応じたマーカーの絞り込み
      const mountainList = mountain_data.filter(data => {
        switch(state.mountain_id) {
          case "1":
            return data.elevation >= 500 && data.elevation <= 1000;
          case "2":
            return data.elevation >= 1000 && data.elevation <= 1500;
          case "3":
            return data.elevation >= 1500 && data.elevation <= 2000;
          default:
            return data.elevation >= 0 && data.elevation <= 500;
        }
      });

      // マーカーの情報を設定する
      mountainList.forEach(data => {
        const marker = new state.google.maps.Marker({
          position: { lat: data.position.lat, lng: data.position.lng },
          map: state.map
        });



        marker.addListener("click", () => {
          openInfoWindow(data);
        });

        //markers.push(marker);
        markers[data.id] = marker;
      });

      setMarker(state.map);
      state.mountainData = mountainList;
    }

    /*
     * マーカーの追加・削除処理を実行する
     */
    const setMarker = (map) => {
      markers.forEach(marker => {
        marker.setMap(map);
      });
    }

    const clickMountainList = (data) => {
      openInfoWindow(data);
    }

    const openInfoWindow = (data) => {
      const contentString = `
        <div>
          <p><b>${data.name}（${data.kana}）</b></p>
          <p>標高 ${data.elevation} m</p>
          <div>
            <a href='http://www.google.co.jp/search?q=${data.name}' target="_blank">「${data.name}」で検索する</a>
          </div>
        </div>
      `;

      state.infoWindow.setContent(contentString);
      state.infoWindow.open({
        anchor: markers[data.id], 
        map: state.map,
        shouldFocus: false
      });
    }

    onMounted(async () => {
      state.google = await GoogleMapsApiLoader({
        apiKey: GOOGLEMAP_API_KEY
      });

      const infoWindow = new state.google.maps.InfoWindow();
      const map = new state.google.maps.Map(googleMap.value, state.mapConfig);

      map.addListener('click', function() {
        if (infoWindow) infoWindow.close();
      });

      state.infoWindow = infoWindow;
      state.map = map;
    });

    return {
      state,
      googleMap,
      hideMarkers,
      addMarker,
      clickMountainList,
      openInfoWindow

    };
  }
});
</script>

<template>
  <div>
    <div class="searchContent">
      <div>
        <span>標高(m)</span>
        <br>
        <input type="radio" id="one" value="1" v-model="state.mountain_id" v-on:change="hideMarkers(); addMarker()" /><label for="one">500 - 1000</label>
        <br>
        <input type="radio" id="two" value="2" v-model="state.mountain_id" v-on:change="hideMarkers(); addMarker()" /><label for="two">1000 - 1500</label>
        <br>
        <input type="radio" id="three" value="3" v-model="state.mountain_id" v-on:change="hideMarkers(); addMarker()" /><label for="three">1500 - 2000</label>
      </div>
      <!-- <div>
        <button v-on:click="hideMarkers">マーカー全削除</button>
      </div> -->
    </div>
  </div>
  <div class="mainContent">
    <div class="mountainList">
      <div class="mountainList_data" v-for="data in state.mountainData" :key="data.id" v-on:click="clickMountainList(data)">
        <span class="mountainList_data_name"><b>{{ data.name }}</b>（{{ data.kana }}）</span>
        <br>
        <span>標高：{{ data.elevation }}m 場所：{{ data.prefectures }} {{ data.location }}</span>
      </div>
    </div>
    <div class="map" ref="googleMap"></div>
  </div>
  <div class="cite">
    <cite>
    「日本の主な山岳一覧」（国土地理院）（https://www.gsi.go.jp/kihonjohochousa/kihonjohochousa41140.html）をもとに作成
    </cite>
  </div>
</template>

<style lang="scss" scoped>
.searchContent {
  font-size: 0.8rem;
}

.mainContent {
  display: flex;
  padding: 10px;
}

.mountainList {
  width: 35vw;
  margin-right: 15px;
}

.mountainList_data {
  font-size: 0.8rem;
  border-top: 1px solid #CCC;
  padding-top: 10px;
  padding-bottom: 10px;
  cursor: pointer;
}

.mountainList_data:hover {
  background-color: #DDD;
}

.mountainList_data:last-of-type {
  border-bottom: 1px solid #CCC;
}

.mountainList_data_name {
  display: inline-block;
}

.map {
  width: calc(100% - 35vw);
  height: 600px;
}

.cite {
  margin-top: 50px;
  font-size: 0.9rem;
  text-align: right;
}
</style>