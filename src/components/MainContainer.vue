<!--
  MainContainer.vue - Container principale della griglia dipinti

  Funzionalità:
  - Griglia draggabile di dipinti
  - Pannello dettagli con animazione FLIP
  - Scroll smooth con Lenis (desktop) e touch (mobile)
  - Animazione intro con zoom
  - Bottone Home per tornare a HeroPage

  Struttura:
  - MainGrid: griglia di colonne con i dipinti
  - PaintingDetails: pannello laterale con dettagli dipinto
-->
<template>
  <!-- Container principale della griglia -->
  <div class="container" ref="containerRef" @click="handleContainerClick">
    <MainGrid
      ref="mainGridRef"
      :paintings="paintingsData.paintings"
      @painting-click="openDetails"
    />

    <PaintingDetails
      ref="paintingDetailsRef"
      :is-visible="showDetails"
      :current-painting="currentPainting"
      @close="closeDetails"
    />
  </div>

  <!-- Bottone Home (fuori dal container per evitare interferenze) -->
  <div ref="homeButtonRef" class="home-button" @click.stop="goToHero">
    <h2>Home</h2>
  </div>
</template>

<script setup>
// ===== IMPORTS =====
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import MainGrid from './MainGrid.vue'
import PaintingDetails from './PaintingDetails.vue'
import { Draggable, Flip, SplitText } from 'gsap/all'
import gsap from 'gsap'
import paintingsData from '../data/paintings.json'

// Registra i plugin GSAP
gsap.registerPlugin(Draggable, Flip, SplitText)

// ===== EVENTI =====
const emit = defineEmits(['navigate'])

// ===== STATO REATTIVO =====
// Riferimenti DOM
const containerRef = ref(null)
const mainGridRef = ref(null)
const paintingDetailsRef = ref(null)
const homeButtonRef = ref(null)

// Stato UI
const isDragging = ref(false) // Indica se la griglia è in drag
const showDetails = ref(false) // Mostra/nasconde pannello dettagli
const currentPainting = ref(null) // Dipinto selezionato
const isClosing = ref(false) // Animazione chiusura in corso
const isAnimating = ref(false) // Animazione intro in corso

// Stato per FLIP animation
const currentPaintingElement = ref(null)
const originalParent = ref(null)
const originalGridPosition = ref({ x: 0, y: 0 })

// Istanze SplitText (per cleanup)
let titleSplit = null
let descSplit = null

// ===== SMOOTH SCROLL STATE =====
// Variabili per lo smooth scroll custom
let targetX = 0
let targetY = 0
let currentX = 0
let currentY = 0
let rafId = null
const LERP_FACTOR = 0.08 // Fattore di interpolazione (più basso = più smooth)

// ===== HELPERS RESPONSIVE =====
const isMobile = () => window.innerWidth <= 768
const isTablet = () => window.innerWidth > 768 && window.innerWidth <= 1024

/** Larghezza pannello dettagli in base al dispositivo */
const getDetailsPanelWidth = () => {
  if (isMobile()) return '100%'
  if (isTablet()) return '60vw'
  return '50vw'
}

/** Spostamento griglia quando si apre il pannello */
const getGridShift = () => {
  if (isMobile()) return '-=20vw'
  if (isTablet()) return '-=40vw'
  return '-=50vw'
}

/** Confine del pannello per gestire click */
const getPanelBoundary = () => {
  if (isMobile()) return window.innerWidth * 0.8
  if (isTablet()) return window.innerWidth * 0.6
  return window.innerWidth * 0.5
}

// ===== NAVIGAZIONE =====
/** Torna alla HeroPage */
const goToHero = (e) => {
  if (e) {
    e.preventDefault()
    e.stopPropagation()
    e.stopImmediatePropagation()
  }
  emit('navigate', 'hero')
  return false
}

// ===== GRIGLIA: POSIZIONAMENTO E SETUP =====
/** Centra la griglia nella viewport */
const centerGrid = () => {
  const grid = mainGridRef.value?.gridRef
  if (!grid) return

  const gridWidth = grid.offsetWidth
  const gridHeight = grid.offsetHeight
  const centerX = (window.innerWidth - gridWidth) / 2
  const centerY = (window.innerHeight - gridHeight) / 2

  gsap.set(grid, { x: centerX, y: centerY })
}

