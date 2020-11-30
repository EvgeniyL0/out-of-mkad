<template>
  <yandex-map
    :coords="[55.755814, 37.617635]"
    :zoom="10"
    :ymap-class="'map-container'"
    :controls="[]"
    @map-was-initialized="mapInit"
    @click="getDist"
  >
  </yandex-map>
</template>

<script>
import { loadYmap, yandexMap } from "vue-yandex-maps";
import { mapSettings, polygonCoords } from "../assets/constants.js";

export default {
  components: {
    yandexMap,
  },
  data() {
    return {
      ymapsRef: {},
      myMap: {},
      mkadPolygon: {},
      location: {},
      closestPoit: {},
      normalVec: {},
      route: {},
      settings: mapSettings,
      mkadPolygonCoords: polygonCoords,
      locationCoords: [],
    };
  },
  methods: {
    mapInit(map) {
      this.myMap = map;
      this.mkadPolygon = new this.ymapsRef.Polygon(this.mkadPolygonCoords);
      this.myMap.geoObjects.add(this.mkadPolygon);
    },
    getDist(event) {
      this.locationCoords = event.get("coords");
      this.closestPoit = this.mkadPolygon.geometry.getClosest(
        this.locationCoords
      );

      if (this.myMap.geoObjects.getLength() !== 1) {
        this.myMap.geoObjects
          .remove(this.location)
          .remove(this.normalVec)
          .remove(this.route);
      }

      this.location = new this.ymapsRef.Placemark(this.locationCoords);
      this.normalVec = new this.ymapsRef.Polyline([
        this.closestPoit.position,
        this.locationCoords,
      ]);

      this.ymapsRef
        .route([
          { type: "wayPoint", point: this.locationCoords },
          {
            type: "wayPoint",
            point: this.closestPoit.position,
          },
        ])
        .then((route) => {
          this.route = route;
          this.myMap.geoObjects
            .add(this.location)
            .add(this.normalVec)
            .add(this.route);
          this.myMap.balloon.open(this.locationCoords, {
            contentBody: `<h1>Расстояние до МКАД </h1>
                          <p>по прямой: ${(this.closestPoit.distance / 1000).toFixed(2)} км</p>
                          <p>на автомобиле: ${(this.route.getLength() / 1000).toFixed(2)} км</p>`,
          });
        });
    },
  },
  created() {
    loadYmap({ ...mapSettings, debug: true }).then(() => {
      this.ymapsRef = window.ymaps;
    });
  },
};
</script>

<style>
.map-container {
  width: 100%;
  height: 100vh;
}
</style>