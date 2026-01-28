<!--
  IntroPage.vue - Loader iniziale

  Mostra un contatore percentuale che va da 0 a 100
  Quando arriva a 100, parte la transizione verso HeroPage
-->
<template>
  <div class="loader" ref="loaderRef">
    <div class="loaderPercentage">
      <h1 ref="percentageRef">0</h1>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import gsap from 'gsap'

// ===== EVENTI =====
const emit = defineEmits(['navigate'])

// ===== REFS =====
const loaderRef = ref(null)
const percentageRef = ref(null)

// ===== ANIMAZIONE LOADER =====
const startLoader = () => {
  // Oggetto per animare il valore numerico
  const counter = { value: 0 }

  // Animazione contatore da 0 a 100
  gsap.to(counter, {
    value: 100,
    duration: 2.5,
    ease: 'power3.inOut',
    onUpdate: () => {
      if (percentageRef.value) {
        percentageRef.value.textContent = Math.round(counter.value)
      }
    },
    onComplete: () => {
      // Quando arriva a 100, parte subito la transizione
      emit('navigate', 'hero')
    },
  })
}

// ===== LIFECYCLE =====
onMounted(() => {
  startLoader()
})
</script>

<style scoped>
.loader {
  position: fixed;
  inset: 0;
  height: 100vh;
  width: 100vw;
  display: flex;
  align-items: flex-end;
  justify-content: flex-end;
  padding: 2rem;
  background: #f5f5f5;
  z-index: 9999;
}

.loaderPercentage {
  overflow: hidden;
}

.loaderPercentage h1 {
  font-family: 'Oranienbaum', serif;
  font-size: 20vw;
  font-weight: 400;
  color: #c49852;
  line-height: 0.85;
  margin: 0;
}

/* ===== RESPONSIVE MOBILE ===== */
@media (max-width: 768px) {
  .loader {
    padding: 1.5rem;
  }

  .loaderPercentage h1 {
    font-size: 30vw;
  }
}
</style>
