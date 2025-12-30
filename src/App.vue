<template>
  <div class="app-container">
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
import { ref, nextTick } from 'vue'
import IntroPage from './components/IntroPage.vue'
import HeroPage from './components/HeroPage.vue'
import MainContainer from './components/MainContainer.vue'
import gsap from 'gsap'

const currentView = ref('intro')
const introRef = ref(null)
const heroRef = ref(null)
const mainRef = ref(null)

const handleNavigate = async (view) => {
  // Transizione da IntroPage a HeroPage
  if (view === 'hero' && currentView.value === 'intro') {
    const introEl = introRef.value
    if (introEl) {
      await gsap.to(introEl, {
        opacity: 0,
        y: -50,
        duration: 0.8,
        ease: 'power2.in',
      })
    }

    currentView.value = 'hero'

    await nextTick()

    const heroEl = heroRef.value
    if (heroEl) {
      gsap.fromTo(
        heroEl,
        {
          opacity: 0,
          y: 50,
        },
        {
          opacity: 1,
          y: 0,
          duration: 0.8,
          ease: 'power2.out',
        },
      )
    }
  }
  // Transizione da HeroPage a MainContainer
  else if (view === 'main' && currentView.value === 'hero') {
    const heroEl = heroRef.value
    if (heroEl) {
      await gsap.to(heroEl, {
        opacity: 0,
        y: -50,
        duration: 0.8,
        ease: 'power2.in',
      })
    }

    currentView.value = 'main'

    await nextTick()

    const mainEl = mainRef.value
    if (mainEl) {
      gsap.fromTo(
        mainEl,
        {
          opacity: 0,
          y: 50,
        },
        {
          opacity: 1,
          y: 0,
          duration: 0.8,
          ease: 'power2.out',
        },
      )
    }
  }
  // Transizione da MainContainer a HeroPage
  else if (view === 'hero' && currentView.value === 'main') {
    const mainEl = mainRef.value
    if (mainEl) {
      await gsap.to(mainEl, {
        opacity: 0,
        y: 50,
        duration: 0.8,
        ease: 'power2.in',
      })
    }

    currentView.value = 'hero'

    await nextTick()

    const heroEl = heroRef.value
    if (heroEl) {
      gsap.fromTo(
        heroEl,
        {
          opacity: 0,
          y: -50,
        },
        {
          opacity: 1,
          y: 0,
          duration: 0.8,
          ease: 'power2.out',
        },
      )
    }
  }
}
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
</style>

<style>
@import url('https://fonts.googleapis.com/css2?family=EB+Garamond:ital,wght@0,400..800;1,400..800&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #f5f5f5;
}
</style>
