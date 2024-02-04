

<script setup lang="ts">
const cookie = useCookie("city");
const serach = ref(cookie.value);
const input = ref("");
const background = ref("");
// const { data: city, error } = useFetch(
//   () =>
//     `https://api.openweathermap.org/data/2.5/weather?q=${serach.value}&units=metric&appid=614babe6fbda713d7be2f750f1e1ff18`
// );
let today = new Date()
today = today.toLocaleDateString("en-Us",{
  weekday : 'long',
  year : 'numeric',
  month : 'long',
  day : 'numeric'
})
const { data: city, error } = useAsyncData(
  "city",
  async () => {
    let response;
    try {
      response = await $fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${serach.value}`,{
          params :{
            units : "metric",
            appid: "614babe6fbda713d7be2f750f1e1ff18"
          }
        }
      );
      const temp = response.main.temp;
      if (temp <= -10) background.value = "/snow.jpg";
      else if (temp > -10 && temp <= 0) background.value = "/download.jpg";
      else if (temp > 0 && temp <= 10) background.value = "/night.jpg";
      else background.value = "/sunny.jpg";
      cookie.value = serach.value
    } catch (error) {
        console.log(error);
        
    }
    return response;
  },
  {
    watch: [serach],
  }
);
const handleClick = () => {
  const formated = input.value.trim().split(" ").join("+");
  serach.value = formated;
  input.value = "";
};
const goBakc = ()=>{
  serach.value = cookie.value
}
</script>

<template>
  <div>
    <div v-if="city" class="h-screen relative overflow-hidden">
      <img :src="background" alt="" class="h-screen w-screen" />
      <div class="absolute w-full h-full top-0 overlay" />
      <div class="absolute w-full h-full top-0 p-48">
        <div class="flex justify-between">
          <div>
            <h1 class="text-7xl text-white">{{ city.name }}</h1>
            <p class="font-extralight text-2xl mt-2 text-white">
              {{today}}
            </p>
            <img
              :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@4x.png`"
              alt=""
              class="w-56 icon"
            />
          </div>
          <div>
            <p class="text-9xl text-white font-thin">
              {{ city.main.temp }}°
            </p>
          </div>
        </div>
        <div class="flex">
          <input type="text" class="p-3 w-1/3" v-model="input" />
          <button class="bg-blue-500 text-white p-3" @click="handleClick">
            Search
          </button>
        </div>
      </div>
    </div>
    <div v-else class="p-10">
      <h1 class="text-7xl">Oops, we can't find that city</h1>
      <button @click="goBakc" class="mt-5 bg-sky-400 px-10 w-50 text-white h-10"> Go Back</button>
    </div>
  </div>
</template>

<style scoped>
.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}
.icon {
  margin-left: -2.75rem;
  margin-top: -2.75rem;
}
</style>
