<template>
  <div>
    <div>
      <button v-if="userStationsLeft > 0" name="button" @click="addMarker">Добавить станцию метро (осталось {{userStationsLeft}})</button>
      <button v-if="userStationAdded > 0" name="button" @click="removeLast">Удалить последнюю</button>
      <button v-if="userStationAdded > 0" name="button" @click="removeAll">Удалить все</button>
      <button v-if="userStationAdded >= 4" name="button" @click="showResults">Готово</button>
      <hr />
      <table id="metro-lines-settings">
        <tr>
          <th>Линии метро</th>
          <th>Показать станции</th>
          <th>Показать линию</th>
        </tr>
        <tr v-for="(item, index) in metroData" :key="index">
          <td>
            {{ item.name }}</td>
          <td style="text-align: center">
            <input v-model="item.markersVisible" type="checkbox" />
          </td>
          <td style="text-align: center">
            <input v-model="item.polyline.visible" type="checkbox" />
          </td>
        </tr>
      </table>
    </div>
    <l-map
      ref="map"
      :zoom.sync="zoom"
      :options="mapOptions"
      :center="center"
      :bounds="bounds"
      :min-zoom="minZoom"
      :max-zoom="maxZoom"
      style="height: 500px; width: 100%"
    >
      <l-control-layers
        :position="layersPosition"
        :collapsed="false"
        :sort-layers="true"
      />
      <l-tile-layer
        v-for="tileProvider in tileProviders"
        :key="tileProvider.name"
        :name="tileProvider.name"
        :visible="tileProvider.visible"
        :url="tileProvider.url"
        :attribution="tileProvider.attribution"
        :token="token"
        layer-type="base"
      />
      <l-control-zoom :position="zoomPosition" />
      <l-control-attribution
        :position="attributionPosition"
        :prefix="attributionPrefix"
      />
      <l-control-scale :imperial="imperial" />
      <l-marker
        v-for="marker in markers"
        :key="marker.id"
        :visible="marker.visible"
        :draggable="marker.draggable"
        :lat-lng.sync="marker.position"
        :icon="marker.icon"
        @click="alert(marker)"
      >
        <l-popup :content="marker.tooltip" />
        <l-tooltip :content="marker.tooltip" />
      </l-marker>
      
      <l-layer-group
        v-for="item in metroData"
        :key="item.id"
        :visible.sync="item.visible"
        layer-type="overlay"
        :name="item.name"
      >
        <l-layer-group :visible="item.markersVisible">
          <l-marker
            v-for="marker in item.markers"
            :key="marker.id"
            :visible="marker.visible"
            :draggable="marker.draggable"
            :lat-lng="marker.position"
            @click="alert(marker)"
          >
            <l-icon
              :icon-size="dynamicSize"
              :icon-anchor="dynamicAnchor"
              icon-url="/images/metro-marker.png" >
            </l-icon>
          </l-marker>
        </l-layer-group>
        <l-polyline
          :lat-lngs="item.polyline.points"
          :visible="item.polyline.visible"
        />
      </l-layer-group>
    </l-map>
    <a href="#" id="getScreenShot" @click="getScreenShot">Get Screenshot</a>
    <img v-if="mapScreenshot" :src="mapScreenshot" id="mapScreenshot"/>
  </div>
</template>

<script>
import "leaflet";
import { latLngBounds } from "leaflet";
import {
  LMap,
  LTileLayer,
  LMarker,
  LPolyline,
  LLayerGroup,
  LTooltip,
  LPopup,
  LControlZoom,
  LControlAttribution,
  LControlScale,
  LControlLayers,
  LIcon,
} from "vue2-leaflet";

import "leaflet-simple-map-screenshoter";

const stations1 = [
  {
    id: "s2",
    position: { lat: 53.848379, lng: 27.474202 },
    tooltip: "Малиновка",
    draggable: false,
    visible: true
  },{
    id: "s3",
    position: { lat: 53.864788, lng: 27.485843 },
    tooltip: "Петровщина",
    draggable: false,
    visible: true
  },{
    id: "s4",
    position: { lat: 53.876945, lng: 27.497036 },
    tooltip: "Михалово",
    draggable: false,
    visible: true
  },{
    id: "s5",
    position: { lat: 53.886562, lng: 27.514768 },
    tooltip: "Грушевка",
    draggable: false,
    visible: true
  },{
    id: "s6",
    position: { lat: 53.886053, lng: 27.540352 },
    tooltip: "Институт Культуры",
    draggable: false,
    visible: true
  },{
    id: "s7",
    position: { lat: 53.893524, lng: 27.548114 },
    tooltip: "пл. Ленина",
    draggable: false,
    visible: true
  },{
    id: "s8",
    position: { lat: 53.902263, lng: 27.56193 },
    tooltip: "Октябрьская",
    draggable: false,
    visible: true
  },{
    id: "s9",
    position: { lat: 53.90927, lng: 27.57527 },
    tooltip: "пл. Победы",
    draggable: false,
    visible: true
  },{
    id: "s10",
    position: { lat: 53.915926, lng: 27.583759 },
    tooltip: "пл. Я. Коласа",
    draggable: false,
    visible: true
  },{
    id: "s11",
    position: { lat: 53.922401, lng: 27.600198 },
    tooltip: "Академия Наук",
    draggable: false,
    visible: true
  },{
    id: "s12",
    position: { lat: 53.924288, lng: 27.613143 },
    tooltip: "Парк Челюскинцев",
    draggable: false,
    visible: true
  },{
    id: "s13",
    position: { lat: 53.928084, lng: 27.627462 },
    tooltip: "Московская",
    draggable: false,
    visible: true
  },{
    id: "s14",
    position: { lat: 53.934605, lng: 27.651097 },
    tooltip: "Восток",
    draggable: false,
    visible: true
  },{
    id: "s15",
    position: { lat: 53.93885, lng: 27.666206 },
    tooltip: "Борисовский тракт",
    draggable: false,
    visible: true
  },{
    id: "s16",
    position: { lat: 53.945565, lng: 27.687676 },
    tooltip: "Уручье",
    draggable: false,
    visible: true
  },
  {
    id: "s1",
    position: { lat: 53.951108, lng: 27.706165 },
    tooltip: "Смоленская",
    draggable: false,
    visible: true
  },
];


