<template>
  <div class="app-container">
    <div class="transition-overlay" ref="transitionRef"></div>

    <div ref="introRef" class="view-container" v-if="currentView === 'intro'">
      <IntroPage @navigate="handleNavigate" />
    </div>

    <div ref="heroRef" class="view-container" v-if="currentView === 'hero'">
      <HeroPage @navigate="handleNavigate" />
    </div>

    <div ref="mainRef" class="view-container" v-if="currentView === 'main'">
      <MainContainer @navigate="handleNavigate" />
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'
import IntroPage from './components/IntroPage.vue'
import HeroPage from './components/HeroPage.vue'
import MainContainer from './components/MainContainer.vue'
import gsap from 'gsap'

const currentView = ref('intro')
const introRef = ref(null)
const heroRef = ref(null)
const mainRef = ref(null)
const transitionRef = ref(null)

// ===== TRANSIZIONE =====
/**
 * @returns {Promise}
 */
const transitionIn = () => {
  return new Promise((resolve) => {
    const overlay = transitionRef.value
    if (!overlay) {
      resolve()
      return
    }

    gsap.set(overlay, {
      display: 'block',
      yPercent: 100
    })

    gsap.to(overlay, {
      yPercent: 0,
      duration: 0.8,
      ease: 'power3.inOut',
      onComplete: resolve,
    })
  })
}

/**

 * @returns {Promise}
 */
const transitionOut = () => {
  return new Promise((resolve) => {
    const overlay = transitionRef.value
    if (!overlay) {
      resolve()
      return
    }

    gsap.to(overlay, {
      yPercent: -100,
      duration: 0.8,
      ease: 'power3.inOut',
      onComplete: () => {
        gsap.set(overlay, { display: 'none' })
        resolve()
      },
    })
  })
}

const handleNavigate = async (targetView) => {
  if (currentView.value === 'intro' && targetView === 'hero') {
    await transitionIn()

    if (introRef.value) {
      gsap.set(introRef.value, { opacity: 0 })
    }

    currentView.value = targetView
    await nextTick()

    if (heroRef.value) {
      gsap.set(heroRef.value, { opacity: 1 })
    }

    await transitionOut()
  }
  else if (currentView.value === 'hero' && targetView === 'main') {
    await transitionIn()

    if (heroRef.value) {
      gsap.set(heroRef.value, { opacity: 0 })
    }

    currentView.value = targetView
    await nextTick()

    if (mainRef.value) {
      gsap.set(mainRef.value, { opacity: 1 })
    }

    await transitionOut()
  }
  else if (currentView.value === 'main' && targetView === 'hero') {
    await transitionIn()

    if (mainRef.value) {
      gsap.set(mainRef.value, { opacity: 0 })
    }

    currentView.value = targetView
    await nextTick()

    if (heroRef.value) {
      gsap.set(heroRef.value, { opacity: 1 })
    }

    await transitionOut()
  }
}

onMounted(() => {
  if (transitionRef.value) {
    gsap.set(transitionRef.value, {
      display: 'none',
      yPercent: 100
    })
  }
})
</script>

<style scoped>
.app-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.view-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.transition-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 99999;
  background-color: #c49852;
  display: none;
  pointer-events: none;
}
</style>


<style>
@import url('https://fonts.googleapis.com/css2?family=EB+Garamond:ital,wght@0,400..800;1,400..800&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Oranienbaum&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #f5f5f5;
}
</style>
