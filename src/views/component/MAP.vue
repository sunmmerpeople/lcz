<!--
 * @Author: lzd
 * @Date: 2021-01-06 16:27:09
 * @LastEditors: lzd
 * @LastEditTime: 2021-02-18 18:49:50
 * @Description: content description
-->
<template>
  <div class="Mapm">
    <div class="map-view" id="map-view"></div>
    <div class="state-view UAV-flex box-shandow">
      <div class="state-item">
        <span>探测设备状态: </span>
        <span
          :class="probeDevState == '正常' ? 'state-normal' : 'state-warning'"
          >{{ probeDevState }}</span
        >
      </div>
      <div>
        <span>反制设备状态: </span>
        <span
          :class="cheatDevState == '正常' ? 'state-normal' : 'state-warning'"
          >{{ cheatDevState }}</span
        >
      </div>
    </div>
    <div class="audio">
      <div
        class="icon-btn iconfont icon-mobile"
        :class="vidoPlay ? 'play-class' : ''"
        @click="vidoPlay = !vidoPlay"
        :title="vidoPlay ? '关闭声音提示' : '打开声音提示'"
      ></div>
      <video ref="audio" :src="audioSrc" loop muted></video>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import "../../assets/js/Semicircle.js";
import "../../assets/js/leaflet.rotatedMarker.js";