/** Animazione intro: zoom della griglia con stagger sui dipinti */
const intro = async () => {
  await nextTick()

  const grid = mainGridRef.value?.gridRef
  const paintings = document.querySelectorAll('.painting')
  const details = document.querySelector('.details')

  if (!grid) return

  // Blocca scroll durante l'animazione
  isAnimating.value = true
  document.body.style.overflow = 'hidden'
  document.body.style.position = 'fixed'
  document.body.style.width = '100%'

  const timeline = gsap.timeline()

  // Stato iniziale: container scalato, dipinti nascosti
  timeline.set(containerRef.value, { scale: 0.5 })
  timeline.set(paintings, { scale: 0.5, opacity: 0 })
  if (details) timeline.set(details, { opacity: 0 })

  // Animazione dipinti con stagger random
  timeline.to(paintings, {
    scale: 1,
    opacity: 1,
    duration: 0.6,
    ease: 'power3.out',
    stagger: { amount: 1.2, from: 'random' },
  })

  // Zoom container
  timeline.to(
    containerRef.value,
    {
      scale: 1,
      duration: 1.2,
      ease: 'power3.inOut',
      onComplete: () => {
        if (details) gsap.set(details, { opacity: 1 })

        // Mostra bottone Home con animazione
        if (homeButtonRef.value) {
          gsap.fromTo(
            homeButtonRef.value,
            { opacity: 0, y: -20 },
            { opacity: 1, y: 0, duration: 0.6, ease: 'power2.out' },
          )
        }

        // Sblocca scroll
        isAnimating.value = false
        document.body.style.overflow = ''
        document.body.style.position = ''
        document.body.style.width = ''
      },
    },
    '+=0.3',
  )
}

// ===== DRAG (SOLO DESKTOP) =====
/** Setup del drag della griglia con GSAP Draggable */
const setupDraggable = () => {
  const grid = mainGridRef.value?.gridRef
  if (!grid) return

  // Su mobile non usiamo Draggable (usiamo touch scroll)
  if (isMobile()) return

  Draggable.create(grid, {
    type: 'x,y',
    bounds: {
      minX: -(grid.offsetWidth - window.innerWidth) - 200,
      maxX: 200,
      minY: -(grid.offsetHeight - window.innerHeight) - 200,
      maxY: 100,
    },
    inertia: true,
    onDragStart: () => {
      isDragging.value = true
      grid.classList.add('--is-dragging')
    },
    onDragEnd: () => {
      isDragging.value = false
      grid.classList.remove('--is-dragging')
    },
  })
}

// ===== SMOOTH SCROLL =====
/**
 * Inizializza lo smooth scroll custom
 * Usa interpolazione lineare (lerp) per movimento fluido
 */
const initSmoothScroll = () => {
  // Su mobile usiamo touch scroll, su desktop smooth scroll
  if (isMobile()) return

  const grid = mainGridRef.value?.gridRef
  if (!grid) return

  // Calcola i bounds della griglia
  const getBounds = () => ({
    minX: -(grid.offsetWidth - window.innerWidth) - 200,
    maxX: 200,
    minY: -(grid.offsetHeight - window.innerHeight) - 200,
    maxY: 100,
  })

  // Limita un valore tra min e max
  const clamp = (value, min, max) => Math.max(min, Math.min(max, value))

  // Inizializza posizione corrente dalla griglia
  currentX = gsap.getProperty(grid, 'x') || 0
  currentY = gsap.getProperty(grid, 'y') || 0
  targetX = currentX
  targetY = currentY

  // Gestisce l'evento wheel/trackpad
  const handleWheel = (e) => {
    e.preventDefault()

    // Non scrollare se dettagli aperti o animazione in corso
    if (showDetails.value || isClosing.value || isAnimating.value || isDragging.value) return

    const bounds = getBounds()

    // Accumula il delta nel target (moltiplicatore per sensibilità)
    const multiplier = 2.5
    targetX = clamp(targetX - e.deltaX * multiplier, bounds.minX, bounds.maxX)
    targetY = clamp(targetY - e.deltaY * multiplier, bounds.minY, bounds.maxY)
  }

  // Loop di animazione - interpola verso il target
  const animate = () => {
    // Non animare durante il drag
    if (isDragging.value) {
      // Sincronizza target con posizione corrente del drag
      currentX = gsap.getProperty(grid, 'x') || 0
      currentY = gsap.getProperty(grid, 'y') || 0
      targetX = currentX
      targetY = currentY
      rafId = requestAnimationFrame(animate)
      return
    }

    // Interpolazione lineare (lerp)
    currentX += (targetX - currentX) * LERP_FACTOR
    currentY += (targetY - currentY) * LERP_FACTOR

    // Applica la posizione alla griglia
    gsap.set(grid, { x: currentX, y: currentY })

    rafId = requestAnimationFrame(animate)
  }

  // Aggiungi listener wheel
  window.addEventListener('wheel', handleWheel, { passive: false })

  // Avvia il loop di animazione
  rafId = requestAnimationFrame(animate)
}

