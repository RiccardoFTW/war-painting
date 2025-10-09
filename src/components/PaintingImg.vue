<template>
  <div class="painting" ref="paintingRef" :data-id="paintingId" @click="handleClick">
    <img :src="src" :alt="alt" />
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  src: {
    type: String,
    required: true,
  },
  alt: {
    type: String,
    required: true,
  },
  paintingId: {
    type: Number,
    required: true,
  },
  paintingData: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['click'])

const paintingRef = ref(null)

const handleClick = (e) => {
  e.stopPropagation()
  // Passa l'elemento img, non il contenitore
  const imgElement = paintingRef.value?.querySelector('img')
  emit('click', {
    data: props.paintingData,
    element: imgElement || paintingRef.value,
  })
}
</script>

<style scoped>
.painting {
  position: relative;
  width: 18.5vw;
  aspect-ratio: 1 / 1;

  div {
    width: 18.5vw;
    aspect-ratio: 1 / 1;
  }

  img {
    position: absolute;
    width: 100%;
    height: 100%;
    object-fit: contain;
    will-change: transform;
    transition: transform 300ms ease-in;
  }

  :hover {
    transform: scale(1.05);
  }
}
</style>
