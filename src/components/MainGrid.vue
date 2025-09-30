<script setup>
import { ref, defineExpose } from 'vue'
import MainColumn from './MainColumn.vue'

const gridRef = ref(null)
const emit = defineEmits(['painting-click'])

const handlePaintingClick = (paintingData) => {
  emit('painting-click', paintingData)
}

defineExpose({
  gridRef,
})
</script>

<template>
  <div class="grid" ref="gridRef">
    <MainColumn @painting-click="handlePaintingClick" />
    <MainColumn @painting-click="handlePaintingClick" />
    <MainColumn @painting-click="handlePaintingClick" />
    <MainColumn @painting-click="handlePaintingClick" />
    <MainColumn @painting-click="handlePaintingClick" />
  </div>
</template>

<style scoped>
.grid {
  position: absolute;
  width: max-content;
  height: max-content;
  display: flex;
  gap: 10vw;
}
</style>
