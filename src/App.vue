<script setup>
import { onMounted, ref} from 'vue';
// import fajr from './assets/prayer_times/fajr.png'; 

// const fajr_img = fajr;

const cityName = ref('');
const prayerTimes = ref([]);
// console.log(prayerTimes);


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
};

//Map prayer names to corresponding image URls
const getPrayerImage = (index) => {
  const prayerNames = ['Fajr', 'Dhuhr', 'Asr', 'Maghrib', 'Isha'];
  const prayerImages = {
    Fajr: require('@/assets/prayer_times/fajr.png'),
    Dhuhr: require('@/assets/prayer_times/zuhr.png'),
    Asr: require('@/assets/prayer_times/asr.png'),
    Maghrib: require('@/assets/prayer_times/maghrib.png'),
    Isha: require('@/assets/prayer_times/isha.png'),
  };
  return prayerImages[prayerNames[index]] || 'https://placehold.co/600x400/orange/white';
};

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
                <img :src="getPrayerImage(index)" alt="Prayer Image" />         
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

img {
  width: 50px; /* Adjust the size as needed */
  height: auto;
  margin-left: 10px; /* Add spacing between text and image */
}
</style>