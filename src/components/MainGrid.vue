<template>
  <div class="grid" ref="gridRef">
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="index"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="index"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="index"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
    <MainColumn
      v-for="(columnPaintings, index) in columns"
      :key="index"
      :paintings="columnPaintings"
      @painting-click="handlePaintingClick"
    />
  </div>
</template>

<script setup>
import { ref, defineExpose, computed } from 'vue'
import MainColumn from './MainColumn.vue'

const props = defineProps({
  paintings: {
    type: Array,
    required: true,
  },
})

const gridRef = ref(null)
const emit = defineEmits(['painting-click'])

const columns = computed(() => {
  const numColumns = 5
  const cols = Array.from({ length: numColumns }, () => [])

  props.paintings.forEach((painting, index) => {
    const columnIndex = index % numColumns
    cols[columnIndex].push(painting)
  })
  return cols
})

const handlePaintingClick = (clickData) => {
  emit('painting-click', clickData)
}

defineExpose({
  gridRef,
})
</script>

<style scoped>
.grid {
  position: absolute;
  width: max-content;
  height: max-content;
  display: flex;
  gap: 10vw;

  @media (max-width: 1024px) {
    gap: 10vw;
  }

  @media (max-width: 768px) {
    gap: 10vw;
  }
}
</style>
