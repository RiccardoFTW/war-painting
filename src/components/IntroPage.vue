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
  font-size: clamp(3rem, 15vw, 128px);
  color: #5d3136;

  @media (max-width: 768px) {
    font-size: clamp(2.5rem, 12vw, 4rem);
  }
}

span {
  display: block;
  overflow: hidden;
  padding: 1vw;
  margin: 0 !important;

  @media (max-width: 768px) {
    padding: 0.5rem;
  }
}

.emotions__container {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 7rem;
  font-family: 'Neue Montreal';
  color: #5d3136;

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

.description__text {
  font-family: 'Instrument Serif', serif;
  font-size: clamp(1rem, 2vw, 1.3rem);
  color: #5d3136;
  text-align: center;
  max-width: 1000px;
  text-transform: uppercase;

  @media (max-width: 1024px) {
    max-width: 800px;
    padding: 0 2rem;
  }

  @media (max-width: 768px) {
    max-width: 100%;
    padding: 0 1.5rem;
  }
}
</style>
