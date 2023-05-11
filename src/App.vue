<script setup>
import { onMounted, ref, computed } from "vue";
import moment from "moment-timezone";
import shortid from "shortid";
import tzlookup from "tz-lookup";
import { weatherApiInstance } from "./api";
import { fetchBackgroundImage } from "./utils";
import Loading from "./components/Loading.vue";
import WeatherBox from "./components/WeatherBox.vue";
import "./assets/style/style.scss";

const loading = ref(true);
const weather = ref("");
const error = ref(false);
const lon = ref(27.142826);
const lat = ref(38.423733);
const code = ref(null);
const backgroundImg = ref("");
const query = ref("");
const threeHoursStream = ref([]);
const cityLocalTime = ref("");
const cityLocalTimeFormatted = ref("");
const timezoneLookup = ref("");

const handleSearch = (queryy) => {
  loading.value = true;
  query.value = queryy;
  fetchWeather();
  fetchForecast();
};

const isDayTime = computed(() => {
  const format = "HH:mm a";
  const time = moment(cityLocalTimeFormatted.value, format);
  const startTime = moment("07:00am", format);
  const endTime = moment("08:00pm", format);

  if (
    (startTime.hour() >= 12 && endTime.hour() <= 12) ||
    endTime.isBefore(startTime)
  ) {
    endTime.add(1, "days");

    if (time.hour() <= 12) {
      time.add(1, "days");
    }
  }

  return time.isBetween(startTime, endTime);
});

const updateHour = () => {
  setInterval(() => {
    getHour();
  }, 60000);
};

const getHour = () => {
  cityLocalTime.value = moment().tz(timezoneLookup.value).format();
  cityLocalTimeFormatted.value = moment()
    .tz(timezoneLookup.value)
    .format("LT dddd");
};

const fetchData = (url) => {
  let parameters;

  if (query.value !== "") {
    parameters = {
      params: {
        q: query.value,
        units: "metric",
        cnt: 10,
      },
    };
  } else {
    parameters = {
      params: {
        lat: lat.value,
        lon: lon.value,
        units: "metric",
        cnt: 10,
      },
    };
  }

  return weatherApiInstance.get(url, parameters);
};

const getBackground = () => {
  const image = fetchBackgroundImage(code.value, isDayTime.value);
  image.then((img) => {
    backgroundImg.value = img;
  });
};

const fetchLocalWeatherAndForecast = () => {
  query.value = "";
  fetchWeather();
  fetchForecast();
};

const fetchForecast = () => {
  fetchData("/forecast")
    .then((res) => {
      const mapList = res.data.list.map((item) => {
        const { main, weather, dt } = item;
        setTimeout(() => {
          loading.value = false;
        }, 100);
        return {
          main,
          weather,
          dt: moment.unix(dt).tz(timezoneLookup.value).format("LT ddd"),
          id: shortid.generate(),
        };
      });

      threeHoursStream.value = mapList;
    })
    .catch((error) => {
      setTimeout(() => {
        error.value = true;
      }, 2000);
    });
};

const fetchWeather = () => {
  fetchData("/weather").then((res) => {
    weather.value = res.data;
    lat.value = res.data.coord.lat;
    lon.value = res.data.coord.lon;
    timezoneLookup.value = tzlookup(lat.value, lon.value);
    getHour();
    code.value = res.data.weather[0].id;
    getBackground();
  });
};

const setImage = (name) => {
  const image = new URL(`./assets/images/${name}`, import.meta.url).href;
  return image;
};

onMounted(() => {
  fetchLocalWeatherAndForecast();
  updateHour();
});
</script>

<template>
  <div class="main-container container-fluid text-center">
    <div
      v-if="backgroundImg"
      class="bg-image"
      :style="{
        backgroundImage: 'url(' + setImage(backgroundImg) + ')',
      }"
    ></div>

    <div class="weather__container">
      <WeatherBox
        :weatherData="weather"
        :time="cityLocalTimeFormatted"
        :loading="loading"
        :bg="backgroundImg"
        @searchForCity="handleSearch"
      />
      <!-- <LoadingVue v-if="loading" :error="error" /> -->
    </div>
  </div>
</template>

<style scoped></style>
