<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search /Results -->
    <div
      class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32"
    >
      <!-- Search Input -->

      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input
            v-model="queryIP"
            type="text"
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            placeholder="Search for any IP address or leave umpty to get your ip info"
          />
          <i
            @click="getIpInfo"
            class="h-full flex items-center rounded-tr-md rounded-br-md cursor-pointer bg-black text-white px-4 py-4 fa-solid fa-chevron-right"
          ></i>
        </div>
      </div>

      <!-- IP Info -->
      <IPInfo v-if="ipInfo" v-bind:ipInfo="ipInfo" />
      <!-- map -->
    </div>
    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
import IPInfo from "../components/IPInfo.vue";
import leaflet from "leaflet";
import { onMounted, ref, watch } from "vue";
import axios from "axios";
export default {
  name: "HomeView",
  components: {
    IPInfo,
  },
  setup() {
    // every thins inside this is going to run priority , before everything

    let mymap;
    const queryIP = ref("");
    let ipInfo = ref(null);
    watch(
      () => ipInfo,
      (value) => {
        console.log(value);
      }
    );
    onMounted(() => {
      mymap = leaflet.map("map").setView([42.505, -83.09], 9);
      leaflet
        .tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
          maxZoom: 19,
          attribution:
            '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
        })
        .addTo(mymap);
    });

    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country?apiKey=at_6FhpbMH6s19mANm24lBwSK2MFi8BT&ipAddress=${queryIP.value}`
        );
        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (error) {
        alert(error.message);
      }
    };

    return {
      queryIP,
      ipInfo,
      getIpInfo,
    };
  },
};
</script>
