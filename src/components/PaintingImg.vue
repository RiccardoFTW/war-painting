<template>
  <div class="painting">
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
})
</script>
