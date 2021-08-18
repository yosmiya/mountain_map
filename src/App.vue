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
      prefectures: '0',
      infoWindow: null,
      contentString: '',
      mountainData: [],
      searchFlg: false,
      mapConfig: {
        center: {
          lat: 35.9675,
          lng: 138.09333
        },
        zoom: 7
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
      if ( state.mountain_id == 0 && state.prefectures == 0 )
      {
        return [];
      }

      state.searchFlg = true;
      let mountainList = mountain_data;

      if ( state.mountain_id != 0 )
      {
        mountainList = mountainList.filter(data => {
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
          <h2>${data.name}（${data.kana}）</h2>
          <table class="info_table">
            <tr><th>標高</th><td>${data.elevation} m</td></tr>
            <tr><th>都道府県</th><td>${data.prefectures} </td></tr>
            <tr><th>所在等</th><td>${data.location} </td></tr>
            <tr><th>緯度</th><td>${data.lat} </td></tr>
            <tr><th>経度</th><td>${data.lng} </td></tr>
          </table>
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
  <header>
    <h1>山岳検索くん</h1>
  </header>
  <div class="mainContent">

    <div class="leftContent">
    
      <div class="searchContent">
        <div class="searchBlock">

          <div class="cp_ipselect cp_sl04" :class="{'cp_ipselect_selected': state.mountain_id != 0}">
            <select v-model="state.mountain_id">
              <option value="0">標高（m）</option>
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

          <div class="cp_ipselect cp_sl04" :class="{'cp_ipselect_selected': state.prefectures != 0}">
            <select v-model="state.prefectures">
              <option value="0">都道府県</option>
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
          <div class="button-wrapper">
            <button @click="onClickSearchButton" class="c-button">検索する</button>
          </div>
        </div>
      </div>

      <div class="mountainList">
        <div v-if="state.mountainData.length > 0">
          <div class="mountainList_data" v-for="data in state.mountainData" :key="data.id" v-on:click="clickMountainList(data)">
            <span class="mountainList_data_name">{{ data.name }}</span> {{ data.elevation }}m
            <br>
            <span class="mountainList_data_sub">{{ data.prefectures }} {{ data.location }}</span>
          </div>
        </div>
        <div v-else-if="state.searchFlg">
          検索条件に合致するデータがありません
        </div>
      </div>
    </div>

    <div class="rightContent">
      <div class="map" ref="googleMap"></div>
    </div>

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

.leftContent {
width: 40vw;
}

.rightContent {
width: 60vw;
}

.searchContent {
  font-size: 0.8rem;
  margin-right: 15px;
}

.searchBlock {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.searchBlock_title {
  font-weight: bold;
}

.mountainList {
  position: relative;
  margin-right: 15px;
  overflow: scroll;
  height: 540px;
}

.mountainList_data {
  font-size: 0.8rem;
  border-top: 1px solid #eee;
  padding: 11px;
  cursor: pointer;
}

.mountainList_data:hover {
  background-color: #eee;
}

.mountainList_data:last-of-type {
  border-bottom: 1px solid #eee;
}

.mountainList_data_name {
  display: inline-block;
  font-size: 0.85rem;
  font-weight: bold;
}

.mountainList_data_sub {
  color: #888;
}

.mountainList_data_bottom {
  position: absolute;
  bottom: 0;
}

.map {
  //width: calc(100% - 35vw);
  width: 100%;
  height: 600px;
}

.cite {
  margin-top: 50px;
  font-size: 0.9rem;
  text-align: right;
}



.button-wrapper {
  display: inline-block;
}

.cp_ipselect {
	overflow: hidden;
	width: 140px;
	/* margin: 2em auto; */
	text-align: center;
  display: inline-block;
  margin-right: 10px;
}
.cp_ipselect select {
	width: 100%;
	padding-right: 1em;
	cursor: pointer;
	text-indent: 0.01px;
	text-overflow: ellipsis;
	border: none;
	outline: none;
	background: transparent;
	background-image: none;
	box-shadow: none;
	-webkit-appearance: none;
	appearance: none;
}
.cp_ipselect select::-ms-expand {
  display: none;
}
.cp_ipselect.cp_sl04 {
	position: relative;
	border-radius: 2px;
	border: 1px solid #ccc;
  border-radius: 50px;
	background: #ffffff;
}
.cp_ipselect.cp_sl04::before {
	position: absolute;
	top: 1.1em;
	right: 0.8em;
	width: 0;
	height: 0;
	padding: 0;
	content: '';
	border-left: 6px solid transparent;
	border-right: 6px solid transparent;
	border-top: 6px solid #333;
	pointer-events: none;
}
.cp_ipselect.cp_sl04 select {
	padding: 10px 38px 10px 14px;
	color: #333;
}
.searchBlock .cp_ipselect_selected {
  border: 2px solid #333;
}

.c-button {
  appearance: none;
  border: 0;
  border-radius: 50px;
  background: #4676D7;
  color: #fff;
  padding: 10px 14px;
  cursor: pointer;
}

.c-button:hover {
  background: #1d49aa;
}

.info_table {
  margin-bottom: 10px;
  text-align: left;
}

.info_table th {
  text-align: left;
}
</style>