const tileProviders = [
  {
    name: "OpenStreetMap",
    visible: true,
    attribution:
      '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
    url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
  },
  {
    name: "OpenTopoMap",
    visible: false,
    url: "https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png",
    attribution:
      'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)'
  }
];

export default {
  name: "LeafletMap",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPolyline,
    LLayerGroup,
    LTooltip,
    LPopup,
    LControlZoom,
    LControlAttribution,
    LControlScale,
    LControlLayers,
    LIcon,
  },
  data() {
    return {
      simpleMapScreenshoter: null,
      center: [53.9000000, 27.5666700],
      token: "your token if using mapbox",
      mapOptions: {
        zoomControl: false,
        attributionControl: false,
        zoomSnap: true
      },
      mapScreenshot: "",
      zoom: 11,
      minZoom: 1,
      maxZoom: 20,
      zoomPosition: "topleft",
      attributionPosition: "bottomright",
      layersPosition: "topright",
      attributionPrefix: "TUT.BY",
      imperial: false,
      Positions: ["topleft", "topright", "bottomleft", "bottomright"],
      tileProviders: tileProviders,
      userStationsCount: 14,
      markers: [
      ],
      iconSize: 32,
      polylines: [
        {
          id: "p1",
          points: [
            { lat: 37.772, lng: -122.214 },
            { lat: 21.291, lng: -157.821 },
            { lat: -18.142, lng: -181.569 },
            { lat: -27.467, lng: -206.973 }
          ],
          visible: true
        },
      ],
      metroData: [
        {
          id: "l1",
          name: "Московская линия",
          markers: stations1,
          polyline: { points: this.getLineCoords(1), visible: true },
          visible: true,
          markersVisible: true
        }
      ],
    };
  },
  methods: {
    alert(item) {
      alert("this is " + JSON.stringify(item));
    },
    getLineCoords: function(lineNumber) {
      if (lineNumber === 1) return [
        { lat: 53.848379, lng: 27.474202 },
        { lat: 53.864788, lng: 27.485843 },
        { lat: 53.876945, lng: 27.497036 },
        { lat: 53.886562, lng: 27.514768 },
        { lat: 53.886053, lng: 27.540352 },
        { lat: 53.893524, lng: 27.548114 },
        { lat: 53.902263, lng: 27.56193 },
        { lat: 53.90927, lng: 27.57527 },
        { lat: 53.915926, lng: 27.583759 },
        { lat: 53.922401, lng: 27.600198 },
        { lat: 53.924288, lng: 27.613143 },
        { lat: 53.928084, lng: 27.627462 },
        { lat: 53.934605, lng: 27.651097 },
        { lat: 53.93885, lng: 27.666206 },
        { lat: 53.945565, lng: 27.687676 },
        { lat: 53.951108, lng: 27.706165 },
      ];
    },
    getScreenShot: function() {
      const map = this.$refs.map.mapObject;
      const that = this;
      if (!this.simpleMapScreenshoter) {
        this.simpleMapScreenshoter = window.L.simpleMapScreenshoter({
          screenName: function() {
            return new Date().toDateString();
          }
        }).addTo(map);
      }

      this.simpleMapScreenshoter
        .takeScreen("blob", {
          caption: function() {
            return "Hello world";
          }
        })
        .then(blob => {
          const reader = new FileReader();
          reader.onload = function() {
              const dataUrl = reader.result;
              const base64 = `data:image/jpeg;base64, ${dataUrl.split(',')[1]}`;
              that.mapScreenshot = base64;
          };
          reader.readAsDataURL(blob);
        })
        .catch(e => {
          alert(e.toString());
        });
    },
    putimage: function(src) {
      console.log(src)
    },
    addMarker: function() {
      const newMarker = {
        position: { lat: 53.902263, lng: 27.56193 },
        draggable: true,
        visible: true
      };
      this.markers.push(newMarker);
    },
    removeLast: function() {
      this.markers.pop();
    },
    removeAll: function() {
      this.markers.splice(0, this.markers.length);
    },
    showResults: function() {
      console.log('game over!')
    },
    removeMarker: function(index) {
      this.markers.splice(index, 1);
    },

    fitPolyline: function() {
      const bounds = latLngBounds(stations1.map(o => o.position));
      console.log(bounds)
      //this.bounds = bounds;
    }
  },
  computed: {
    dynamicSize () {
      return [this.iconSize, this.iconSize * 1.15];
    },
    dynamicAnchor () {
      return [this.iconSize / 2, this.iconSize * 1.15];
    },
    userStationsLeft () {
      return this.userStationsCount - this.markers.length;
    },
    userStationAdded () {
      return this.markers.length
    },
  },
  mounted() {
    this.$nextTick(() => {
      //const map = this.$refs.map.mapObject;

      /*var simpleMapScreenshoter = window.L.simpleMapScreenshoter({
        hidden: true
      }).addTo(map);*/

      // window.L.simpleMapScreenshoter({
      //   screenName: function() {
      //     return new Date().toDateString();
      //   }
      // }).addTo(map);
    });
  }
};
</script>

<style scoped>
  #metro-lines-settings th {
    text-align: left;
  }
</style>
