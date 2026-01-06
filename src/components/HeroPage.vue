<!--
  HeroPage.vue - Homepage principale

  Contiene:
  - Navbar con info progetto
  - Titolo "WAR ON CANVAS" centrato
  - Footer con credits
  - Image trail: immagini che appaiono al centro dello schermo
    - Desktop: attivato dal movimento del mouse
    - Mobile: automatico ogni 1.5 secondi

  Navigazione verso MainContainer:
  - Desktop: scroll down con rotella
  - Mobile: swipe down
-->
<template>
  <div class="hero__container">
    <!-- Titolo principale -->
    <div class="hero__title">
      <div class="title__container">
        <h1 class="title__one">War</h1>
        <h1 class="title__two">On</h1>
        <h1 class="title__three">Canvas</h1>
      </div>
    </div>

    <!-- Footer con credits -->
    <div class="footer">
      <p>Made in Vue + GSAP</p>
      <p>
        Based on this great
        <a
          href="https://tympanus.net/codrops/2025/09/01/recreating-palmers-draggable-product-grid-with-gsap/"
          >article</a
        >
        by <a href="https://www.linkedin.com/in/joffrey-spitzer-08aaa9158/">Jeffrey</a>
      </p>
    </div>
  </div>

  <!-- Container per le immagini dell'image trail -->
  <div class="img-wrapper">
    <img
      v-for="(image, index) in images"
      :key="index"
      :src="image.src"
      :alt="image.alt"
      class="trail-image"
    />
  </div>
</template>

<script setup>
// ===== IMPORTS =====
import { onMounted, onUnmounted, computed, ref, nextTick } from 'vue'
import gsap from 'gsap'
import { SplitText } from 'gsap/all'
import paintingsData from '../data/paintings.json'

gsap.registerPlugin(SplitText)

// ===== COMPUTED: IMMAGINI PER L'IMAGE TRAIL =====
/**
 * Genera un array di immagini randomizzate per l'effetto trail
 * Usa import.meta.url per risolvere correttamente i path in produzione
 */
const images = computed(() => {
  const shuffled = [...paintingsData.paintings].sort(() => Math.random() - 0.5)
  return shuffled.slice(0, 40).map((painting) => {
    const imageName = painting.image.split('/').pop()
    return {
      src: new URL(`../assets/img/paintings/${imageName}`, import.meta.url).href,
      alt: painting.title,
    }
  })
})

// ===== STATO REATTIVO =====
const imgNum = ref(0) // Indice dell'immagine corrente nell'image trail
const startFromX = ref(0) // Posizione X di partenza per calcolare il movimento
const startFromY = ref(0) // Posizione Y di partenza per calcolare il movimento
const allImages = ref([]) // Array di elementi DOM delle immagini
const hasNavigated = ref(false) // Previene navigazioni multiple
const autoImageInterval = ref(null) // Intervallo per l'auto-trail su mobile

// ===== EVENTI =====
const emit = defineEmits(['navigate'])

// ===== HELPERS RESPONSIVE =====
const isMobile = () => window.innerWidth <= 768
const isTablet = () => window.innerWidth > 768 && window.innerWidth <= 1024

/**
 * Soglia di movimento per attivare una nuova immagine
 * Più piccola su mobile per compensare schermi più piccoli
 */
const getThreshold = () => {
  if (isMobile()) return 80
  if (isTablet()) return 120
  return 200
}

/**
 * Scala delle immagini nell'animazione
 */
const getImageScale = () => {
  if (isMobile()) return 2
  if (isTablet()) return 2
  return 3
}

// ===== GESTIONE SCROLL (DESKTOP) =====
/**
 * Gestisce lo scroll con rotella del mouse
 * Scroll down naviga verso MainContainer
 */
const handleScroll = (e) => {
  if (hasNavigated.value) return

  if (e.deltaY > 0) {
    hasNavigated.value = true
    emit('navigate', 'main')
  }
}

// ===== GESTIONE TOUCH (MOBILE) =====
let touchStartY = 0
let touchEndY = 0
let touchStartTime = 0

