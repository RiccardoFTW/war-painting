<template>
  <div class="grid" ref="gridRef">
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="`col-1-${index}`"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="`col-2-${index}`"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="`col-3-${index}`"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="`col-4-${index}`"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
  </div>
</template>

<script setup>
// ===== IMPORTS =====
import { ref, defineExpose, computed } from 'vue'
import MainColumn from './MainColumn.vue'

// ===== PROPS =====
const props = defineProps({
  paintings: {
    type: Array,
    required: true,
  },
})

// ===== STATO =====
const gridRef = ref(null)

// ===== EVENTI =====
const emit = defineEmits(['painting-click'])

// ===== COMPUTED =====
const columns = computed(() => {
  const numColumns = 5
  const cols = Array.from({ length: numColumns }, () => [])

  props.paintings.forEach((painting, index) => {
    const columnIndex = index % numColumns
    cols[columnIndex].push(painting)
  })

  return cols
})

// ===== METODI =====
const handlePaintingClick = (clickData) => {
  emit('painting-click', clickData)
}

defineExpose({ gridRef })
</script>

<style scoped>
.grid {
  position: absolute;
  width: max-content;
  height: max-content;
  display: flex;
  gap: 10vw;
  touch-action: pan-x pan-y;

  @media (max-width: 768px) {
    gap: 20vw;
  }
}
</style>
