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
import PaintingImg from './PaintingImg.vue'

const props = defineProps({
  paintings: {
    type: Array,
    required: true,
  },
})

const emit = defineEmits(['painting-click'])

const getImagePath = (imagePath) => {
  if (!imagePath) return ''

  try {
    // Estrai il nome del file dal path (stesso metodo usato in HeroPage.vue)
    const imageName = imagePath.split('/').pop()
    // Usa new URL con il path corretto relativo a import.meta.url
    // Questo è lo stesso metodo che funziona in HeroPage.vue
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
}

.column:nth-child(even) {
  margin-top: 10vw;
}
</style>
