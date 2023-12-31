<script setup>
import { onMounted, ref} from 'vue';

const cityName = ref('');
const prayerTimes = ref([]);
console.log(prayerTimes);


//AIP intregrating 

const getPrayerTimes = async (latitude, longitude) =>{
  try{
      const response = await fetch(
        `https://api.aladhan.com/v1/calendar?latitude=${latitude}&longitude=${longitude}&method=2`
        );
      
      const data = await response.json();
      const timings = data.data[0].timings;

      console.log(timings);
      //getting time zone according on API
      cityName.value = data.data[0].meta.timezone;
      prayerTimes.value = [timings.Fajr, timings.Dhuhr, timings.Asr, timings.Maghrib, timings.Isha];

      
  }catch(error){
    console.log('Error fetching error', error);

  }
};

//findout latitude and longitude
const getLocation = () => {
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const {latitude, longitude} = position.coords;
        getPrayerTimes(latitude,longitude);
      }, 
      (error) =>{
        console.error('Error getting user location: ', error);
      }
    );

  }
  else {
    console.error('Geolocation is not supported by this browser.');
  }
}

//Formate time in 12 hours clock
const formatTime = (time) => {
  const [hours, minutes] = time.split(':');
  // const formatedHour =  parseInt(hour) % 12 || 12;
  // const period = formatedHour < 12 ? 'AM' : 'PM';
  // return `${formatedHour}.${munite} ${period}`;
  
  const bstTime = new Date(2024, 0, 1, parseInt(hours), parseInt(minutes), 0);
  const formattedTime = bstTime.toLocaleTimeString('en-US', { hour: 'numeric', minute: 'numeric', hour12: true });
  return formattedTime;
};

//Add corresponding Name of the prayers
const getPrayerName = (index) => {
  const prayerName = ['Fajr', 'Dhuhr', 'Asr', 'Maghrib', 'Isha'];
  return prayerName[index];
}


onMounted(() => {
  getLocation();
});


</script>

<template>
  <div>
    <div id="app">
      <h1>Prayer Time App</h1>
        <p>{{ cityName }}</p>
      <div v-if="prayerTimes.length">
       
        <ul>
          <li v-for="(time, index) in prayerTimes" :key="index">
                <span>{{ getPrayerName(index)}}</span> <span>{{ formatTime(time)  }}</span>          
          </li>
        </ul>

      </div>
    </div>
  </div>
</template>

<style scoped>
#app {
  text-align: center;
  margin-top: 20px;
}
ul >li {
  list-style: none;
}
</style>