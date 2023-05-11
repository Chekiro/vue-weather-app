<script setup>
import { ref } from "vue";
const emit = defineEmits(["searchForCity"]);

const query = ref("");

const handleInput = () => {
  emit("searchForCity", query.value);
  console.log("kliknalem");
  query.value = "";
};
</script>

<template>
  <div class="search">
    <h3>Search for a city</h3>
    <input type="text" @keyup.enter="handleInput" v-model="query" />
  </div>
</template>

<style lang="scss" scoped></style>
