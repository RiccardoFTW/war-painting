<template>
  <div class="painting" :data-id="paintingId" @click="handleClick">
    <img :src="props.src" :alt="props.alt" />
  </div>
</template>

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

<script setup>
const props = defineProps({
  src: String,
  alt: String,
  paintingId: Number,
  paintingData: Object,
})

const emit = defineEmits(['click'])

const handleClick = (e) => {
  e.stopPropagation()
  console.log('click su painting:', props.paintingData)
  emit('click', props.paintingData)
}
</script>