const CONFIG = window.CONFIG;
export default {
  name: "Mapm",
  props: {
    probeDevState: {
      type: String,
      default: "正常",
    },
    cheatDevState: {
      type: String,
      default: "异常",
    },
    uavList: {
      type: Array,
      default: () => {
        return [
          // {
          //   id: "221321",
          //   azimuth: 28,
          //   model: "北斗无人机",
          //   frequency: "2.4GHz",
          //   isWhite: false,
          //   longitude: 112.980312728,
          //   latitude: 28.2186782319,
          // },
          // {
          //   id: "221321",
          //   azimuth: "28°",
          //   model: "北斗无人机",
          //   frequency: "2.4GHz",
          //   isWhite: true,
          //   longitude: 112.980312728,
          //   latitude: 28.2100782319,
          // },
        ];
      },
    },
    devLng: {
      type: Number,
      default: 112.979352788,
    },
    devLat: {
      type: Number,
      default: 28.2134782309,
    },
    carCourse: {
      type: Number,
      default: 30,
    },
  },
  components: {},
  data() {
    return {
      audioSrc: require("../../assets/voice/0.mp3"),
      carImg: require("../../assets/images/che.png"),
      UAVImg_b: require("../../assets/images/bwrj.png"),
      UAVImg_h: require("../../assets/images/hwrj.png"),
      carLayers: new L.layerGroup(),
      carCircle: null,
      carCircle_500: null,
      carCircle_1500: null,
      carDom: null,
      UAVLayers: new L.layerGroup(),
      setLayers: new L.layerGroup(),
      semiCircleLayers: new L.layerGroup(),
      vidoPlay: false,
    };
  },
  computed: {},
  watch: {
    haveNoWhiteUav(newval) {
      if (newval) {
        this.$refs.audio.muted = false;
      } else {
        this.$refs.audio.muted = true;
      }
    },
    carCourse() {
      this.carDom && this.carDom.setRotationAngle(this.carCourse);
    },
    devLng() {
      this.changeCarAndCricleLatLng(this.devLat, this.devLng);
    },
    devLat() {
      this.changeCarAndCricleLatLng(this.devLat, this.devLng);
    },
    uavList: {
      handler() {
        this.drawUAVList();
      },
      deep: true,
    },
    vidoPlay(newval) {
      if (newval) {
        this.$refs.audio.play();
        return;
      } else {
        this.$refs.audio.pause();
        return;
      }
    },
  },
  methods: {
    init() {
      this.map = L.map("map-view", {
        center: [this.devLat, this.devLng],
        zoom: 13,
        minZoom: 12,
        maxZoom: 18,
        attributionControl: false,
        doubleClickZoom: false,
        tap: true,
      });
      // L.tileLayer("http://localhost:8888/custom/{z}/{x}/{y}.png").addTo(
      //   this.map
      // );
      // L.tileLayer(
      //   "http://webrd02.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}"
      // ).addTo(this.map);

      // this.map.setMaxBounds([
      //   [28.303172, 112.588577],
      //   [28.303172, 112.72007],
      //   [28.18779, 112.589092],
      //   [28.186731, 112.719555],
      // ]);
      // debugger
      var baseLayers = {
        谷歌在线地图: L.tileLayer(CONFIG.onlineMapUrl).addTo(this.map),
        谷歌离线地图: L.tileLayer(CONFIG.offlineMapUrl),
      };

      L.control
        .layers(baseLayers, {}, { position: "topright" })
        .addTo(this.map);
    },
    refreshMp() {
      this.map.invalidateSize(true);
    },
    drawCricle() {
      let circle = L.circle([this.devLat, this.devLng], {
        radius: 1000,
        color: "green",
        fillColor: "green",
        dashArray: "8",
        weight: 2,
        fillOpacity: 0.1,
      });
      let circle_500 = L.circle([this.devLat, this.devLng], {
        radius: 500,
        color: "green",
        fillColor: "green",
        dashArray: "8",
        weight: 2,
        fillOpacity: 0.1,
      });
      let circle_1500 = L.circle([this.devLat, this.devLng], {
        radius: 5000,
        color: "green",
        fillColor: "green",
        dashArray: "8",
        weight: 2,
        fillOpacity: 0.01,
      });
      this.carLayers.addLayer(circle);
      this.carLayers.addLayer(circle_500);
      this.carLayers.addLayer(circle_1500);
      this.carCircle = circle;
      this.carCircle_500 = circle_500;
      this.carCircle_1500 = circle_1500;
    },
    //number类型半径，单位m
    changeCricleRadius(radius) {
      this.carCircle.setRadius(radius);
    },
    changeCarAndCricleLatLng(lat, lng) {
      this.map.panTo({ lat: lat, lng: lng });
      this.carCircle.setLatLng({ lat: lat, lng: lng });
      this.carCircle_500.setLatLng({ lat: lat, lng: lng });
      this.carCircle_1500.setLatLng({ lat: lat, lng: lng });
      this.carDom.setLatLng({ lat: lat, lng: lng });

      let semiCircleLayers = this.semiCircleLayers.getLayers(); //以下为为报警扇形区域重置经纬度
      semiCircleLayers &&
        semiCircleLayers.forEach((item) => {
          item.setLatLng({ lat: lat, lng: lng });
        });
    },
    drowSemiCircle(item) {
      let semiCircleDom = L.semiCircle([this.devLat, this.devLng], {
        radius: 5000,
        // startAngle: 45,
        // stopAngle: 135,
        weight: 0.8,
        color: "red",
        opacity: 1,
        sign: item.id,
      })
        .setDirection(item.azimuth, 8)
        .addTo(this.semiCircleLayers);
      // semiCircleDom.setDirection(90, 10);
      return semiCircleDom;
    },
    drawCar() {
      let carIcon = L.icon({
        iconUrl: this.carImg,
        iconSize: [50, 50],
        iconAnchor: [25, 25],
        // popupAnchor: [0, 0],
        // shadowSize: [25, 25],
        // shadowAnchor: [25, 25],
      });
      let carDom = L.marker([this.devLat, this.devLng], {
        icon: carIcon,
        rotationAngle: this.carCourse,
      });
      this.carLayers.addLayer(carDom);
      // debugger
      return carDom;
    },

    drawUAV(item) {
      let UAVIcon = L.icon({
        iconUrl: item.isWhite ? this.UAVImg_b : this.UAVImg_h,
        iconSize: [14, 14],
        iconAnchor: [7, 7],
        popupAnchor: [5, -20],
        // shadowSize: [25, 25],
        // shadowAnchor: [25, 25],
      });
      let UAVDom = L.marker([item.latitude, item.longitude], {
        icon: UAVIcon,
        alt: item.id,
        title: item.id,
        sign: item.id,
      });
      UAVDom.addTo(this.UAVLayers).bindPopup(
        `<div class="m-plane-popup-content">
          <div><span>ID: </span><span>${item.id}</span></div>
          <div><span>频段: </span><span>${item.frequency}</span></div>
          <div><span>类型: </span><span>${item.model}</span></div>
        </div>`,
        {
          className: "m-plane-popup-class",
          closeButton: true,
          closeOnClick: false,
          autoPan: false,
        }
      );
      // .openPopup();

      // this.UAVLayers_h.addLayer(UAVDom);
      UAVDom.addEventListener("click", function (ev) {
        console.log(ev);
      });
      return UAVDom;
    },
    drawUAVList() {
      let find = this.uavList.find((item) => {
        return !item.isWhite;
      });
      if (find) {
        this.$refs.audio.muted = false;
      } else {
        this.$refs.audio.muted = true;
      }

      let UAVLayersArr = this.UAVLayers.getLayers();
      let semiCircleLayersArr = this.semiCircleLayers.getLayers();

      UAVLayersArr.forEach((item) => {
        let sign = item.options.sign;
        let find = this.uavList.find((zitem) => {
          return zitem.id == sign;
        });
        if (find) {
          item.setLatLng({ lat: find.latitude, lng: find.longitude });
        } else {
          this.UAVLayers.removeLayer(item);
        }
      }); //去掉消失无人机，并更新移动无人机位置
      semiCircleLayersArr.forEach((item) => {
        let sign = item.options.sign;
        let find = this.uavList.find((zitem) => {
          return zitem.id == sign;
        });
        if (find) {
          item.setDirection(find.azimuth, 8);
        } else {
          this.semiCircleLayers.removeLayer(item);
        }
      }); //去掉消失的警告方向，并更新警告方向
      // debugger
      this.uavList.forEach((item) => {
        let find = semiCircleLayersArr.find((zitem) => {
          return zitem.options.sign == item.id;
        });
        if (!find) {
          this.drawUAV(item);
          !item.isWhite && this.drowSemiCircle(item);
          // !item.isWhite && this.voicePolice();
        }
      }); //画出新增的警告方向和无人机方位
    },
    voicePolice() {
      //下面为声音报警
      this.$refs.audio.muted = false;
      this.timeOut && clearTimeout(this.timeOut);
      this.timeOut = setTimeout(() => {
        this.$refs.audio.muted = true;
        this.timeOut = null;
      }, 15000);
    },
  },
  created() {
    this.$nextTick(() => {
      this.init();

      this.map.addLayer(this.carLayers); //加入车载图层
      this.map.addLayer(this.UAVLayers); //加入无人机图层
      this.map.addLayer(this.setLayers); //加入设置点图层
      this.map.addLayer(this.semiCircleLayers); //加入扇形图层

      this.drawCricle();
      this.carDom = this.drawCar();

      this.drawUAVList(); //无人机画出来，还有报警方位

      this.map.addEventListener("click", (ev) => {
        let lat = ev.latlng.lat;
        let lng = ev.latlng.lng;
        console.log(lat, lng, ev);
        // L.circle([lat, lng], {
        //   radius: 30,
        //   color: "blue",
        //   fillColor: "blue",
        //   fillOpacity: 0.3,
        // }).addTo(this.setLayers);
      });
    });
  },
  mounted() {
    // let that = this;
    // setTimeout(()=>{
    //   debugger
    //   that.$refs.audio.play()
    // },3000)
  },
  updated() {},
  destroyed() {},
};
</script>

<style lang='less' scoped>
/deep/ .m-plane-popup-class {
  margin: 10px 5px 5px;
}
/deep/ .m-plane-popup-content {
  font-size: 14px;
  text-align: left;
  & > div {
    display: flex;
    align-items: center;
    justify-content: space-between;
    min-width: 80px;
  }
}
.audio {
  position: absolute;
  bottom: 3px;
  right: 3px;
  z-index: 10000;
}
.icon-btn {
  z-index: 10000;
  position: absolute;
  right: 3px;
  bottom: 3px;
  height: 40px;
  width: 40px;
  text-align: center;
  line-height: 40px;
  border-radius: 50%;
  box-shadow: inset 0 0 15px var(--box-shandow);
  background: var(--q-bacground);
  z-index: 9999;
  cursor: pointer;
}
.play-class {
  color: var(--box-shandow);
}
</style>