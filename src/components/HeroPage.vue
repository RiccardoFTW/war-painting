<template>
  <div class="hero__container">
    <nav class="navbar">
      <p>Experimental Project</p>
      <p>©2025</p>
    </nav>
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
  <div class="blum-overlay"></div>
  <div class="blum-overlay"></div>
  <div class="blum-overlay"></div>
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
import { onMounted, onUnmounted, computed, ref, nextTick } from 'vue'
import gsap from 'gsap'
import { SplitText } from 'gsap/all'
import paintingsData from '../data/paintings.json'

gsap.registerPlugin(SplitText)

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

const imgNum = ref(0)
const isCounting = ref(true)
const startFromX = ref(0)
const startFromY = ref(0)
const allImages = ref([])
const hasNavigated = ref(false)

const emit = defineEmits(['navigate'])

const handleScroll = (e) => {
  if (hasNavigated.value) return

  const deltaY = e.deltaY
  if (deltaY > 0) {
    hasNavigated.value = true
    emit('navigate', 'main')
  }
}

// ===== HELPERS RESPONSIVE =====
const isMobile = () => window.innerWidth <= 768
const isTablet = () => window.innerWidth > 768 && window.innerWidth <= 1024

const getThreshold = () => {
  if (isMobile()) return 80
  if (isTablet()) return 120
  return 150
}

const getImageScale = () => {
  if (isMobile()) return 2
  if (isTablet()) return 2
  return 2.5
}

const handleMouseMove = (e) => {
  const x = e.clientX
  const y = e.clientY
  const threshold = getThreshold()

  const hasCrossedThreshold =
    x > startFromX.value + threshold ||
    x < startFromX.value - threshold ||
    y > startFromY.value + threshold ||
    y < startFromY.value - threshold

  if (hasCrossedThreshold) {
    showNextImage(e)
    isCounting.value = true
  }

  if (isCounting.value) {
    startFromX.value = x
    startFromY.value = y
  }

  isCounting.value = false
}

const showNextImage = () => {
  if (!allImages.value.length) return

  gsap.killTweensOf('.trail-image')
  allImages.value.forEach((img) => {
    gsap.set(img, {
      opacity: 0,
    })
  })

  const movingImage = allImages.value[imgNum.value]
  const centerX = window.innerWidth / 2
  const centerY = window.innerHeight / 2

  gsap.set(movingImage, {
    left: centerX,
    top: centerY,
    opacity: 0,
  })

  const tl = gsap.timeline()

  tl.to(movingImage, {
    opacity: 1,
    duration: 0.6,
    scale: getImageScale(),
    ease: 'power3.out',
  })

  tl.to(
    movingImage,
    {
      opacity: 1,
      scale: getImageScale(),
      duration: 1.5,
      ease: 'power1.out',
    },
    '-=0.1',
  )

  tl.to(movingImage, {
    opacity: 0,
    duration: 0.8,
    ease: 'power1.inOut',
  })

  imgNum.value++
  if (imgNum.value >= allImages.value.length) {
    imgNum.value = 0
  }
}

// ===== TEXT ANIMATIONS =====
const animateTexts = async () => {
  await nextTick()

  const timeline = gsap.timeline()
  const splits = []
  // NAVBAR
  const navbarTexts = document.querySelectorAll('.navbar p')
  navbarTexts.forEach((text, index) => {
    gsap.set(text, { opacity: 1 })

    const split = new SplitText(text, {
      type: 'chars',
      charsClass: 'char',
    })
    splits.push(split)

    timeline.fromTo(
      split.chars,
      {
        y: '120%',
        opacity: 0,
      },
      {
        y: 0,
        opacity: 1,
        duration: 1,
        ease: 'power3.in',
        stagger: 0.06,
      },
      index * 0.1,
    )
  })
  // FOOTER
  const footerTexts = document.querySelectorAll('.footer p')
  footerTexts.forEach((text, index) => {
    gsap.set(text, { opacity: 1 })

    const split = new SplitText(text, {
      type: 'chars',
      charsClass: 'char',
    })
    splits.push(split)

    timeline.fromTo(
      split.chars,
      {
        y: '120%',
        opacity: 0,
      },
      {
        y: 0,
        opacity: 1,
        duration: 1,
        ease: 'power4.out',
        stagger: 0.02,
      },
      navbarTexts.length * 0.1 + index * 0.3,
    )
  })
}

// ===== LIFECYCLE HOOKS =====
onMounted(async () => {
  await nextTick()
  const imgWrapper = document.querySelector('.img-wrapper')
  if (imgWrapper) {
    allImages.value = [...imgWrapper.querySelectorAll('.trail-image')]
  }

  document.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('wheel', handleScroll, { passive: false })
  animateTexts()
})

onUnmounted(() => {
  document.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('wheel', handleScroll)
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

.navbar {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding: 0 0.8rem;
  pointer-events: auto;
}

.navbar p {
  margin: 0;
  font-family: 'EB Garamond', ' Serif';
  font-style: italic;
  font-weight: 300;
  font-size: clamp(0.875rem, 1vw, 1rem);
  line-height: 1.5;
  text-align: center;
  opacity: 0;

  @media (max-width: 768px) {
    margin: 0;
    font-size: clamp(0.8rem, 1vw, 0.8rem);
  }
}

.hero__title {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 0 0.8rem;
  width: 100%;
  margin-top: auto;
  margin-bottom: auto;
  pointer-events: auto;
  color: #c49852;

  @media (max-width: 768px) {
    gap: 1rem;
  }
}

.title__container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 3rem;

  @media (max-width: 768px) {
    gap: 0.5rem;
  }
}

.hero__title h1 {
  font-family: 'EB Garamond', 'Serif';
  font-size: clamp(2.5rem, 5vw, 4rem);
  font-weight: 300;
  line-height: 0.8;
  text-transform: uppercase;
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
  font-family: 'EB Garamond', 'Serif';
  font-style: italic;
  font-weight: 300;
  font-size: clamp(0.875rem, 1vw, 1rem);
  line-height: 1.5;
  text-align: center;
  text-transform: none;
  opacity: 0;

  @media (max-width: 768px) {
    font-size: clamp(0.8rem, 1vw, 0.8rem);
  }
}

.footer a {
  color: #c49852;
}

.blum-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-image: url('../assets/img/blum.png');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  pointer-events: none;
  z-index: 7;
  opacity: 0.5;
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

.char,
.line {
  transform: translate3d(0, 100%, 0);
}
</style>
