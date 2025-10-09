<template>
  <div class="container" ref="containerRef" @click="handleContainerClick">
    <MainGrid ref="mainGridRef" @painting-click="openDetails" />

    <PaintingDetails
      ref="paintingDetailsRef"
      :is-visible="showDetails"
      :current-painting="currentPainting"
      @close="closeDetails"
    />
  </div>
</template>

<script setup>
// IMPORTS
import { ref, onMounted, nextTick } from 'vue'
import MainGrid from './MainGrid.vue'
import PaintingDetails from './PaintingDetails.vue'
import { Draggable, Flip, SplitText } from 'gsap/all'
import gsap from 'gsap'

// PLUGIN REGISTRATION
gsap.registerPlugin(Draggable, Flip, SplitText)

// REACTIVE STATE

// Grid Refs
const containerRef = ref(null)
const mainGridRef = ref(null)
const paintingDetailsRef = ref(null)

// State management
const isDragging = ref(false)
const showDetails = ref(false)
const currentPainting = ref(null)

// Flip animation state
const currentPaintingElement = ref(null)
const originalParent = ref(null)

// SplitText instances per cleanup
let titleSplit = null
let descSplit = null

// FUNCTIONS

// Grid Functions
const centerGrid = () => {
  const grid = mainGridRef.value.gridRef

  if (!grid) return

  const gridWidth = grid.offsetWidth
  const gridHeight = grid.offsetHeight
  const windowWidth = window.innerWidth
  const windowHeight = window.innerHeight

  const centerX = (windowWidth - gridWidth) / 2
  const centerY = (windowHeight - gridHeight) / 2

  gsap.set(grid, { x: centerX, y: centerY })
}

const intro = async () => {
  await nextTick()

  const grid = mainGridRef.value.gridRef
  const paintings = document.querySelectorAll('.painting')
  const details = document.querySelector('.details')

  if (!grid) return

  const timeline = gsap.timeline()

  timeline.set(containerRef.value, {
    scale: 0.5,
  })

  timeline.set(paintings, {
    scale: 0.5,
    opacity: 0,
  })

  if (details) {
    timeline.set(details, { opacity: 0 })
  }

  timeline.to(paintings, {
    scale: 1,
    opacity: 1,
    duration: 0.6,
    ease: 'power3.out',
    stagger: { amount: 1.2, from: 'random' },
  })

  timeline.to(containerRef.value, {
    scale: 1,
    duration: 1.2,
    ease: 'power3.inOut',
    onComplete: () => {
      if (details) {
        gsap.set(details, { opacity: 1 })
      }
    },
  })
}

