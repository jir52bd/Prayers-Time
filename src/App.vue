<script setup>
import { onMounted, ref} from 'vue';

const cityName = ref('');
const prayerTimes = ref([]);
console.log(prayerTimes);


//AIP intregrating 

const getPrayerTimes = async () =>{
  try{
      const response = await fetch('https://api.aladhan.com/v1/calendar/2017/4?latitude=51.508515&longitude=-0.1254872&method=2');
      
      const data = await response.json();
      const timings = data.data[0].timings;

      console.log(timings);
      cityName.value = data.data[0].meta.timezone;
      prayerTimes.value = [timings.Fajr, timings.Dhuhr, timings.Asr, timings.Maghrib, timings.Isha];

      
  }catch(error){
    console.log('Error fetching error', error);

  }
};

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

const getPrayerName = (index) => {
  const prayerName = ['Fajr', 'Dhuhr', 'Asr', 'Maghrib', 'Isha'];
  return prayerName[index];
}


onMounted(() => {
  getPrayerTimes();
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