// ===== TOUCH SCROLL (MOBILE) =====
const addTouchScrollListener = () => {
  if (!isMobile()) return

  const grid = mainGridRef.value?.gridRef
  if (!grid) return

  // Calcola i bounds della griglia
  const getBounds = () => ({
    minX: -(grid.offsetWidth - window.innerWidth) - 200,
    maxX: 200,
    minY: -(grid.offsetHeight - window.innerHeight) - 200,
    maxY: 100,
  })

  const clamp = (value, min, max) => Math.max(min, Math.min(max, value))

  let touchStartX = 0
  let touchStartY = 0
  let touchStartGridX = 0
  let touchStartGridY = 0
  let isTouching = false
  let isScrolling = false
  let touchMoveAnimation = null
  let touchStartTime = 0

  const handleTouchStart = (e) => {
    if (showDetails.value || isClosing.value || isAnimating.value) return

    touchStartTime = Date.now()
    isTouching = true
    isScrolling = false
    touchStartX = e.touches[0].clientX
    touchStartY = e.touches[0].clientY
    touchStartGridX = gsap.getProperty(grid, 'x')
    touchStartGridY = gsap.getProperty(grid, 'y')

    if (touchMoveAnimation) touchMoveAnimation.kill()
  }

  const handleTouchMove = (e) => {
    if (!isTouching) return

    const currentX = e.touches[0].clientX
    const currentY = e.touches[0].clientY
    const deltaX = touchStartX - currentX
    const deltaY = touchStartY - currentY
    const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY)

    // Se il movimento è > 10px, è uno scroll (non un tap)
    if (distance > 10) {
      if (!isScrolling) isScrolling = true
      e.preventDefault()
      e.stopPropagation()

      const bounds = getBounds()
      const newX = clamp(touchStartGridX - deltaX, bounds.minX, bounds.maxX)
      const newY = clamp(touchStartGridY - deltaY, bounds.minY, bounds.maxY)

      gsap.set(grid, { x: newX, y: newY })
    }
  }

  const handleTouchEnd = (e) => {
    if (!isTouching) return

    const touchDuration = Date.now() - touchStartTime
    const currentX = e.changedTouches[0]?.clientX || touchStartX
    const currentY = e.changedTouches[0]?.clientY || touchStartY
    const finalDeltaX = touchStartX - currentX
    const finalDeltaY = touchStartY - currentY
    const finalDistance = Math.sqrt(finalDeltaX * finalDeltaX + finalDeltaY * finalDeltaY)

    // Aggiungi inerzia se era uno scroll
    if (isScrolling && finalDistance > 10) {
      const velocity = finalDistance / Math.max(touchDuration, 1)
      const inertiaX = finalDeltaX * velocity * 0.2
      const inertiaY = finalDeltaY * velocity * 0.2
      const bounds = getBounds()

      const currentGridX = gsap.getProperty(grid, 'x')
      const currentGridY = gsap.getProperty(grid, 'y')
      const newX = clamp(currentGridX - inertiaX, bounds.minX, bounds.maxX)
      const newY = clamp(currentGridY - inertiaY, bounds.minY, bounds.maxY)

      // Durata basata sulla velocità
      const inertiaDuration = Math.min(1, velocity * 0.4)

      touchMoveAnimation = gsap.to(grid, {
        x: newX,
        y: newY,
        duration: inertiaDuration,
        ease: 'power3.out',
      })
    }

    isTouching = false
    isScrolling = false
  }

  window.addEventListener('touchstart', handleTouchStart, { passive: true })
  window.addEventListener('touchmove', handleTouchMove, { passive: false })
  window.addEventListener('touchend', handleTouchEnd, { passive: true })
}

