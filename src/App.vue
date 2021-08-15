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
      prefectures: '',
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
      let mountainList = [];

      if ( state.mountain_id != 0 )
      {
        mountainList = mountain_data.filter(data => {
          switch(state.mountain_id) {
            case "1":
              return data.elevation >= 0 && data.elevation <= 500;
            case "2":
              return data.elevation >= 500 && data.elevation <= 1000;
            case "3":
              return data.elevation >= 1000 && data.elevation <= 1500;
            case "4":
              return data.elevation >= 1500 && data.elevation <= 2000;
            case "5":
              return data.elevation >= 2000 && data.elevation <= 2500;
            case "6":
              return data.elevation >= 2500 && data.elevation <= 3000;
            case "7":
              return data.elevation >= 3000 && data.elevation <= 3500;
            case "8":
              return data.elevation >= 3500 && data.elevation <= 4000;
            default:
              return true;
          }
        });
      }

      if ( state.prefectures != 0 )
      {
        mountainList = mountainList.filter(data => {
          return data.prefectures === state.prefectures;
        });
      }

      // マーカーの情報を設定する
      mountainList.forEach(data => {
        const marker = new state.google.maps.Marker({
          position: { lat: parseFloat(data.lat), lng: parseFloat(data.lng) },
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

    const onClickSearchButton = () => {
      hideMarkers(); 
      addMarker();
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
      openInfoWindow,
      onClickSearchButton
    };
  }
});
</script>

<template>
  <div class="mainContent">

    <div class="searchContent">
      <div class="searchBlock">
        <span class="searchBlock_title">標高(m)</span>
        <br>
        <!-- <br>
        <input type="radio" id="one" value="1" v-model="state.mountain_id" v-on:change="hideMarkers(); addMarker()" /><label for="one">500 - 1000</label>
        <br>
        <input type="radio" id="two" value="2" v-model="state.mountain_id" v-on:change="hideMarkers(); addMarker()" /><label for="two">1000 - 1500</label>
        <br>
        <input type="radio" id="three" value="3" v-model="state.mountain_id" v-on:change="hideMarkers(); addMarker()" /><label for="three">1500 - 2000</label> -->
        <select v-model="state.mountain_id">
          <option value="0">-</option>
          <option value="1">0 - 500</option>
          <option value="2">500 - 1000</option>
          <option value="3">1000 - 1500</option>
          <option value="4">1500 - 2000</option>
          <option value="5">2000 - 2500</option>
          <option value="6">2500 - 3000</option>
          <option value="7">3000 - 3500</option>
          <option value="8">3500 - 4000</option>
        </select>
      </div>
      <div class="searchBlock">
        <span class="searchBlock_title">都道府県</span>
        <br>
        <select name="ken" v-model="state.prefectures">
          <option value="0">-</option>
          <option value="北海道">北海道</option>
          <option value="青森県">青森県</option>
          <option value="岩手県">岩手県</option>
          <option value="宮城県">宮城県</option>
          <option value="秋田県">秋田県</option>
          <option value="山形県">山形県</option>
          <option value="福島県">福島県</option>
          <option value="茨城県">茨城県</option>
          <option value="栃木県">栃木県</option>
          <option value="群馬県">群馬県</option>
          <option value="埼玉県">埼玉県</option>
          <option value="千葉県">千葉県</option>
          <option value="東京都">東京都</option>
          <option value="神奈川県">神奈川県</option>
          <option value="新潟県">新潟県</option>
          <option value="富山県">富山県</option>
          <option value="石川県">石川県</option>
          <option value="福井県">福井県</option>
          <option value="山梨県">山梨県</option>
          <option value="長野県">長野県</option>
          <option value="岐阜県">岐阜県</option>
          <option value="静岡県">静岡県</option>
          <option value="愛知県">愛知県</option>
          <option value="三重県">三重県</option>
          <option value="滋賀県">滋賀県</option>
          <option value="京都府">京都府</option>
          <option value="大阪府">大阪府</option>
          <option value="兵庫県">兵庫県</option>
          <option value="奈良県">奈良県</option>
          <option value="和歌山県">和歌山県</option>
          <option value="鳥取県">鳥取県</option>
          <option value="島根県">島根県</option>
          <option value="岡山県">岡山県</option>
          <option value="広島県">広島県</option>
          <option value="山口県">山口県</option>
          <option value="徳島県">徳島県</option>
          <option value="香川県">香川県</option>
          <option value="愛媛県">愛媛県</option>
          <option value="高知県">高知県</option>
          <option value="福岡県">福岡県</option>
          <option value="佐賀県">佐賀県</option>
          <option value="長崎県">長崎県</option>
          <option value="熊本県">熊本県</option>
          <option value="大分県">大分県</option>
          <option value="宮崎県">宮崎県</option>
          <option value="鹿児島県">鹿児島県</option>
          <option value="沖縄県">沖縄県</option>
        </select>
      </div>
      <div class="searchBlock">
        <input type="button" value="検索する" @click="onClickSearchButton">
      </div>

      <!-- <div>
        <button v-on:click="hideMarkers">マーカー全削除</button>
      </div> -->
    </div>

    <div class="mountainList">
      <div class="mountainList_data" v-for="data in state.mountainData" :key="data.id" v-on:click="clickMountainList(data)">
        <span class="mountainList_data_name"><b>{{ data.name }} {{ data.elevation }}m</b></span>
        <br>
        <span>{{ data.prefectures }} {{ data.location }}</span>
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
.mainContent {
  display: flex;
}

.searchContent {
  font-size: 0.8rem;
  width: 12vw;
  margin-right: 15px;
}

.searchBlock {
  margin-bottom: 20px;
}

.searchBlock_title {
  font-weight: bold;
}

.mountainList {
  width: 40vw;
  margin-right: 15px;
  overflow: scroll;
  height: 550px;
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