<template>
  <div class="intro__container">
    <h1 class="war">
      <span>
        <span>WAR</span>
      </span>
    </h1>
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
</template>

<script setup>
import { onMounted, nextTick } from 'vue'
import gsap from 'gsap'

onMounted(async () => {
  await nextTick()

  const tl = gsap.timeline()

  tl.from('h1 span>span', {
    duration: 3,
    y: 200,
    ease: 'Power3.out',
    stagger: 1.5,
  })
    .from(
      '.description__text span>span',
      {
        duration: 2,
        y: 50,
        autoAlpha: 0,
        ease: 'Power3.out',
      },
      '+=0.1',
    )

    .from(
      '.emotions__container p span>span',
      {
        duration: 2,
        y: 50,
        autoAlpha: 0,
        ease: 'Power3.out',
        stagger: 0.8,
      },
      '+=0.8',
    )
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Instrument+Serif:ital@0;1&display=swap');

@font-face {
  font-family: 'Neue Montreal';
  src: url(../assets/fonts/NeueMontreal-Regular.otf);
}

.top__voices {
  display: flex;
  justify-content: space-between;
  margin: 0;
  padding: 0;
  height: 20px;
}

.top__voices span {
  font-family: 'Neue Montreal', serif;
  color: #5d3136;
  padding: 0 !important;
  font-size: clamp(0.5rem, 1vw, 1rem);
}

.intro__container {
  position: fixed;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  position: relative;
}

.intro__container::before {
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
  opacity: 0.05;
  z-index: -1;
  pointer-events: none;
}

.war {
  font-family: 'Instrument Serif', serif;
  font-weight: 400;
  font-size: 128px;
  color: #5d3136;
}

span {
  display: block;
  overflow: hidden;
  padding: 1vw;
  margin: 0 !important;
}

.emotions__container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr;
  gap: 7rem;
  font-family: 'Neue Montreal';
  color: #5d3136;
}

.description__text {
  font-family: 'Instrument Serif', serif;
  font-size: clamp(1rem, 2vw, 1.3rem);
  color: #5d3136;
  text-align: center;
  max-width: 1000px;
  text-transform: uppercase;
}
</style>
