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
  e.preventDefault()

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
  touch-action: manipulation;
  cursor: pointer;

  @media (max-width: 1024px) {
    width: 25vw;
  }

  @media (max-width: 768px) {
    width: 45vw;
  }
}

.painting img {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: contain;
  will-change: transform;
  transition: transform 300ms ease-in;
}

.painting:hover img {
  transform: scale(1.05);
}
</style>
