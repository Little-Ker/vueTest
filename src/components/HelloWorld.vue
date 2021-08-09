<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇所在區域 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="cityName" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="cityName" class="form-control" v-model="select.city">
              <option value="">請選擇縣市</option>
              <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                {{ c.CityName }}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="area" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="area" class="form-control" v-model="select.area">
              <option value="">請選擇地區</option>
              <!-- 建立專案時若有選擇 Airbnb 相關規範設定的話
              需留意過長的程式碼會造成 line length error，換行即可解決此問題 -->
              <option :value="a.AreaName" v-for="a in cityName.find((city) => city.CityName === select.city).AreaList" :key="a.AreaName">
                {{ a.AreaName }}
              </option>
            </select>
          </div>
        </div>
      </div>
      <!-- 顯示藥局位置 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import cityName from '../assets/data/taiwanAreaData.json';
import axios from 'axios';
// L 代表 Leaflet
import L from 'leaflet';
// 設定空物件
let openStreetMap = {};

export default {
  name: 'App',
  data: () => ({
    data: [],
    cityName,
    select: {
      city: '臺北市',
      area: '',
    },
  }),
  computed: {
    pharmacies() {
      return this.data.filter((pharmacy) => {
        if (!this.select.area) {
          return pharmacy.properties.county === this.select.city;
        }
        return pharmacy.properties.town === this.select.area;
      });
    },
  },
  watch: {
    // pharmacies(value) {
    //   console.log(value);
    //   this.updateMap();
    // },
  },
  methods: {
    updateMap() {
//       openStreetMap.panTo([37.91082, 128.73583], {
// animate: true
// })
      // clear markers
      openStreetMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          openStreetMap.removeLayer(layer);
        }
      });

      // 客製化icon
      const customIcon = L.icon({
        iconUrl: 'https://img-premium.flaticon.com/png/512/1078/1078454.png?token=exp=1627801949~hmac=5f95a748b7120a098db1d4473f0defab',
        iconSize: [42, 42],
      });
  
      // add markers
      this.pharmacies.forEach((pharmacy) => {
        const center = [pharmacy.geometry.coordinates[1], pharmacy.geometry.coordinates[0]];
        // 透過藥局經緯度疊加標記
        const marker = L.marker(center, {
            icon: customIcon,
            title: '跟 <a> 的 title 一樣', // 跟 <a> 的 title 一樣
            opacity: 1.0,
        }
        ).addTo(openStreetMap);

        // 地圖中心點改變
        openStreetMap.panTo(center)

        // 圖形標記
        // L.circle(center, {
        //     color: 'red',
        //     fillColor: '#f03',
        //     fillOpacity: 0.5,
        //     radius: 500
        // }).addTo(openStreetMap);

          // 點選標記時有說明框
          marker.bindPopup(`<p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
              <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 - ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 - ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}</strong><br>
              地址: ${pharmacy.properties.address}<br>
              電話: ${pharmacy.properties.phone}<br>
              <small>最後更新時間: ${pharmacy.properties.updated}</small>`);

          // 滑鼠靠近地標時有說明文字
           marker.bindTooltip("my tooltip text", {
            direction: 'bottom', // right、left、top、bottom、center。default: auto
            sticky: false, // true 跟著滑鼠移動。default: false
            permanent: false, // 是滑鼠移過才出現，還是一直出現
            opacity: 1.0
          }).openTooltip();



      })

  
  
    }
  },
  mounted() {
    const zoo = [24.9983469, 121.5810358];
    // 生成地圖
    openStreetMap = L.map('map', {
      center: zoo,
      // 可以嘗試改變 zoom 的數值
      // 筆者嘗試後覺得 18 的縮放比例是較適當的查詢範圍
      zoom: 19,
      zoomControl: true , // 是否秀出 - + 按鈕
    });
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 19,
    }).addTo(openStreetMap);


    // AXIOS取得藥局資料
    axios.get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json')
      .then(response => {
        this.data = response.data.features;
        console.log(' this.data', this.data)
      });
  }
}
</script>

<!-- ... -->

<style>

  #map {
    position: relative;
    width: 480px;
    height: 540px;
  }
</style>