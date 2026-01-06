<!--
  App.vue - Componente principale dell'applicazione

  Gestisce il routing tra le tre viste principali:
  - IntroPage: pagina di introduzione con animazione testi
  - HeroPage: homepage con titolo e image trail
  - MainContainer: griglia principale con i dipinti

  Le transizioni tra le viste sono animate con GSAP
-->
<template>
  <div class="app-container">
    <!-- Vista Intro: animazione iniziale con nomi artisti/musei -->
    <div ref="introRef" class="view-container" v-if="currentView === 'intro'">
      <IntroPage @navigate="handleNavigate" />
    </div>

    <!-- Vista Hero: homepage con titolo "War On Canvas" e image trail -->
    <div ref="heroRef" class="view-container" v-if="currentView === 'hero'">
      <HeroPage @navigate="handleNavigate" />
    </div>

    <!-- Vista Main: griglia interattiva con tutti i dipinti -->
    <div ref="mainRef" class="view-container" v-if="currentView === 'main'">
      <MainContainer @navigate="handleNavigate" />
    </div>
  </div>
</template>

<script setup>
// ===== IMPORTS =====
import { ref, nextTick } from 'vue'
import IntroPage from './components/IntroPage.vue'
import HeroPage from './components/HeroPage.vue'
import MainContainer from './components/MainContainer.vue'
import gsap from 'gsap'

// ===== STATO REATTIVO =====
const currentView = ref('intro') // Vista corrente: 'intro' | 'hero' | 'main'
const introRef = ref(null) // Riferimento al container IntroPage
const heroRef = ref(null) // Riferimento al container HeroPage
const mainRef = ref(null) // Riferimento al container MainContainer

// ===== FUNZIONE DI TRANSIZIONE =====
/**
 * Gestisce le transizioni animate tra le viste
 * @param {string} targetView - La vista di destinazione ('intro' | 'hero' | 'main')
 */
const handleNavigate = async (targetView) => {
  // Configurazioni per ogni tipo di transizione
  const transitions = {
    // Da Intro a Hero
    'intro-hero': {
      fromRef: introRef,
      toRef: heroRef,
      fromAnim: { y: -50, ease: 'power2.in' },
      toAnim: { fromY: 50, ease: 'power2.out' },
    },
    // Da Hero a Main
    'hero-main': {
      fromRef: heroRef,
      toRef: mainRef,
      fromAnim: { y: -150, ease: 'power3.in' },
      toAnim: { fromY: 50, ease: 'power2.out' },
    },
    // Da Main a Hero
    'main-hero': {
      fromRef: mainRef,
      toRef: heroRef,
      fromAnim: { y: 150, ease: 'power2.in' },
      toAnim: { fromY: -50, ease: 'power2.out' },
    },
  }

  // Determina quale transizione eseguire
  const transitionKey = `${currentView.value}-${targetView}`
  const config = transitions[transitionKey]

  if (!config) return

  // Anima l'uscita della vista corrente
  const fromEl = config.fromRef.value
  if (fromEl) {
    await gsap.to(fromEl, {
      opacity: 0,
      y: config.fromAnim.y,
      duration: 0.8,
      ease: config.fromAnim.ease,
    })
  }

  // Cambia la vista corrente
  currentView.value = targetView
  await nextTick()

  // Anima l'entrata della nuova vista
  const toEl = config.toRef.value
  if (toEl) {
    gsap.fromTo(
      toEl,
      { opacity: 0, y: config.toAnim.fromY },
      { opacity: 1, y: 0, duration: 0.8, ease: config.toAnim.ease },
    )
  }
}
</script>

<style scoped>
/* Container principale - occupa tutta la viewport */
.app-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

/* Container per ogni vista - posizionamento assoluto per sovrapporre */
.view-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>

<!-- Stili globali -->
<style>
/* Import font EB Garamond - font principale dell'app */
@import url('https://fonts.googleapis.com/css2?family=EB+Garamond:ital,wght@0,400..800;1,400..800&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Oranienbaum&display=swap');

/* Reset CSS base */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* Sfondo dell'applicazione */
body {
  background: #f5f5f5;
}
</style>
