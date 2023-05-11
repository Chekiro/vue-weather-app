<script setup>
import { ref } from "vue";
import ForecastItem from "./ForecastItem.vue";

const props = defineProps(["threeHoursStream", "loading"]);

const transition = ref(false);

const toggleTransition = () => {
  transition.value = !transition.value;
};
</script>
<template>
  <div class="weather__other-days">
    <div
      class="weather__forecast-day"
      v-for="item in threeHoursStream"
      :key="item.id"
    >
      <transition name="slide">
        <ForecastItem :item="item" v-if="!loading" />
      </transition>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.slide-enter,
.slide-leave-to {
  opacity: 0;
  transform: translateX(-40%);
}

.slide-enter-active {
  transition: opacity 0.5s, transform 1s;
}
.slide-leave-active {
  transition: opacity 0.5s, transform 1s;
}
</style>
