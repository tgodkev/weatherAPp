<script lang="ts" setup>
// public run time config.
const config = useRuntimeConfig();

// state for the user searched city.
const input = ref("");
const background = ref("")
// composable for cookies

const cookie = useCookie("city");
// piece of state for starting city.
if (!cookie.value) cookie.value = 'Cleveland';
const search = ref(cookie.value);

// nuxt comes with built in composables, for api calls we have useFetch and useAsyncData.
// const {
//   data: city,
//   error
//   //must use template literal if you want to add dynamic values;
//   // use fetch must have a function calling the api to dynamically update.
// } = useFetch(() => `https://api.openweathermap.org/data/2.5/weather?q=${search.value}&units=imperial&appid=`)

// city is to uniquely identify
// refresh to update data in a useAsync call.
const {data: city, error} = useAsyncData("city", async () => {
  let response;
  try {
    response = await $fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${search.value}`,
        {
          // how to shorten fetch url with alot of querys
          params: {
            units: "imperial",
            appid: config.WEATHER_APP_SECRET,
          }
        }
    );
    cookie.value = search.value
    const temp = response.main.temp;
    if (temp <= 20) {
      background.value = "https://images.unsplash.com/photo-1483664852095-d6cc6870702d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80"
    } else if (temp > 20 && temp <= 0) {
      background.value = "https://images.unsplash.com/photo-1476820865390-c52aeebb9891?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80"
    } else if (temp > 32 && temp <= 50) {
      background.value = "https://images.unsplash.com/photo-1560258018-c7db7645254e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=4032&q=80"
    } else {
      background.value = "https://images.unsplash.com/photo-1507525428034-b723cf961d3e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3546&q=80"
    }


  } catch (e) {

  }
  return response
  // alternative to refresh()
}, {
  watch: [search]
})

let today = new Date()
today = today.toLocaleDateString("en-US", {
  weekday: "long",
  year: "numeric",
  month: "long",
  day: "numeric",
})

// function to search user input city

const handleClick = () => {
  // formatting search in case of multiple word citys Los Angeles = Los+Angeles.
  const formatedSearch = input.value.trim().split(" ").join("+")
  search.value = formatedSearch;
  input.value = "";

};

const goBack = () => {
  search.value = cookie.value;
}
</script>

<template>
  <div v-if="city" class="h-screen relative overflow-hidden">
    <img :src="background" alt=""/>
    <div class="absolute w-full h-full top-0 overlay "/>
    <div class="absolute w-full h-full top-0 p-48">
      <div class="flex justify-between">
        <div>
          <h1 class="text-7xl text-white">{{ city.name }} </h1>
          <p class="font-extralight text-2xl mt-2 text-white">{{ today }}</p>
          <!--    how to get openweather map icons-->
          <img :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@4x.png`"
               alt="icon"
               class="w-56 icon"/>
        </div>
        <div>
          <p class="text-9xl text-white font-extralight">
            {{ city.main.temp }}F
          </p>
        </div>
      </div>
      <div class="mt-20 ">
        <input v-model="input" class="w-1/2 h-10" placeholder="Search A City..." type="text">
        <button class="bg-sky-400 w-20 text-white h-10" @click="handleClick">Search</button>
      </div>

    </div>

  </div>
  <div v-else class="p-10">
    <h1 class="text-7xl ">Oops City Not Found.</h1>
    <button class="mt-5 bg-sky-400 px-10 w-50 text-white h-10" @click="goBack">Go Back.</button>
  </div>
</template>


<style scoped>
.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}

.icon {
  margin-left: -2.75rem;
  margin-top: -2.75rem
}

</style>



