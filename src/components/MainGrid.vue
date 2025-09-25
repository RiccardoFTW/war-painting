<script setup>
import MainColumn from './MainColumn.vue'
</script>

<template>
  <div class="grid">
    <MainColumn />
    <MainColumn />
    <MainColumn />
    <MainColumn />
    <MainColumn />
  </div>
</template>

<style scoped>
.grid {
  position: absolute;
  width: max-content;
  height: max-content;
  display: flex;
  gap: 5vw;
}
</style>
