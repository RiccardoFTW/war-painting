<template>
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
  <div ref="homeButtonRef" class="home-button" @click.stop="goToHero">
    <h2>Home</h2>
  </div>
  <div class="blum-overlay"></div>
  <div class="blum-overlay"></div>
  <div class="blum-overlay"></div>
</template>

<script setup>
// IMPORTS
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import MainGrid from './MainGrid.vue'
import PaintingDetails from './PaintingDetails.vue'
import { Draggable, Flip, SplitText } from 'gsap/all'
import gsap from 'gsap'
import paintingsData from '../data/paintings.json'

const emit = defineEmits(['navigate'])

const goToHero = (e) => {
  if (e) {
    e.preventDefault()
    e.stopPropagation()
    e.stopImmediatePropagation()
  }
  emit('navigate', 'hero')
  return false
}

// PLUGIN REGISTRATION
gsap.registerPlugin(Draggable, Flip, SplitText)

// REACTIVE STATE
const containerRef = ref(null)
const mainGridRef = ref(null)
const paintingDetailsRef = ref(null)
const homeButtonRef = ref(null)

const isDragging = ref(false)
const showDetails = ref(false)
const currentPainting = ref(null)
const isClosing = ref(false)
const isAnimating = ref(false)

const currentPaintingElement = ref(null)
const originalParent = ref(null)
const originalGridPosition = ref({ x: 0, y: 0 })

let titleSplit = null
let descSplit = null

// RESPONSIVE HELPER FUNCTIONS
const isMobile = () => window.innerWidth <= 768
const isTablet = () => window.innerWidth > 768 && window.innerWidth <= 1024

const getDetailsPanelWidth = () => {
  if (isMobile()) return '100%'
  if (isTablet()) return '60vw'
  return '50vw'
}

const getGridShift = () => {
  if (isMobile()) return '-=20vw'
  if (isTablet()) return '-=40vw'
  return '-=50vw'
}

const getPanelBoundary = () => {
  if (isMobile()) return window.innerWidth * 0.8
  if (isTablet()) return window.innerWidth * 0.6
  return window.innerWidth * 0.5
}

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

  // Blocca lo scroll durante l'animazione
  isAnimating.value = true
  document.body.style.overflow = 'hidden'
  document.body.style.position = 'fixed'
  document.body.style.width = '100%'

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
    stagger: {
      amount: 1.2,
      from: 'random',
    },
  })

  timeline.to(
    containerRef.value,
    {
      scale: 1,
      duration: 1.2,
      ease: 'power3.inOut',
      onComplete: () => {
        if (details) {
          gsap.set(details, { opacity: 1 })
        }
        // Home button delay
        if (homeButtonRef.value) {
          gsap.fromTo(
            homeButtonRef.value,
            {
              opacity: 0,
              y: -20,
            },
            {
              opacity: 1,
              y: 0,
              duration: 0.6,
              ease: 'power2.out',
            },
          )
        }
        // Sblocca lo scroll dopo che l'animazione è completata
        isAnimating.value = false
        document.body.style.overflow = ''
        document.body.style.position = ''
        document.body.style.width = ''
      },
    },
    '+=0.3',
  )
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

    if (showDetails.value || isClosing.value || isAnimating.value) return

    const deltaX = -e.deltaX * 3
    const deltaY = -e.deltaY * 3

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
    const grid = mainGridRef.value?.gridRef

    if (title) gsap.set(title, { opacity: 0 })
    if (description) gsap.set(description, { opacity: 0 })
    if (cross) gsap.set(cross, { scale: 1, opacity: 0 })

    if (grid) {
      gsap.to(grid, {
        x: getGridShift(),
        duration: 2,
        ease: 'power2.out',
        delay: 0.1,
      })
    }

    if (details) {
      gsap.fromTo(
        details,
        {
          x: getDetailsPanelWidth(),
        },
        {
          x: 0,
          duration: 1.6,
          ease: 'power3.out',
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
  isClosing.value = true

  const cross = document.querySelector('.cross')
  const grid = mainGridRef.value?.gridRef

  const timeline = gsap.timeline()

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

  if (grid) {
    const currentX = gsap.getProperty(grid, 'x')
    const shiftMultiplier = isMobile() ? 0.2 : isTablet() ? 0.4 : 0.5
    const targetX = currentX + window.innerWidth * shiftMultiplier

    timeline.to(
      grid,
      {
        x: targetX,
        duration: 1.2,
        ease: 'power3.inOut',
      },
      0.3,
    )
  }

  const details = document.querySelector('.details')
  if (details) {
    timeline.to(
      details,
      {
        x: getDetailsPanelWidth(),
        duration: 1.2,
        ease: 'power3.inOut',
      },
      0.3,
    )
  }

  timeline.call(
    () => {
      unFlipProduct()
    },
    null,
    0.6,
  )

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

const flipProduct = (clickedElement) => {
  const detailsThumb = document.querySelector('.details__thumb')

  if (!detailsThumb || !clickedElement) return

  currentPaintingElement.value = clickedElement
  originalParent.value = clickedElement.parentNode

  const state = Flip.getState(clickedElement)

  detailsThumb.appendChild(clickedElement)

  gsap.set(clickedElement, {
    width: '100%',
    height: '100%',
  })

  Flip.from(state, {
    absolute: true,
    duration: 1.4,
    ease: 'power3.inOut',
    simple: true,
  })
}

const unFlipProduct = () => {
  if (!currentPaintingElement.value || !originalParent.value) return

  const state = Flip.getState(currentPaintingElement.value)

  originalParent.value.appendChild(currentPaintingElement.value)

  Flip.from(state, {
    absolute: true,
    duration: 1.4,
    ease: 'power3.inOut',
    simple: true,
    onComplete: () => {
      gsap.set(currentPaintingElement.value, {
        clearProps: 'all',
      })
      currentPaintingElement.value = null
      originalParent.value = null
    },
  })
}

const animateTexts = () => {
  const title = document.querySelector('.details__title p')
  const description = document.querySelector('.details__body p')

  if (titleSplit) titleSplit.revert()
  if (descSplit) descSplit.revert()

  if (title) {
    gsap.set(title, { opacity: 1 })

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
    gsap.set(description, { opacity: 1 })

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

  const isInPanel = x > getPanelBoundary()

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
  if (showDetails.value && e.clientX < getPanelBoundary()) {
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

  // Aggiungi event listener manualmente su home button
  if (homeButtonRef.value) {
    homeButtonRef.value.addEventListener('click', goToHero)
    homeButtonRef.value.addEventListener('mousedown', goToHero)
  }
})

onUnmounted(() => {
  if (homeButtonRef.value) {
    homeButtonRef.value.removeEventListener('click', goToHero)
    homeButtonRef.value.removeEventListener('mousedown', goToHero)
  }
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

.home-button {
  position: fixed;
  top: 2rem;
  left: 2rem;
  z-index: 10001;
  cursor: pointer;
  pointer-events: auto;
  user-select: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  opacity: 0;
}

.home-button h2 {
  margin: 0;
  font-family: 'EB Garamond', 'Serif';
  font-size: clamp(1rem, 1.5vw, 1.5rem);
  font-weight: 400;
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
  z-index: -1;
  opacity: 0.5;
}

.container::before {
  content: '';
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  /* background-image: url(../assets/img/intro-img/6.svg); */
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
