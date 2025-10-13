<template>
  <div class="hero__container">
    <div class="hero__title">
      <h1>WAR PAINTINGS</h1>
    </div>
    <div class="emotions__container">
      <p>
        <span>
          <span>GLORIOUS</span>
        </span>
      </p>
      <p>
        <span>
          <span>APOCALYPTIC</span>
        </span>
      </p>
      <p>
        <span>
          <span>BRUTAL</span>
        </span>
      </p>
      <p>
        <span>
          <span>HEROIC</span>
        </span>
      </p>
      <p>
        <span>
          <span>VISCERAL</span>
        </span>
      </p>
      <p>
        <span>
          <span>FIERCE</span>
        </span>
      </p>
    </div>
    <div class="about__container">
      <p>a collection of works to tell the story of the war over the years</p>
    </div>
  </div>

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
import { onMounted, onUnmounted, computed } from 'vue'
import gsap from 'gsap'
import paintingsData from '../data/paintings.json'

const images = computed(() => {
  const shuffled = [...paintingsData.paintings].sort(() => Math.random() - 0.5)
  return shuffled.slice(0, 25).map((painting) => {
    const imageName = painting.image.split('/').pop()
    return {
      src: new URL(`../assets/img/paintings/${imageName}`, import.meta.url).href,
      alt: painting.title,
    }
  })
})

let imgNum = 0
let lastPosX = 0
let lastPosY = 0
let isCounting = true
let startFromX = 0
let startFromY = 0
let allImages = []

// RESPONSIVE HELPERS
const isMobile = () => window.innerWidth <= 768
const isTablet = () => window.innerWidth > 768 && window.innerWidth <= 1024

const getThreshold = () => {
  if (isMobile()) return 80
  if (isTablet()) return 120
  return 150
}

const getImageScale = () => {
  if (isMobile()) return 1.2
  if (isTablet()) return 1.3
  return 1.4
}

const handleMouseMove = (e) => {
  const x = e.clientX
  const y = e.clientY
  const threshold = getThreshold()

  const hasCrossedThreshold =
    x > startFromX + threshold ||
    x < startFromX - threshold ||
    y > startFromY + threshold ||
    y < startFromY - threshold

  if (hasCrossedThreshold) {
    showNextImage(e)
    isCounting = true
  }

  if (isCounting) {
    startFromX = x
    startFromY = y
  }

  isCounting = false
}

const showNextImage = (e) => {
  if (!allImages.length) return

  const movingImage = allImages[imgNum]
  const curPosX = e.clientX
  const curPosY = e.clientY

  gsap.set(movingImage, {
    left: curPosX,
    top: curPosY,
    opacity: 0,
  })

  const movingDistanceX = ((curPosX - lastPosX) * 100) / 100
  const movingDistanceY = ((curPosY - lastPosY) * 100) / 100

  const tl = gsap.timeline()

  tl.to(movingImage, {
    opacity: 0.8,
    duration: 0.6,
    scale: getImageScale(),
    ease: 'power3.out',
  })

  tl.to(
    movingImage,
    {
      left: lastPosX + movingDistanceX,
      top: lastPosY + movingDistanceY,
      duration: 1.5,
      ease: 'power1.out',
    },
    '-=0.4',
  )

  tl.to(
    movingImage,
    {
      opacity: 0,
      duration: 0.8,
      ease: 'power1.inOut',
    },
    '+=0.3',
  )

  imgNum++
  if (imgNum >= allImages.length) {
    imgNum = 0
  }

  lastPosX = curPosX
  lastPosY = curPosY
}

onMounted(() => {
  setTimeout(() => {
    const imgWrapper = document.querySelector('.img-wrapper')
    if (imgWrapper) {
      allImages = [...imgWrapper.querySelectorAll('.trail-image')]
    }
  }, 80)

  document.addEventListener('mousemove', handleMouseMove)
})

onUnmounted(() => {
  document.removeEventListener('mousemove', handleMouseMove)
  gsap.killTweensOf('.trail-image')
})
</script>

<style scoped>
.hero__container {
  position: fixed;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  position: relative;
  z-index: 10;
}

.hero__container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url(../assets/img/intro-img/6.svg);
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  opacity: 0.08;
  z-index: -1;
  pointer-events: none;
}

.hero__title {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0.5rem 0;
}

.hero__title h1 {
  font-family: 'Instrument Serif', serif;
  font-size: clamp(1rem, 23vw, 15rem);
  font-weight: 400;
  color: blue;
  line-height: 0.8;
  letter-spacing: 0.02em;
  text-transform: uppercase;
}

.emotions__container {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 7rem;
  font-family: 'Neue Montreal';
  color: blue;
  font-size: clamp(0.8rem, 1.2vw, 1rem);

  @media (max-width: 1024px) {
    grid-template-columns: repeat(3, 1fr);
    gap: 4rem;
  }

  @media (max-width: 768px) {
    grid-template-columns: repeat(2, 1fr);
    gap: 2rem;
    padding: 0 1rem;
  }
}

.about__container {
  font-family: 'Neue Montreal', serif;
  font-size: clamp(1rem, 2vw, 1.5rem);
  color: blue;
  text-align: center;
  padding: 1rem;
  text-transform: uppercase;
}

.img-wrapper {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  pointer-events: none;
  z-index: 5;
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
</style>