const handleTouchStart = (e) => {
  if (hasNavigated.value) return
  touchStartY = e.touches[0].clientY
  touchStartTime = Date.now()
}

const handleTouchMove = (e) => {
  if (hasNavigated.value) return
  touchEndY = e.touches[0].clientY
}

/**
 * Gestisce la fine del touch
 * Naviga se lo swipe è verso il basso e supera la soglia
 */
const handleTouchEnd = () => {
  if (hasNavigated.value) return

  const deltaY = touchStartY - touchEndY
  const touchDuration = Date.now() - touchStartTime

  // Naviga se: swipe > 50px OPPURE swipe > 30px e veloce (< 300ms)
  if (deltaY > 50 || (deltaY > 30 && touchDuration < 300)) {
    hasNavigated.value = true
    emit('navigate', 'main')
  }

  // Reset
  touchStartY = 0
  touchEndY = 0
  touchStartTime = 0
}

// ===== IMAGE TRAIL =====
/**
 * Gestisce il movimento del mouse per l'image trail
 * Mostra una nuova immagine quando il mouse si muove oltre la soglia
 */
const handleMouseMove = (e) => {
  const x = e.clientX
  const y = e.clientY
  const threshold = getThreshold()

  // Verifica se il mouse ha superato la soglia in qualsiasi direzione
  const hasCrossedThreshold =
    Math.abs(x - startFromX.value) > threshold || Math.abs(y - startFromY.value) > threshold

  if (hasCrossedThreshold) {
    showNextImage()
    // Aggiorna la posizione di partenza
    startFromX.value = x
    startFromY.value = y
  }
}

/**
 * Mostra l'immagine successiva nell'image trail
 * Animazione smooth: fade in con scale -> fade out con blur
 */
const showNextImage = () => {
  if (!allImages.value.length) return

  // Ferma tutte le animazioni e nascondi immediatamente
  gsap.killTweensOf('.trail-image')
  allImages.value.forEach((img) => gsap.set(img, { opacity: 0 }))

  // Seleziona l'immagine corrente
  const movingImage = allImages.value[imgNum.value]
  const centerX = window.innerWidth / 2
  const centerY = window.innerHeight / 2

  // Posiziona al centro
  gsap.set(movingImage, {
    left: centerX,
    top: centerY,
    opacity: 0,
    scale: getImageScale() * 0.85,
  })

  // Fade in
  gsap.to(movingImage, {
    opacity: 1,
    scale: getImageScale(),
    duration: 0.5,
    ease: 'power3.out',
  })

  // Passa all'immagine successiva (loop circolare)
  imgNum.value = (imgNum.value + 1) % allImages.value.length
}

// ===== ANIMAZIONE TESTI =====
/**
 * Anima navbar e footer con SplitText
 * I caratteri appaiono dal basso con stagger
 */
const animateTexts = async () => {
  await nextTick()

  const timeline = gsap.timeline()

  // Anima testi della navbar
  document.querySelectorAll('.navbar p').forEach((text, index) => {
    gsap.set(text, { opacity: 1 })

    const split = new SplitText(text, { type: 'chars', charsClass: 'char' })

    timeline.fromTo(
      split.chars,
      { y: '120%', opacity: 0 },
      { y: 0, opacity: 1, duration: 1, ease: 'power3.in', stagger: 0.06 },
      index * 0.1,
    )
  })

  // Anima testi del footer
  document.querySelectorAll('.footer p').forEach((text, index) => {
    gsap.set(text, { opacity: 1 })

    const split = new SplitText(text, { type: 'chars', charsClass: 'char' })

    timeline.fromTo(
      split.chars,
      { y: '120%', opacity: 0 },
      { y: 0, opacity: 1, duration: 1, ease: 'power4.out', stagger: 0.02 },
      0.2 + index * 0.3,
    )
  })
}

// ===== AUTO IMAGE TRAIL (MOBILE) =====
/**
 * Su mobile, avvia l'image trail automatico
 * Le immagini cambiano ogni 1.5 secondi
 */
