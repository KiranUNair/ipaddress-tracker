<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-center text-white text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input type="text"
                 v-model="queryIp" 
                 class="flex-1 px-2 py-3 rounded-tl-md rounded-bl-md focus:outline-none"
                 placeholder="Search for any IP address or leave empty to get your ip info" />
          <i class="cursor-pointer 
                    bg-black 
                    text-white 
                    px-4 
                    items-center 
                    rounded-tr-md 
                    rounded-br-md 
                    flex 
                    fas fa-chevron-right"
              @click="getIpInfo"></i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo"/>
    </div>

    <!-- Map -->
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from '@/components/IPInfo';
import leaflet from 'leaflet';
import {onMounted, ref} from 'vue';
import axios from 'axios';

export default {
  name: 'Home',
  components: { 
    IPInfo
  },
  setup(){
    let mymap;
    const queryIp = ref('');
    const ipInfo = ref(null);    
    onMounted(() => {
      mymap = leaflet.map('mapid').setView([19.07283, 72.88261], 11);
      leaflet.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1Ijoia25haXIiLCJhIjoiY2tzdnRpeDRwMHpiNTMxbHN2eDhuZWc3MCJ9.gsS8SG6RJPg5F8ZqtWlefA', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
      id: 'mapbox/streets-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: 'pk.eyJ1Ijoia25haXIiLCJhIjoiY2tzdnRpeDRwMHpiNTMxbHN2eDhuZWc3MCJ9.gsS8SG6RJPg5F8ZqtWlefA'
      }).addTo(mymap);
    });

    const getIpInfo = async () => {
      try {
        const data = await axios.get(`https://geo.ipify.org/api/v1?apiKey=at_zfTv1TLTcJbC5O5Hku8QCIVa00yK2&ipAddress=${queryIp.value}`);
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
      }
      catch(err) {
        alert(err.message);
      }
    };

    return {
      queryIp,
      ipInfo,
      getIpInfo,
    };
    
  },
}
</script>
