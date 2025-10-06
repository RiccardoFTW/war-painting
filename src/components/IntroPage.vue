<template>
  <div class="intro__container">
    <div class="war__title">
      <h1 class="war">
        <span>
          <span>war</span>
        </span>
      </h1>
    </div>

    <div class="illustration__container">
      <img
        v-for="(image, index) in images"
        :key="index"
        :src="image.src"
        :alt="image.alt"
        class="warrior__image"
        :class="{ 'is-visible': index <= currentImageIndex }"
      />
    </div>

    <div class="war__description">
      <p class="description__text">
        <span>
          <span
            >Organized, large-scale,armed conflict between countries or between national, ethnic, or
            other sizeable groups, usually but not always involving active engagement of military
            forces.</span
          >
        </span>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import gsap from 'gsap'

const images = ref([
  { src: '/src/assets/img/intro-img/6.svg', alt: '6' },
  { src: '/src/assets/img/intro-img/7.svg', alt: '7' },
  { src: '/src/assets/img/intro-img/10.svg', alt: '10' },
  { src: '/src/assets/img/intro-img/11.svg', alt: '11' },
  { src: '/src/assets/img/intro-img/12.svg', alt: '12' },
])

const currentImageIndex = ref(-1)

onMounted(async () => {
  await nextTick()

  const tl = gsap.timeline()

  tl.from('h1 span>span', {
    duration: 3,
    y: 150,
    autoAlpha: 0,
    ease: 'Power3.out',
    stagger: 1.5,
  })
    .from(
      'p span>span',
      {
        duration: 2,
        y: 150,
        autoAlpha: 0,
        ease: 'Power3.out',
        stagger: 1.5,
      },
      '+=0.04',
    )

    .call(animateWarriorsSequentially, [], '+=2')
})

const animateWarriorsSequentially = () => {
  const warriorImages = document.querySelectorAll('.warrior__image')

  gsap.set(warriorImages, { opacity: 0, scale: 0.9 })

  // Timeline infinita
  const tl = gsap.timeline({
    repeat: 1,
    repeatDelay: 0.8, // Pausa di 800ms tra cicli
  })

  warriorImages.forEach((img, index) => {
    // Appari
    tl.to(
      img,
      {
        opacity: 1,
        scale: 1.5,
        duration: 0.6,
        ease: 'power3.out',
      },
      index === 0 ? 0 : '+=0.2',
    ) // Prima immagine subito, altre dopo 300ms

      // Scompari
      .to(
        img,
        {
          opacity: 0,
          scale: 0.8,
          duration: 0.3,
        },
        '+=0.3',
      )
  })
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');

.intro__container {
  position: fixed;
  height: 100vh;
  width: 100vw;
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  align-items: center;
  padding: 0 1vw;
  gap: 7rem;
}

.war {
  font-family: 'Bebas Neue', serif;
  font-size: clamp(3rem, 2vw, 8rem);
  font-weight: 700;
  color: firebrick;
  margin: 0;
  line-height: 1.5;
  letter-spacing: -0.02em;
}

span {
  display: block;
  overflow: hidden;
}

.description__text {
  font-family: 'Bebas Neue', serif;
  font-size: clamp(1rem, 0.2vw, 1.3rem);
  line-height: 1.4;
  color: firebrick;
  letter-spacing: -0.02em;
  text-align: right;
  max-width: 300px;
}

.illustration__container {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 30vh;
  width: 30vw;
}

.warrior__image {
  position: absolute;
  max-width: 40%;
  max-height: 100%;
  object-fit: contain;
  opacity: 0;
}

.warrior__image.is-visible {
  opacity: 1;
}
</style>
