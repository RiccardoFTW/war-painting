<template>
  <div class="details" :class="{ '--is-showing': isVisible }">
    <div class="details__thumb"></div>

    <div class="details__content">
      <div class="details__title">
        <p :data-title="currentPainting?.id">{{ currentPainting?.title }}</p>
      </div>

      <div class="details__body">
        <p :data-desc="currentPainting?.id" data-text>{{ currentPainting?.description }}</p>
      </div>
    </div>
  </div>

  <div class="cross" @click="closeDetails">
    <span></span>
    <span></span>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue'

const props = defineProps({
  isVisible: Boolean,
  currentPainting: Object,
})

const emit = defineEmits(['close'])

const closeDetails = () => {
  emit('close')
}

const crossRef = ref(null)

defineExpose({
  crossRef,
})
</script>

<style scoped>
.details {
  position: fixed;
  top: 0;
  right: 0;
  width: 50vw;
  height: 100vh;
  background: #000;
  color: white;
  transform: translateX(50vw);
  transition: transform 0.3s ease;
  z-index: 9999;

  display: flex;
  flex-direction: column;
  padding: 2rem;
}

.details.--is-showing {
  transform: translateX(0);
}

.details__thumb {
  width: 100%;
  height: 40vh;
  background: #111;
  margin-bottom: 2rem;
}

.details__title {
  font-size: 2rem;
  font-weight: bold;
  margin-bottom: 1rem;
}

.details__body {
  font-size: 1.2rem;
  line-height: 1.6;
}

.cross {
  position: fixed;
  top: 0;
  left: 0;
  width: 40px;
  height: 40px;
  cursor: pointer;
  z-index: 10000;
  transform: scale(0);
  pointer-events: none;
}

.cross span {
  position: absolute;
  width: 100%;
  height: 2px;
  background: white;
  top: 50%;
  left: 0;
}

.cross span:first-child {
  transform: rotate(45deg);
}

.cross span:last-child {
  transform: rotate(-45deg);
}
</style>
