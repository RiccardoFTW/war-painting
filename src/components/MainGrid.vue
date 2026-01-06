<!--
  MainGrid.vue - Griglia principale dei dipinti

  Organizza i dipinti in colonne verticali.
  I dipinti sono distribuiti ciclicamente tra le colonne.
  La griglia è ripetuta 4 volte per creare un effetto infinito.
-->
<template>
  <div class="grid" ref="gridRef">
    <!-- Griglia ripetuta 4 volte per effetto infinito -->
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
const gridRef = ref(null) // Riferimento DOM per drag/scroll

// ===== EVENTI =====
const emit = defineEmits(['painting-click'])

// ===== COMPUTED =====
/**
 * Distribuisce i dipinti in 5 colonne
 * Ogni dipinto va nella colonna corrispondente al suo indice % 5
 */
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
/** Propaga l'evento click sul dipinto al parent */
const handlePaintingClick = (clickData) => {
  emit('painting-click', clickData)
}

// Espone il riferimento della griglia al parent
defineExpose({ gridRef })
</script>

<style scoped>
/* Griglia flex orizzontale */
.grid {
  position: absolute;
  width: max-content;
  height: max-content;
  display: flex;
  gap: 10vw;
  touch-action: pan-x pan-y; /* Abilita scroll touch */

  @media (max-width: 768px) {
    gap: 20vw; /* Più spazio tra colonne su mobile */
  }
}
</style>
