<script setup>
import PaintingImg from './PaintingImg.vue'
const images = [
  { src: new URL('../assets/img/nw-01.png', import.meta.url).href, alt: 'immagine 1' },
  { src: new URL('../assets/img/nw-02.png', import.meta.url).href, alt: 'immagine 2' },
  { src: new URL('../assets/img/nw-03.png', import.meta.url).href, alt: 'immagine 3' },
  { src: new URL('../assets/img/nw-05.png', import.meta.url).href, alt: 'immagine 4' },
  { src: new URL('../assets/img/nw-06.png', import.meta.url).href, alt: 'immagine 5' },
  { src: new URL('../assets/img/nw-07.png', import.meta.url).href, alt: 'immagine 6' },
  { src: new URL('../assets/img/nw-08.png', import.meta.url).href, alt: 'immagine 7' },
  { src: new URL('../assets/img/nw-09.png', import.meta.url).href, alt: 'immagine 8' },
  { src: new URL('../assets/img/nw-10.png', import.meta.url).href, alt: 'immagine 9' },
  { src: new URL('../assets/img/nw-11.png', import.meta.url).href, alt: 'immagine 10' },
  { src: new URL('../assets/img/nw-12.png', import.meta.url).href, alt: 'immagine 11' },
]

const paintingData = [
  { id: 1, title: 'title 1', description: 'description 1' },
  { id: 2, title: 'title 2', description: 'description 2' },
  { id: 3, title: 'title 3', description: 'description 3' },
  { id: 4, title: 'title 4', description: 'description 4' },
  { id: 5, title: 'title 5', description: 'description 5' },
  { id: 6, title: 'title 6', description: 'description 6' },
  { id: 7, title: 'title 7', description: 'description 7' },
  { id: 8, title: 'title 8', description: 'description 8' },
  { id: 9, title: 'title 9', description: 'description 9' },
  { id: 10, title: 'title 10', description: 'description 10' },
  { id: 11, title: 'title 11', description: 'description 11' },
  { id: 12, title: 'title 12', description: 'description 12' },
]

const emit = defineEmits(['painting-click'])

const handlePaintingClick = (paintingData) => {
  emit('painting-click', paintingData)
}
</script>

<template>
  <div class="column">
    <PaintingImg
      v-for="(image, index) in images"
      :key="index"
      :src="image.src"
      :alt="image.alt"
      :painting-id="paintingData[index]?.id"
      :painting-data="paintingData[index]"
      @click="handlePaintingClick"
    />
  </div>
</template>

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