// ===== DETTAGLI DIPINTO =====
/** Apre il pannello dettagli per un dipinto */
const openDetails = (clickData) => {
  if (showDetails.value) {
    closeDetails()
    return
  }

  currentPainting.value = clickData.data
  showDetails.value = true

  // Salva posizione griglia per il ritorno
  const grid = mainGridRef.value?.gridRef
  if (grid) {
    originalGridPosition.value = {
      x: gsap.getProperty(grid, 'x'),
      y: gsap.getProperty(grid, 'y'),
    }
  }

  nextTick(() => {
    const details = document.querySelector('.details')
    const title = document.querySelector('.details__title p')
    const description = document.querySelector('.details__body p')
    const cross = document.querySelector('.cross')

    // Prepara elementi per animazione
    if (title) gsap.set(title, { opacity: 0 })
    if (description) gsap.set(description, { opacity: 0 })
    if (cross) gsap.set(cross, { scale: 1, opacity: 0 })

    // Sposta la griglia
    if (grid) {
      gsap.to(grid, {
        x: getGridShift(),
        duration: 2,
        ease: 'power2.out',
        delay: 0.1,
      })
    }

    // Anima entrata pannello
    if (details) {
      gsap.fromTo(
        details,
        { x: getDetailsPanelWidth() },
        { x: 0, duration: 1.6, ease: 'power3.out', delay: 0.2 },
      )
    }
  })

  // FLIP: sposta l'immagine nel pannello
  flipProduct(clickData.element)

  // Anima i testi dopo che il pannello è entrato
  setTimeout(() => animateTexts(), 1200)

  // Attiva cursor cross
  window.addEventListener('mousemove', handleMouseMove)
}

/** Chiude il pannello dettagli */
const closeDetails = () => {
  isClosing.value = true

  const cross = document.querySelector('.cross')
  const grid = mainGridRef.value?.gridRef
  const details = document.querySelector('.details')

  const timeline = gsap.timeline()

  // Nascondi cursore cross
  if (cross) {
    timeline.to(cross, { scale: 0, opacity: 0, duration: 0.4, ease: 'power2.in' }, 0)
  }

  // Anima uscita titolo
  if (titleSplit?.chars) {
    timeline.to(
      titleSplit.chars,
      { y: '-100%', opacity: 0, duration: 0.5, ease: 'power3.in', stagger: 0.008 },
      0,
    )
  }

  // Anima uscita descrizione
  if (descSplit?.lines) {
    timeline.to(
      descSplit.lines,
      { y: '-100%', opacity: 0, duration: 0.4, ease: 'power3.in', stagger: 0.02 },
      0.1,
    )
  }

  // Riporta griglia alla posizione originale
  if (grid) {
    const currentX = gsap.getProperty(grid, 'x')
    const shiftMultiplier = isMobile() ? 0.2 : isTablet() ? 0.4 : 0.5
    const targetX = currentX + window.innerWidth * shiftMultiplier

    timeline.to(grid, { x: targetX, duration: 1.2, ease: 'power3.inOut' }, 0.3)
  }

  // Anima uscita pannello
  if (details) {
    timeline.to(details, { x: getDetailsPanelWidth(), duration: 1.2, ease: 'power3.inOut' }, 0.3)
  }

  // FLIP: riporta immagine nella griglia
  timeline.call(() => unFlipProduct(), null, 0.6)

  // Cleanup
  timeline.call(
    () => {
      if (titleSplit) {
        titleSplit.revert()
        titleSplit = null
      }
      if (descSplit) {
        descSplit.revert()
        descSplit = null
      }

      showDetails.value = false
      currentPainting.value = null
      isClosing.value = false
    },
    null,
    1.8,
  )

  window.removeEventListener('mousemove', handleMouseMove)
}

// ===== FLIP ANIMATION =====
/** Anima l'immagine dalla griglia al pannello dettagli */
const flipProduct = (clickedElement) => {
  const detailsThumb = document.querySelector('.details__thumb')
  if (!detailsThumb || !clickedElement) return

  currentPaintingElement.value = clickedElement
  originalParent.value = clickedElement.parentNode

  // Cattura stato iniziale
  const state = Flip.getState(clickedElement)

  // Sposta nel pannello
  detailsThumb.appendChild(clickedElement)
  gsap.set(clickedElement, { width: '100%', height: '100%' })

  // Anima la transizione
  Flip.from(state, {
    absolute: true,
    duration: 1.4,
    ease: 'power3.inOut',
    simple: true,
  })
}

/** Riporta l'immagine nella griglia */
const unFlipProduct = () => {
  if (!currentPaintingElement.value || !originalParent.value) return

  const state = Flip.getState(currentPaintingElement.value)

  // Riporta nel parent originale
  originalParent.value.appendChild(currentPaintingElement.value)

  Flip.from(state, {
    absolute: true,
    duration: 1.4,
    ease: 'power3.inOut',
    simple: true,
    onComplete: () => {
      gsap.set(currentPaintingElement.value, { clearProps: 'all' })
      currentPaintingElement.value = null
      originalParent.value = null
    },
  })
}

