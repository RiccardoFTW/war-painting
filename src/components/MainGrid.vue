<script setup>
import { ref, defineExpose } from 'vue'
import MainColumn from './MainColumn.vue'
import gsap from 'gsap'

const gridRef = ref(null)

defineExpose({
  gridRef,
})
</script>

<template>
  <div class="grid" ref="gridRef">
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
