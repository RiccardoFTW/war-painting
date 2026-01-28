<template>
  <div class="column">
    <PaintingImg
      v-for="painting in paintings"
      :key="painting.id"
      :src="getImagePath(painting.image)"
      :alt="painting.title"
      :painting-id="painting.id"
      :painting-data="painting"
      @click="handlePaintingClick"
    />
  </div>
</template>

<script setup>
// ===== IMPORTS =====
import PaintingImg from './PaintingImg.vue'

// ===== PROPS =====
defineProps({
  paintings: {
    type: Array,
    required: true,
  },
})

// ===== EVENTI =====
const emit = defineEmits(['painting-click'])

// ===== METODI =====
const getImagePath = (imagePath) => {
  if (!imagePath) return ''

  try {
    const imageName = imagePath.split('/').pop()
    return new URL(`../assets/img/paintings/${imageName}`, import.meta.url).href
  } catch (error) {
    console.error('Errore caricamento immagine:', imagePath, error)
    return ''
  }
}

const handlePaintingClick = (clickData) => {
  emit('painting-click', clickData)
}
</script>

<style scoped>
.column {
  display: flex;
  flex-direction: column;
  gap: 10vw;

  @media (max-width: 768px) {
    gap: 25vw;
  }
}


.column:nth-child(even) {
  margin-top: 10vw;

  @media (max-width: 768px) {
    margin-top: 30vw;
  }
}
</style>
