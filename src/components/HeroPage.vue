<template>
  <div class="hero__container">
    <div class="hero__title">
      <div class="title__container">
        <h1 class="title__one">War</h1>
        <h1 class="title__two">On</h1>
        <h1 class="title__three">Canvas</h1>
      </div>
    </div>

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
const imgNum = ref(0)
const startFromX = ref(0)
const startFromY = ref(0)
const allImages = ref([])
const hasNavigated = ref(false)
const autoImageInterval = ref(null)

// ===== EVENTI =====
const emit = defineEmits(['navigate'])

// ===== HELPERS RESPONSIVE =====
const isMobile = () => window.innerWidth <= 768
const isTablet = () => window.innerWidth > 768 && window.innerWidth <= 1024

const getThreshold = () => {
  if (isMobile()) return 80
  if (isTablet()) return 120
  return 200
}


const getImageScale = () => {
  if (isMobile()) return 3
  if (isTablet()) return 3
  return 2.5
}

// ===== GESTIONE SCROLL (DESKTOP) =====
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

const handleTouchEnd = () => {
  if (hasNavigated.value) return

  const deltaY = touchStartY - touchEndY
  const touchDuration = Date.now() - touchStartTime


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
const handleMouseMove = (e) => {
  const x = e.clientX
  const y = e.clientY
  const threshold = getThreshold()

  const hasCrossedThreshold =
    Math.abs(x - startFromX.value) > threshold || Math.abs(y - startFromY.value) > threshold

  if (hasCrossedThreshold) {
    showNextImage()
    startFromX.value = x
    startFromY.value = y
  }
}

const showNextImage = () => {
  if (!allImages.value.length) return

  gsap.killTweensOf('.trail-image')
  allImages.value.forEach((img) => gsap.set(img, { opacity: 0 }))

  const movingImage = allImages.value[imgNum.value]
  const centerX = window.innerWidth / 2
  const centerY = window.innerHeight / 2

  gsap.set(movingImage, {
    left: centerX,
    top: centerY,
    opacity: 0,
    scale: getImageScale() * 0.85,
  })


  gsap.to(movingImage, {
    opacity: 1,
    scale: getImageScale(),
    duration: 0.5,
    ease: 'power3.out',
  })

  imgNum.value = (imgNum.value + 1) % allImages.value.length
}

// ===== ANIMAZIONE TESTI =====
const animateTexts = async () => {
  await nextTick()

  const timeline = gsap.timeline()

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
const startAutoImageTrail = () => {
  if (isMobile()) {
    autoImageInterval.value = setInterval(showNextImage, 1500)
  }
}

// ===== LIFECYCLE =====
onMounted(async () => {
  await nextTick()

  const imgWrapper = document.querySelector('.img-wrapper')
  if (imgWrapper) {
    allImages.value = [...imgWrapper.querySelectorAll('.trail-image')]
  }

  // Setup in base al dispositivo
  if (isMobile()) {
    startAutoImageTrail()
    setTimeout(showNextImage, 1000)

    window.addEventListener('touchstart', handleTouchStart, { passive: true })
    window.addEventListener('touchmove', handleTouchMove, { passive: true })
    window.addEventListener('touchend', handleTouchEnd, { passive: true })
  } else {
    document.addEventListener('mousemove', handleMouseMove)
  }
  window.addEventListener('wheel', handleScroll, { passive: false })

  animateTexts()
})

onUnmounted(() => {
  document.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('wheel', handleScroll)

  if (isMobile()) {
    window.removeEventListener('touchstart', handleTouchStart)
    window.removeEventListener('touchmove', handleTouchMove)
    window.removeEventListener('touchend', handleTouchEnd)
  }

  if (autoImageInterval.value) {
    clearInterval(autoImageInterval.value)
  }

  gsap.killTweensOf('.trail-image')
})
</script>

<style scoped>
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
  pointer-events: none;
}

.hero__title {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1rem;
  width: 100%;
  margin-top: 0.5rem;
  margin-bottom: auto;
  pointer-events: auto;
  color: #c49852;

  @media (max-width: 768px) {
    gap: 1rem;
  }
}

.title__container {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
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
  opacity: 0;

  @media (max-width: 768px) {
    font-size: clamp(0.8rem, 1vw, 0.8rem);
  }
}

.footer a {
  color: #c49852;
}

.trail-image {
  position: absolute;
  width: 200px;
  height: auto;
  opacity: 0;
  pointer-events: none;
  transform: translate(-50%, -50%);

  @media (max-width: 1024px) {
    width: 150px;
  }

  @media (max-width: 768px) {
    width: 100px;
  }
}

.char {
  transform: translate3d(0, 100%, 0);
}
</style>
