<template>
  <div id="map"></div>
  <button @click="getLocation()">Get Location</button>
</template>

<script setup lang="ts">
import leaflet from "leaflet";
import { onMounted, watchEffect, ref } from "vue";
import { useGeolocation } from "@vueuse/core";

import { userMarker, nearbyMarkers } from "@/stores/mapStore";

const { coords } = useGeolocation();

const latitude = ref(0);
const longitude = ref(0);

let map: leaflet.Map;
let userGeoMarker: leaflet.Marker;

function getLocation() {
  console.log("* Geolocation function called.");
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition((position) => {
      latitude.value = position.coords.latitude;
      longitude.value = position.coords.longitude;

      console.log("* Latitude: " + position.coords.latitude);
      console.log("* Longitude: " + position.coords.longitude);
    });
  } else {
    console.log("Geolocation is not supported by this browser.");
  }
}
onMounted(() => {
  map = leaflet
    .map("map")
    .setView([userMarker.value.latitude, userMarker.value.longitude], 13);

  leaflet
    .tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
      maxZoom: 19,
      attribution:
        '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
    })
    .addTo(map);

  nearbyMarkers.value.forEach(({ latitude, longitude }) => {
    leaflet
      .marker([latitude, longitude])
      .addTo(map)
      .bindPopup(
        `Saved Marker at (<strong>${latitude.toFixed(2)},${longitude.toFixed(
          2
        )}</strong>)`
      );
  });

  map.addEventListener("click", (e) => {
    const { lat: latitude, lng: longitude } = e.latlng;

    leaflet
      .marker([latitude, longitude])
      .addTo(map)
      .bindPopup(
        `Saved Marker at (<strong>${latitude.toFixed(2)},${longitude.toFixed(
          2
        )}</strong>)`
      );

    nearbyMarkers.value.push({ latitude, longitude });
  });
});

watchEffect(() => {
  if (
    coords.value.latitude !== Number.POSITIVE_INFINITY &&
    coords.value.longitude !== Number.POSITIVE_INFINITY
  ) {
    userMarker.value.latitude = coords.value.latitude;
    userMarker.value.longitude = coords.value.longitude;

    if (userGeoMarker) {
      map.removeLayer(userGeoMarker);
    }

    userGeoMarker = leaflet
      .marker([userMarker.value.latitude, userMarker.value.longitude])
      .addTo(map)
      .bindPopup("User Marker");

    map.setView([userMarker.value.latitude, userMarker.value.longitude], 13);

    const el = userGeoMarker.getElement();
    if (el) {
      el.style.filter = "hue-rotate(120deg)";
    }
  }
});
</script>

 <style scoped>
#map {
  width: 100%;
  height: 98vh;
}
</style>