// Drag functions
const setupDraggable = () => {
  const grid = mainGridRef.value.gridRef

  if (!grid) return

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

const addScrollListener = () => {
  const handleWheel = (e) => {
    e.preventDefault()

    const grid = mainGridRef.value?.gridRef
    if (!grid) return

    if (showDetails.value) return

    const deltaX = -e.deltaX * 7
    const deltaY = -e.deltaY * 7

    const currentX = gsap.getProperty(grid, 'x')
    const currentY = gsap.getProperty(grid, 'y')

    const newX = currentX + deltaX
    const newY = currentY + deltaY

    const bounds = {
      minX: -(grid.offsetWidth - window.innerWidth) - 200,
      maxX: 200,
      minY: -(grid.offsetHeight - window.innerHeight) - 200,
      maxY: 100,
    }

    const clampedX = Math.max(bounds.minX, Math.min(bounds.maxX, newX))
    const clampedY = Math.max(bounds.minY, Math.min(bounds.maxY, newY))

    gsap.to(grid, {
      x: clampedX,
      y: clampedY,
      duration: 0.3,
      ease: 'power3.out',
    })
  }

  window.addEventListener('wheel', handleWheel, { passive: false })
}

// Details functions
const openDetails = (clickData) => {
  if (showDetails.value) {
    closeDetails()
    return
  }

  currentPainting.value = clickData.data
  showDetails.value = true

  nextTick(() => {
    const details = document.querySelector('.details')
    const title = document.querySelector('.details__title p')
    const description = document.querySelector('.details__body p')
    const cross = document.querySelector('.cross')

    // Nascondi immediatamente testi prima delle animazioni
    if (title) gsap.set(title, { opacity: 0 })
    if (description) gsap.set(description, { opacity: 0 })
    if (cross) gsap.set(cross, { scale: 1, opacity: 0 })

    // Anima apertura pannello
    if (details) {
      gsap.fromTo(
        details,
        {
          x: window.innerWidth <= 600 ? '100%' : '60vw',
          opacity: 0,
        },
        {
          x: 0,
          opacity: 1,
          duration: 1.6,
          ease: 'power4.out',
          delay: 0.2,
        },
      )
    }
  })

  flipProduct(clickData.element)

  setTimeout(() => {
    animateTexts(clickData.data.id)
  }, 1200)

  window.addEventListener('mousemove', handleMouseMove)
}

const closeDetails = () => {
  const cross = document.querySelector('.cross')

  const timeline = gsap.timeline()

  // Nascondi cross
  if (cross) {
    timeline.to(
      cross,
      {
        scale: 0,
        opacity: 0,
        duration: 0.4,
        ease: 'power2.in',
      },
      0,
    )
  }

  // Nascondi testi usando i split salvati
  if (titleSplit && titleSplit.chars) {
    timeline.to(
      titleSplit.chars,
      {
        y: '-100%',
        opacity: 0,
        duration: 0.5,
        ease: 'power3.in',
        stagger: 0.008,
      },
      0,
    )
  }

  if (descSplit && descSplit.lines) {
    timeline.to(
      descSplit.lines,
      {
        y: '-100%',
        opacity: 0,
        duration: 0.4,
        ease: 'power3.in',
        stagger: 0.02,
      },
      0.1,
    )
  }

  // Chiudi pannello
  const details = document.querySelector('.details')
  if (details) {
    timeline.to(
      details,
      {
        x: window.innerWidth <= 600 ? '100%' : '60vw',
        opacity: 0,
        duration: 1.2,
        ease: 'power4.in',
      },
      0.5,
    )
  }

  // Unflip dopo che il pannello inizia a chiudersi
  timeline.call(
    () => {
      unFlipProduct()
    },
    null,
    0.6,
  )

  // Cleanup finale
  timeline.call(
    () => {
      // Pulisci i split
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
    },
    null,
    1.8,
  )

  window.removeEventListener('mousemove', handleMouseMove)
}

const flipProduct = (clickedElement) => {
  const detailsThumb = document.querySelector('.details__thumb')

  if (!detailsThumb || !clickedElement) return

  // Salva riferimenti per il flip inverso
  currentPaintingElement.value = clickedElement
  originalParent.value = clickedElement.parentNode

  // Cattura lo stato iniziale (posizione nella griglia)
  const state = Flip.getState(clickedElement)

  // Imposta dimensioni fisse prima di spostare
  gsap.set(clickedElement, {
    width: '18.5vw',
    height: '18.5vw',
  })

  // Sposta fisicamente l'immagine dentro il thumb
  detailsThumb.appendChild(clickedElement)

  // Anima dalla posizione originale alla nuova posizione
  Flip.from(state, {
    absolute: true,
    duration: 1.4,
    ease: 'power3.inOut',
    simple: true,
  })
}

const unFlipProduct = () => {
  if (!currentPaintingElement.value || !originalParent.value) return

  // Cattura lo stato attuale (dentro il thumb)
  const state = Flip.getState(currentPaintingElement.value)

  // Rimetti l'immagine nella posizione originale
  originalParent.value.appendChild(currentPaintingElement.value)

  // Anima dal thumb alla posizione originale
  Flip.from(state, {
    absolute: true,
    duration: 1.4,
    ease: 'power3.inOut',
    simple: true,
    onComplete: () => {
      // Pulisci tutti gli stili inline aggiunti da GSAP
      gsap.set(currentPaintingElement.value, { clearProps: 'all' })
      currentPaintingElement.value = null
      originalParent.value = null
    },
  })
}

// Animations functions
const animateTexts = () => {
  const title = document.querySelector('.details__title p')
  const description = document.querySelector('.details__body p')

  // Pulisci eventuali split precedenti
  if (titleSplit) titleSplit.revert()
  if (descSplit) descSplit.revert()

  if (title) {
    // Rendi visibile il paragrafo prima di splittarlo
    gsap.set(title, { opacity: 1 })

    // Crea nuovo split e salvalo per cleanup futuro
    titleSplit = new SplitText(title, {
      type: 'chars',
      charsClass: 'char',
    })

    gsap.fromTo(
      titleSplit.chars,
      {
        y: '120%',
        opacity: 0,
      },
      {
        y: 0,
        opacity: 1,
        duration: 1.4,
        delay: 0.8,
        ease: 'power4.out',
        stagger: 0.015,
      },
    )
  }

  if (description) {
    // Rendi visibile il paragrafo prima di splittarlo
    gsap.set(description, { opacity: 1 })

    // Crea nuovo split e salvalo per cleanup futuro
    descSplit = new SplitText(description, {
      type: 'lines',
      linesClass: 'line',
    })

    gsap.fromTo(
      descSplit.lines,
      {
        y: '120%',
        opacity: 0,
      },
      {
        y: 0,
        opacity: 1,
        duration: 1.3,
        delay: 1.1,
        ease: 'power4.out',
        stagger: 0.06,
      },
    )
  }
}

const handleMouseMove = (e) => {
  if (!showDetails.value) return

  const cross = document.querySelector('.cross')
  if (!cross) return

  const x = e.clientX
  const y = e.clientY

  const isInPanel = x > window.innerWidth / 2

  if (isInPanel) {
    gsap.to(cross, {
      opacity: 0,
      duration: 0.2,
    })
  } else {
    gsap.to(cross, {
      x: x - 20,
      y: y - 20,
      opacity: 1,
      duration: 0.1,
      ease: 'none',
    })
  }
}

const handleContainerClick = (e) => {
  if (showDetails.value && e.clientX < window.innerWidth / 2) {
    closeDetails()
  }
}

// Observer functions
const observeProducts = () => {
  const paintings = document.querySelectorAll('.painting')

  if (paintings.length === 0) return

  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          gsap.to(entry.target, {
            scale: 1,
            opacity: 1,
            duration: 0.5,
            ease: 'power2.out',
          })
        } else {
          gsap.to(entry.target, {
            opacity: 0.3,
            scale: 0.8,
            duration: 0.5,
            ease: 'power2.in',
          })
        }
      })
    },
    {
      root: null,
      threshold: 0.1,
    },
  )

  paintings.forEach((painting) => {
    observer.observe(painting)
  })
}

// LIFECYCLE
onMounted(async () => {
  await nextTick()
  intro()
  centerGrid()
  setupDraggable()
  addScrollListener()
  observeProducts()
})
</script>

<style scoped>
.container {
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;

  transform-origin: center center;
  will-change: transform;
}

.container::before {
  content: '';
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  background-image: url(../assets/img/intro-img/6.svg);
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  opacity: 0.08;
  z-index: -1;
  pointer-events: none;
}

.line,
.char {
  transform: translate3d(0, 100%, 0);
}
</style>
