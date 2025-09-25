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
</script>

<template>
  <div class="column">
    <PaintingImg v-for="(image, index) in images" :key="index" :src="image.src" :alt="image.alt" />
  </div>
</template>

<style scoped>
.column {
  display: flex;
  flex-direction: column;
  gap: 5vw;
}

.column:nth-child(even) {
  margin-top: 10vw;
}
</style>
