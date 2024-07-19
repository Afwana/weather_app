<script setup lang="ts">
import RandomText from './components/RandomText.vue'
import TodaysWeather from './components/TodaysWeather.vue'
import WeatherOnTime from './components/WeatherOnTime.vue'
</script>

<template>
  <main class="flex items-center justify-center">
    <div
      class="bg-image grid grid-cols-1 md:grid-cols-2 gap-5 p-2 md:p-4 rounded-xl w-full lg:w-2/3"
    >
      <TodaysWeather />
      <div class="w-full flex flex-col gap-5">
        <WeatherOnTime />
        <RandomText />
      </div>
    </div>
  </main>
</template>

<style scoped>
.bg-image {
  background-image: url('src/assets/bgimg.webp');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.main-content {
  background: rgba(0, 0, 0, 0.5);
}
</style>
