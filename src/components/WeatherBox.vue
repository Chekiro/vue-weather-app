<script setup>
import { computed } from "vue";
import WeatherInput from "./WeatherInput.vue";

const props = defineProps(["weatherData", "time", "loading", "bg"]);
const emit = defineEmits(["searchForCity"]);

const searchForCity = (queryy) => {
  emit("searchForCity", queryy);
};

const kmHour = computed(() => {
  return (props.weatherData.wind.speed * 3.6).toFixed(2);
});

const setImage = (name) => {
  const image = new URL(`../assets/images/${name}`, import.meta.url).href;
  return image;
};
</script>

<template>
  <div
    class="weather__box"
    :style="{
      backgroundImage: 'url(' + setImage(props.bg) + ')',
    }"
  >
    <WeatherInput @searchForCity="searchForCity" />
    <transition name="slide">
      <span v-if="!loading">
        <div class="weather__city">
          <div>
            <img
              :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
              alt
            />
            <div class="city">
              {{ `${weatherData.name}, ${weatherData.sys.country}` }}
            </div>
          </div>
          <div class="localtime">{{ time }}</div>
        </div>
        <div>
          <div class="weather">
            {{ weatherData.main.temp.toFixed(1) }}
            <span class="unit degree">&deg;C</span>
          </div>
        </div>
        <div>
          <div class="wind">
            {{ kmHour }}
            <span class="unit">km/h</span>
          </div>
        </div>
        <div class="text-temp">{{ weatherData.weather[0].description }}</div>
      </span>
    </transition>
  </div>
</template>

<style lang="scss" scoped>
.localtime {
  font-size: 16px;
}
.city {
  margin: 5px 0;
}
.text-temp {
  margin-top: 5px;
}
</style>
