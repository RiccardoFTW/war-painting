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

defineProps({
  isVisible: {
    type: Boolean,
    default: false,
  },
  currentPainting: {
    type: Object,
    default: null,
  },
})

const emit = defineEmits(['close'])
const crossRef = ref(null)
const detailsRef = ref(null)

const closeDetails = () => {
  emit('close')
}

defineExpose({
  crossRef,
  detailsRef,
})
</script>

<style scoped>
.details {
  position: fixed;
  top: 0;
  right: 0;
  width: 50vw;
  height: 100vh;
  padding: 3vw 4vw;
  background-color: #f1f1f1;
  opacity: 1;
  transform: translate3d(60vw, 0, 0);
  z-index: 9999;
  will-change: transform;

  @media (max-width: 600px) {
    width: 90%;
    height: 100vh;
    padding: 8vw 7vw;

    transform: translate3d(90%, 0, 0);
  }
}

.details__thumb {
  position: relative;
  width: 100%;
  height: 40vh;
  background: #111;
  margin-bottom: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: visible;

  img {
    width: 18.5vw;
    height: 18.5vw;
    object-fit: contain;
  }
}

.details__title {
  margin-bottom: 4vw;
  position: relative;
  display: grid;

  @media (max-width: 600px) {
    margin-bottom: 8vw;
  }

  p {
    grid-area: 1 / -1;
    overflow: hidden;
    font-size: 3vw;

    @media (max-width: 600px) {
      font-size: 2rem;
    }
  }
}

.details__body {
  display: flex;
  flex-wrap: wrap;
  align-items: end;
  gap: 2vw;

  p {
    overflow: hidden;
  }
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
  background: #111;
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