// ===== ANIMAZIONE TESTI DETTAGLI =====
/** Anima titolo e descrizione nel pannello */
const animateTexts = () => {
  const title = document.querySelector('.details__title p')
  const description = document.querySelector('.details__body p')

  // Cleanup precedenti
  if (titleSplit) titleSplit.revert()
  if (descSplit) descSplit.revert()

  // Anima titolo
  if (title) {
    gsap.set(title, { opacity: 1 })
    titleSplit = new SplitText(title, { type: 'chars', charsClass: 'char' })

    gsap.fromTo(
      titleSplit.chars,
      { y: '120%', opacity: 0 },
      { y: 0, opacity: 1, duration: 1.4, delay: 0.8, ease: 'power3.out', stagger: 0.015 },
    )
  }

  // Anima descrizione (per linee)
  if (description) {
    gsap.set(description, { opacity: 1 })
    descSplit = new SplitText(description, { type: 'lines', linesClass: 'line' })

    gsap.fromTo(
      descSplit.lines,
      { y: '120%', opacity: 0 },
      { y: 0, opacity: 1, duration: 1.3, delay: 1.1, ease: 'power4.out', stagger: 0.06 },
    )
  }
}

// ===== GESTIONE CURSORE CROSS =====
/** Mostra/nasconde il cursore cross in base alla posizione */
const handleMouseMove = (e) => {
  if (!showDetails.value) return

  const cross = document.querySelector('.cross')
  if (!cross) return

  const isInPanel = e.clientX > getPanelBoundary()

  if (isInPanel) {
    gsap.to(cross, { opacity: 0, duration: 0.2 })
  } else {
    gsap.to(cross, {
      x: e.clientX - 20,
      y: e.clientY - 20,
      opacity: 1,
      duration: 0.1,
      ease: 'none',
    })
  }
}

// ===== GESTIONE CLICK CONTAINER =====
/** Chiude i dettagli se si clicca fuori dal pannello */
const handleContainerClick = (e) => {
  const clientX = e.clientX || (e.touches && e.touches[0]?.clientX) || 0

  // Non chiudere se click su un dipinto
  if (e.target.closest('.painting')) return

  if (showDetails.value && clientX < getPanelBoundary()) {
    closeDetails()
  }
}

// ===== INTERSECTION OBSERVER =====
/** Osserva i dipinti per applicare effetti quando entrano/escono dalla viewport */
const observeProducts = () => {
  const paintings = document.querySelectorAll('.painting')
  if (paintings.length === 0) return

  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          gsap.to(entry.target, { scale: 1, opacity: 1, duration: 0.5, ease: 'power2.out' })
        } else {
          gsap.to(entry.target, { opacity: 0.3, scale: 0.8, duration: 0.5, ease: 'power2.in' })
        }
      })
    },
    { root: null, threshold: 0.1 },
  )

  paintings.forEach((painting) => observer.observe(painting))
}

// ===== LIFECYCLE =====
onMounted(async () => {
  await nextTick()

  intro()
  centerGrid()
  setupDraggable()

  // Inizializza smooth scroll su desktop, touch scroll su mobile
  if (!isMobile()) {
    initSmoothScroll()
  } else {
    addTouchScrollListener()
  }

  observeProducts()

  // Event listener extra per il bottone Home
  if (homeButtonRef.value) {
    homeButtonRef.value.addEventListener('click', goToHero)
  }
})

onUnmounted(() => {
  // Cleanup animation frame
  if (rafId) {
    cancelAnimationFrame(rafId)
    rafId = null
  }

  // Rimuovi listener wheel
  window.removeEventListener('wheel', () => {})

  if (homeButtonRef.value) {
    homeButtonRef.value.removeEventListener('click', goToHero)
  }
})
</script>

<style scoped>
/* Container principale - fixed per coprire la viewport */
.container {
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  transform-origin: center center;
  will-change: transform;
  touch-action: pan-x pan-y;
}

/* Bottone Home - fisso in alto a sinistra */
.home-button {
  position: fixed;
  top: 2rem;
  left: 2rem;
  z-index: 10001;
  cursor: pointer;
  pointer-events: auto;
  user-select: none;
  opacity: 0; /* Nascosto inizialmente, animato dopo intro */
}

.home-button h2 {
  margin: 0;
  font-family: 'EB Garamond', serif;
  font-size: clamp(1rem, 1.5vw, 1.5rem);
  font-weight: 400;
  color: #c49852;
}

/* Pseudo-elemento per effetto sfondo (opzionale) */
.container::before {
  content: '';
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  opacity: 0.08;
  z-index: -1;
  pointer-events: none;
}

/* Stato iniziale per SplitText */
.line,
.char {
  transform: translate3d(0, 100%, 0);
}
</style>
