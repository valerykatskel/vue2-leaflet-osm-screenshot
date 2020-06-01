<template>
  <div>
    <div>
      <h3>List of Markers</h3>
      <button name="button" @click="addMarker">Add a marker</button>
      <table>
        <tr>
          <th>Marker</th>
          <th>Latitude</th>
          <th>Longitude</th>
          <th>Tooltip</th>
          <th>Is Draggable ?</th>
          <th>Is Visible ?</th>
          <th>Remove</th>
        </tr>
        <tr v-for="(item, index) in markers" :key="index">
          <td>{{ "Marker " + (index + 1) }}</td>
          <td>
            <input v-model.number="item.position.lat" type="number" />
          </td>
          <td>
            <input v-model.number="item.position.lng" type="number" />
          </td>
          <td>
            <input v-model="item.tooltip" type="text" />
          </td>
          <td style="text-align: center">
            <input v-model="item.draggable" type="checkbox" />
          </td>
          <td style="text-align: center">
            <input v-model="item.visible" type="checkbox" />
          </td>
          <td style="text-align: center">
            <input type="button" value="X" @click="removeMarker(index)" />
          </td>
        </tr>
      </table>

      <hr />
      <table>
        <tr>
          <th>Layer</th>
          <th>Is Visible ?</th>
          <th>Are Markers visible ?</th>
          <th>Is Polyline visible ?</th>
        </tr>
        <tr v-for="(item, index) in stuff" :key="index">
          <td>{{ "Layer " + (index + 1) }}</td>
          <td style="text-align: center">
            <input v-model="item.visible" type="checkbox" />
          </td>
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
        v-for="item in stuff"
        :key="item.id"
        :visible.sync="item.visible"
        layer-type="overlay"
        name="Layer 1"
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
          @click="alert(item.polyline)"
        />
      </l-layer-group>
    </l-map>
    <a href="#" id="getScreenShot" @click="getScreenShot">Get Screenshot</a>
    <img v-if="mapScreenshot" :src="mapScreenshot" id="mapScreenshot"/>
    {{ mapScreenshot }}
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
      stuff: [
        {
          id: "l1",
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
      const simpleMapScreenshoter = window.L.simpleMapScreenshoter({
        screenName: function() {
          return new Date().toDateString();
        }
      }).addTo(map);

      // const simpleMapScreenshoter = window.L.simpleMapScreenshoter({
      //   screenName: function() {
      //     return new Date().toDateString();
      //   }
      // }).addTo(map);

      simpleMapScreenshoter
        .takeScreen("blob", {
          caption: function() {
            return "Hello world";
          }
        })
        .then(blob => {
          //this.blobToBase64(blob, this.putImage());
          //saveAs(blob, "screen.png");
          const reader = new FileReader();
          reader.onload = function() {
              const dataUrl = reader.result;
              const base64 = `data:image/jpeg;base64, ${dataUrl.split(',')[1]}`;
              console.log(base64);
              
              that.mapScreenshot = base64;
              //this.mapScreenshot = "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wCEAAkGBwgHBgkIBwgKCgkLDRYPDQwMDRsUFRAWIB0iIiAdHx8kKDQsJCYxJx8fLT0tMTU3Ojo6Iys/RD84QzQ5OjcBCgoKDQwNGg8PGjclHyU3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3Nzc3N//AABEIAIIAwwMBIgACEQEDEQH/xAAbAAABBQEBAAAAAAAAAAAAAAACAAEDBAUGB//EADwQAAIBAwMCBAMFBgQHAQAAAAECAwAEERIhMQVBEyJRYTJxgQYUQpHBFVKhsdHwIzNy4RY0Q1NikvGC/8QAGQEAAwEBAQAAAAAAAAAAAAAAAAECAwQF/8QAJREAAgICAgICAgMBAAAAAAAAAAECESExAxITQSJRBGEygcEj/9oADAMBAAIRAxEAPwDvySo1KcMu4I7GrHUrlZJUMeXXACgHknmq+rBxoYHjBxQNvOoGTtup/CK6eubOJWsEtq0OvVNCXVlwQO1bdvc28oCwkLgfBjBFYBj3JUlSeQOKY+ImDs4B+RFRPjUi4T66OjlJRSV3LHCj3/pQGHJ3kfPqAB+lULPqKtp8ds6RgNzg+9O3VHc5gChO2rcmufxSujo80Ui1daLcJKpCMvlA7MO4ory4EMGUKl2+D+tZ1xeNcRaGiVSGBLA5pdQdXuFZfgMa6PlVx4spMzly4bRXJLMSTuTk59aFmCjLHApnbGABljx7e9R8Nn45P4L/AErqRzVexyzMuWJjjHryacasYjTQPVj+lOqebU+7dvaj/l60A3QCpvljqPbtijpfOpLdVeZUcNucYHNJushsjpmAYEMAR6GrN4Io5fDhX4eW1Ekmq+d8URd5E8MjyY8BiSvZvT51h9c6191eTXdfc7KCRYJbkIHZp2Qusa6vKuwGWbbzKMdx0HPO9ch9qLaOCw6lYdQkmi6L1GaOae4ht1meFwAuCp4BCrhhupX3oldYNONJvJo2XVgb0xw337SsPHFq9w0IjkhnK6gG0+VlOMZwMHA3zW9XIfZmz6N91isfspDdfseK7F3c3lzzcSqMKiA9gcEnHYDfJx19KN1kXKo38RUJRWOSDn1BxRUqogj0P/3CfmAaJY1BycsfU/pRUqAsVKlSpCAfYaSQ4zgEAZBpgpJ0gBO5OxJ/vFTzQyRECWPAznONvoaAAZOBgnsKE0zRv9ChtmnlVNZJPvgD32qa46bJbDxEfKjkg/oaGPxomVo9SO/GBv8AlVz9oaMJLFrIXS5JG5rOUpX8S4pV8jKxqbfySAbEDkU+sr/mrt2Zdx+VG6K+24APlPcUKMwYo3PIPrWpnY+A4GDkeoovMcDJOBgZPAoDGhJJyD7Ej+VMY0Ay2oD3c/1pE4G1eaR+yjAoo1Cxjb5n3oVAbSFBEY34xqqxCFMg1kBRzmhuin9EZIVSxOABk1yFx1SwWK1uev3Nys97Cs8UH3uW1hgR/gRTGDrkwMnPB9K7pws8gSGPVnmTdRj9eK4bqFzD9kGgt+sfZNOsW0mm26fcCJWkRQSUgYMOxYhSORgHcZOLk5I14opPJrdHuStwbfx57m1lt1ubR7hSJlXUVeOTjJVh8XcHfjJ6OyhQoCkkiXBBI7D6GsfpnQ7qG2W/vLaO0lMSxJZQNqSzt1yVQHucklj9BxvvdNeUEJLEV8m0hU7fpSlJuFj6/PRn9XSWC5htrUANcM51CMuY0VcswH4m4AHqe9c30vrQ6pOsNhH1aB7iJrjp8l5h4r2NQNRZQP8AD3ONvUHviuz61aS3ccUtsqvcQE6Uk+CZGUh4z6Ajv6gVw3Q7LpqX9yv2e6PfW9/GpQm8k1R9NUjDLGM99zjuTniiDckmvRM1CDal/R0tpOt1aw3EedEqK4B7AjNKVFd1WRQySKVZWGQQexH509vClvbxQxDEcaBFz6AYFPKd1PfUMfxroMY2NawQ2tvHBaxJFDGulI0GAo9qkpUqAFSpUqQhUqVMSAMkgfOmA9Ko/Gj9T/6mlQOmdDoMkWibByMMBwarp0+NHDai2M4B4q2d+BQMwjwzZ2Oy/vGuFSa0d/RNoUGHiDZyZBqP5cVggYFa81yLSJY86pgOBwD/AErJLkfixk4wB/Ct+FNKzD8hpuhE4GTxWB9ouudO6F0yy6l127uYY72TFpBaIpfRkZdiR2BBx7gVvlTjQzAEg+m/t/EVzX2j6T0PqcFjafaaC7YWK+HaTQsSrRkjKlQRhsADPsDWz7NfHYuFL3s3I00TeC85mSSJZ4JwSPEjbgkev9RUwjQNnTlvU7mg6d0+ZIrcxW0sNvDAlvaxSya3SJeNRPf+WBWmnTpCuZHVfkM0nyJbYpxk5VEy76d7eAvFH4kzsscak4DOxCrk9hk1ny3MMX2nh+zb9dlHWJYDMF8Ffu+rnRpxngE/FnHfeuivOmfebAxW8yi4jkWaN2XiRCGXI9M4+lcVNJZyfbJeoPZ3P/EKyjTZi1jOMKAcTc6Ns6ucZ+VcfJzJvLPS/F/Gm+P/AJxt+9PHrfo73o1wLywSSSARSxs0UsY4DqdLY9RkbVdbHlOFwp79qpdItZLSyEU7B5ndpZXHGtiSce2+B8qukYXjPtWaMuXr3fXRHPiSJ4gMgjiq8Fz4SaLsBGxkMeG9atqDzt8h2ocB0dHGQxINVFrTM2m1a2U7+6YYSNXQHfWRjPyqjC7Ry+IS3xBmweTxv67UdxbyW7AMdSnZSNx8h70K280gOlCMDOWBGa6oqKicknJyI+dzyd6CTcKQCQrZIFStG6KrP+Lt+6aH3rVNNYM8p5GVgwDKQQR2p6iV1QupznVkADPO9FqdtlTA9XFMfUP3OwoDIp2Tzn/xH60vCBP+IS/z4/KjFAYRHh25On2Xc04iQHOMn1O9HRRxySnESFvXHApNpBl6BpVZHT7jHC/+1PU+SP2PxzNgbnFZnULiSOfyNpC7A/xP6flWgrBvKysjAZOT2+dV1dWaTMZIyGBK+w4rji0nbO6UW4ujIZmdsnLMdyeSacoQPMpGe5FbFrBFDkqhDE/E43+VS3MImiKEDPYsOK38+a9HN4HW8mHGjOQkaZPYAVb6fZ+JpluoVwpygYZyfXf0ozZ3NsoFvICXwGBHf1q+pREwCuEGDvxS5OXHxHx8bu2FjFZ017dRTyRG0LLn/DKHdgPlmrmtpceHlR+8Rj8hUiIsY8i8nc9/965kdKVGVFdXCOStjOWZW+LJPxbAnH/zvgVbiELOtzLaiGdkALkDUvcrnnmp3ysiP2PkI+fFO+yOYxlvwj+VN0x9qBMhl2jAZe7Hj/ekqOP+oykemMVBFc3JOqW0ZR8XPH++x/MUwuLoqddodWkHZsb43H55p9WGtFnwj/3X354GaHeAambXH+LUNx71CtxMcFrZgR6EgEfWp7aYzQiTSVU755yM806oOzE8YbSrKfIQykdiKMybeYEe9Db7IygYGo6dvepCur4m2pMmSyUJpLdpJY3G7HOOMYHPzNZ+nKa1zp7g8itLqAjMIdiniIfJkZz7Gq4sbhpZHhugiSEkqYwcZA2z7VtGSijBw7OigNpjvsVzR4xRSWdyt0UjkV8RnLGMd9/7+lOtjeGNi0yoW+HKA4O/+35Vr5VRD4WANyAOTVmGwnkUMQqKe5Of5VoR2yLb6YwNRAySOalVGSNFGBpG/pWcuZvRUeFXkrwWMC/EDIRyW4z8qtAKowigfIUCsDsuSKMc+9Ytt7N0ktC1DuGz8qVCWfO2r+FKppBZQujeGTyWwZgcDfysu3O+/f60ET36YCdPjTScBTJwud9/XvWp29vWhJHKHzLtTsrtZHczi0tprm5mihgiQvIzcKo7k1g2nXup9RW4nselwSQQPolje6CzqcZwUIwpwQdLMDvvir32kt57zodxDbqXmXw5FiX/AKuh1cqP9QUr9a8/tOlXD9Qu+txdd6fF0G66j4t5YlIY2aMNqPiNgHIbIKkau1bccV1urZV40ek9OuYep2kd4jNobKhCCjIwOCCOQwIwfTFSiKHx1KxqcqT65O29c/0+8kNtf3H3G8WO+unuIQI8YTSACQTsW05x771s2l22tBJbTKGyA54UA4/v/wCVHJBRboVtN/RdHmOMn3ohkmhVsuxYAds04IBGGz7VkZoa4wYiD3xj50LTxJbGaaVYoyu8jkKBtRNhiNX0xXG/a3qVx064uJoI7WV7YAwRXQdkXKszMFUEk7Y9hmj3RrBJ3fo6S2nluYmezvre4xyYnVgD6ce/8qJxesVQTLqOQMRjH12/v+FcP9k+t9V6zCOp3y9MElvcxwGWyjljModgNJDDDDBJ9iK7XrV7LZ2sS2yKbm4lEMOvhSQSWIHIChjjvjFaxty6hPScSx4N4T/zKY/0DI2+XrU0AkXaZxIWY4IGMe3FedWH2xiurm2Wz6h1jNy6pbXN1aR/drlydlKr5kB3AO3HfG/f9JvF6j0+C60eGZBlkJyUYbMPoQRVTg0rTsmya2/yIz3Kgn5mpHUMMGooP8pBwQMMPQ96kCgVmD2QXMZyskTKsgGkasf2Kx728+6IHvbhIEXyhpHCr+ZreZA4NcB1h+qQ9VuOu2wspbbpl00NxHcvpxFpjwq5BCklyxYb7Ab8VrxyrZHhU7bdJG9ZdQRpEubaZJ4ZBpLIwYH6irEtzK5GG0heAu1c1a9TPU+q2d2vTILGSS8+63KwXIkEnkdhrUKMMNKkH0eumktmVj4fmUb+9aXDbMubjcK66Zfs5zJbEuxLA4B9amjGmQjnI5zWQOru9sJLGwnntVOn7wWSNWPHl1MNXz/LNWrG7hvIfEiaQFcoyOpDIw5BHY1zS/kbdJRirLikiQhVyM1IzDJPHzqOM6FOSMfPionuraRARMhxuSDQrHTol0at9XNPVc3tspx96jHsaVHV/Quv6LAd1PmibB/dINCkgLFR8ZPwHY1MSQowd/fvUQiBx4ihtzgEZApFYJQrLkkb8/L2rnbv7F9FvftBF1u4tFNwg8yYHhyN+F2Hcj++K3Fhj5KAemNv5U/gkA4kdSf/ACq4TlDMXQ060wbyONhql30g0UceiNEYjYZOPU7mqV/1OOxiVLlJJnlPhwxQR6nlbHwgZ9BnPA74qCLrBhlgHU7C5skn0pHM5R42Y8AsrHSSeM7HOM1KTaBxbRs4A4H0pYH7opm1DbK54otqmiCFoyrak7b4rKvrW5mb75Yy+BfpkKxHlcfuMO4757ZrYkbEUhAOymq9orQsAzZDqMexGf0/kaOqaKhJxlgzukW3ULvw73rUi+JHvDbRgBYz+8R3b+VWuuWMt5axvaFVuraTxYdRwrHBBUnsCCRntnNHdj7s6ywqQWPmOf4VJDP4ihkY87jjergnCpImXN3llUeVdJ6J0mDqFnZdLXrzdXs5Y3+73TQrGhjPl8R1TJRcnHz27V6p020Tptjb2Pi63RfM5G7sd2b6kk1MgiSVpfDRZXADOF8zAcAn2zRIimVpMktwPb5Vpy8sZYiqX+hf0CFZWZo8Mrb6ScYP9ineQ7ExyDHPl4qU4GSRn6VHIwGTq04XUWJ4ArFF2vYGfGbMbD6GsL7RdIvtEt70Tw2uio8W1mQPDdAcalP4h2NSm/nvbeXqFuvT4rSMg+JdzFWwRkFtI8gIIIzvggkVc6N1RepQPJoKvG2llyGB4IYEcgg7HvRGVPAocijK0ZXTOlXfhydT62YX6rOuplijCpDwNscvgLljk7Y4rW6lDNLYXcMDhZXhdUP/AJEHFXiA5KsHwRkgjkVFEp+7EnLGMke5xTm3LI5ycpdvo5i+sZerP0LqfSgLixsIzFN09QuYn8ozhmVQy6SpzuATjOa2LCSG66v1W8tmU2zNGiuvDugOph+YH/5rO6x9mLTrXUILxo5IV4uNDACZR6/kN/T6VuW6JAFS3RY4o9lVRgKOwpyZUpRUbXsYXJwyi1ZjpGNQ2J9OKBnj8oWw3JOBoAHv29z+daK/T6Ux5yT86doiymGjYZNhudzmMZpVa8Mn8VKjsHYcKPVgMZxmmJxgZLDtvk0l3GOT6mm04H61lZNiY+YBtjvj2ptYLAkEmhcBz21ikwGdgdfp607Yrs5n7UQXUnVrCW3vv2eZEMKXLAEI/iIxTB2y6gqM1ldJ6R1ron2Aj6f1vq5nmnt/AtumeEhKMwwkYceYlTg54GPQV3F5Bb31rLaXkKyQyrpeNuGFYf2f+y0HRLmW7jmku5jkRtNu0SH8Kn+Z7+1axkqNYuL3s3PClSNSX8RgBnUOdt9xRePGRpDMp7hlPl9iai6jfLY2DXT6pcYVI1GDI7EKq7+pIFcuPtVD9++5H7TdHHUvG8H9nfd3x4mceH4mc5ztqx9KlRvJUYWreDsAHYDUwPfRTTYEeAMaSCB752qt026+92SXUMRjbLLLA3KMpIYD6j61PMUmQaWBIddhzzSoXRxkTOEYFSBuPnUMcSxAIvwDv3JpGMqmNTavTO1LxCVwVIwPSleKMWk8hKAB7GmWQxgnOQKEyKU0rnVvtjimC40l8FRufnUsRMz5BPfso5NQ3sP3myubNSVM0LJq9MgjP8aQH+JqUYJ25qxhSuNW/rT9D2qPLvtN0676v1uFn67bdItFtRDf2UkwR3kVXA8rKVkTzAd9ie9dz0AxEz3kaCC0KRQQLp0Aqi/Fj0Orb2FWOodG6f1Ca3ub+BZJLc5B4yPQ+o74q2sZdjIVGWUYXHwj+v8ASpTledIUIy3LSAnPjxSr4chDoVBwV29uDVGKzvAJEN2FbRjADAAlQA2A3Ygn083sK1wFA1Y4HrxVZSJJXmwCUJjXA4xya0s0bVYKS2N0DJqvpGBdmCkfCCRjvvsMb+vrVmCJreFRI7TOzBQ0hOT86sqwUg9sYOf4UUsesxkNjQ2obULImsWGCeCDn+FLGaehJOcdqBsamoxnFKlYhiN/aopUJI0n6ZqRM6sbU7rtkc0hMjQb45AoplBHl2YDYinQZ3OAf0osDOeflSBFc+c5bUxHIFJdjs2c9id6eUaTqU88CnjTWgaTOrufSnYin161luumPDa4W5Vlli1HALKc4J7Z4z715J/w30XIU9buP2xp+7iH9ln70X06catWNWPxZx3zXtDIdIDAHPBXbFVl6faJ1Fb428Zu/D8Px8b6fStITSwzeE4V15Ff0QdCs5un9KRLo5uZGaSXByAzHOPptvV6SMsqksAynIYjipucGmkXOMVLdsiU25ORAHLqSwwQTn2NEp1IdRJx70LxsjM4UlSPMO/zH98U/hZXIIbUMgg7VDREl7Q6R6l3yB7U6oVzn4aJNkHbNSj0piSIUYYyQKHIBwOfWlIABlePSoskHO+aQmySV9TrG+yk5J9cf2Ks68LnmqqYaUkNwuMt653qz+EhT223oLIzjS7O2Exn5VDaMFjJ4DksP9/4UCu0zK0ozHyIxx7Z9as+Ur9eDTExMQQQcHI27USMdlbbbnFMyKRzyOKWgqF0cjjJpjT9B5Gccmlyd6YMDlhnfmo3l0nPamwZNvT1D4jei0qVBgKPhvnSb4qVKpENIBqTYVDHs6Y23NNSpoCYHLnO+/6URAC7ClSoYMddwM+1VJWPhR7n4vWlSoWwJ7UkwjPvUp7UqVHsaH7/AENRwfC47B3x+dKlTZXoGT/JX50Y5X5UqVSjNbGf4T9Kgl2JI2IU4pUqA9hYAiUAAD0oSTj6ClSoHL+TJ7L/AJdP770b/rSpVXsZGeaMcUqVNAgfxN9P1qCY7D/UP50qVIHsMcClSpUiD//Z";//base64;
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
        position: { lat: 50.5505, lng: -0.09 },
        draggable: true,
        visible: true
      };
      this.markers.push(newMarker);
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
    }
  },
  mounted() {
    this.$nextTick(() => {
      const map = this.$refs.map.mapObject;

      /*var simpleMapScreenshoter = window.L.simpleMapScreenshoter({
        hidden: true
      }).addTo(map);*/

      window.L.simpleMapScreenshoter({
        screenName: function() {
          return new Date().toDateString();
        }
      }).addTo(map);
    });
  }
};
</script>
