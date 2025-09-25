<script setup>
import MainGrid from './MainGrid.vue'
</script>

<template>
  <div class="main__container">
    <MainGrid />
  </div>
</template>

<style scoped>
.main__container {
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
}
</style>
