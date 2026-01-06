<!--
  IntroPage.vue - Loader iniziale

  Mostra un contatore percentuale che va da 0 a 100
  Al termine, naviga automaticamente verso HeroPage
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

  // Timeline principale
  const tl = gsap.timeline()

  // Animazione contatore da 0 a 100
  tl.to(counter, {
    value: 100,
    duration: 2.5, // Durata del conteggio
    ease: 'power3.inOut',
    onUpdate: () => {
      // Aggiorna il testo con il valore arrotondato
      if (percentageRef.value) {
        percentageRef.value.textContent = Math.round(counter.value)
      }
    },
  })

  // Animazione di uscita del numero (slide up + fade)
  tl.to(
    percentageRef.value,
    {
      y: '-100%',
      opacity: 0,
      duration: 0.8,
      ease: 'power3.inOut',
    },
    '+=0.3', // Piccola pausa dopo il 100
  )

  // Fade out del loader
  tl.to(
    loaderRef.value,
    {
      opacity: 0,
      duration: 0.5,
      ease: 'power2.out',
      onComplete: () => {
        // Naviga verso HeroPage
        emit('navigate', 'hero')
      },
    },
    '-=0.3',
  )
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