const startAutoImageTrail = () => {
  if (isMobile()) {
    autoImageInterval.value = setInterval(showNextImage, 1500)
  }
}

// ===== LIFECYCLE =====
onMounted(async () => {
  await nextTick()

  // Raccoglie tutti gli elementi immagine
  const imgWrapper = document.querySelector('.img-wrapper')
  if (imgWrapper) {
    allImages.value = [...imgWrapper.querySelectorAll('.trail-image')]
  }

  // Setup in base al dispositivo
  if (isMobile()) {
    // Mobile: image trail automatico + touch scroll
    startAutoImageTrail()
    setTimeout(showNextImage, 1000) // Prima immagine dopo 1s

    window.addEventListener('touchstart', handleTouchStart, { passive: true })
    window.addEventListener('touchmove', handleTouchMove, { passive: true })
    window.addEventListener('touchend', handleTouchEnd, { passive: true })
  } else {
    // Desktop: image trail su movimento mouse
    document.addEventListener('mousemove', handleMouseMove)
  }

  // Scroll con rotella (funziona anche su desktop)
  window.addEventListener('wheel', handleScroll, { passive: false })

  // Anima i testi
  animateTexts()
})

onUnmounted(() => {
  // Cleanup di tutti i listener
  document.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('wheel', handleScroll)

  if (isMobile()) {
    window.removeEventListener('touchstart', handleTouchStart)
    window.removeEventListener('touchmove', handleTouchMove)
    window.removeEventListener('touchend', handleTouchEnd)
  }

  // Ferma l'intervallo automatico
  if (autoImageInterval.value) {
    clearInterval(autoImageInterval.value)
  }

  // Ferma tutte le animazioni delle immagini
  gsap.killTweensOf('.trail-image')
})
</script>

<style scoped>
/* Container principale - fixed per coprire tutta la viewport */
.hero__container {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  z-index: 10;
  pointer-events: none; /* Permette click-through tranne dove specificato */
}

/* ===== TITOLO ===== */
.hero__title {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0 0.8rem;
  width: 100%;
  margin-top: 0.5rem; /* Titolo in alto */
  margin-bottom: auto; /* Spinge il footer in basso */
  pointer-events: auto;
  color: #c49852; /* Oro */

  @media (max-width: 768px) {
    gap: 1rem;
  }
}

.title__container {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start; /* Allinea i titoli in alto */
  gap: 1rem;

  @media (max-width: 768px) {
    gap: 0.5rem;
  }
}

.hero__title h1 {
  font-family: 'Oranienbaum', serif;
  font-size: clamp(8rem, 10vw, 8rem);
  font-weight: 300;
  line-height: 0.8;
  text-transform: uppercase;

  @media (max-width: 768px) {
    font-size: clamp(3rem, 4vw, 3rem);
  }
}

.title__two {
  position: relative;
  top: 30% !important;
  font-size: clamp(2rem, 3vw, 2rem) !important;
}

/* ===== FOOTER ===== */
.footer {
  width: 100%;
  padding: 0 0.8rem;
  pointer-events: auto;
  display: flex;
  justify-content: space-between;
}

.footer p {
  margin: 0;
  font-family: 'EB Garamond', serif;
  font-style: italic;
  font-weight: 300;
  font-size: clamp(0.875rem, 1vw, 1rem);
  line-height: 1.5;
  text-align: center;
  opacity: 0; /* Nascosto inizialmente */

  @media (max-width: 768px) {
    font-size: clamp(0.8rem, 1vw, 0.8rem);
  }
}

.footer a {
  color: #c49852;
}

/* ===== IMAGE TRAIL ===== */
.trail-image {
  position: absolute;
  width: 200px;
  height: auto;
  opacity: 0;
  pointer-events: none;
  transform: translate(-50%, -50%); /* Centra rispetto al punto */

  @media (max-width: 1024px) {
    width: 150px;
  }

  @media (max-width: 768px) {
    width: 100px;
  }
}

/* Stato iniziale per SplitText */
.char {
  transform: translate3d(0, 100%, 0);
}
</style>
