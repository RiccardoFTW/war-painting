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
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import MainGrid from './MainGrid.vue'
import PaintingDetails from './PaintingDetails.vue'
import { Draggable, Flip, SplitText } from 'gsap/all'
import gsap from 'gsap'
import paintingsData from '../data/paintings.json'

gsap.registerPlugin(Draggable, Flip, SplitText)

// ===== EVENTI =====
const emit = defineEmits(['navigate'])

// ===== STATO REATTIVO =====
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


let targetX = 0
let targetY = 0
let currentX = 0
let currentY = 0
let rafId = null
const LERP_FACTOR = 0.08

// ===== HELPERS RESPONSIVE =====
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

// ===== NAVIGAZIONE =====
const goToHero = (e) => {
  if (e) {
    e.preventDefault()
    e.stopPropagation()
    e.stopImmediatePropagation()
  }
  emit('navigate', 'hero')
  return false
}

const centerGrid = () => {
  const grid = mainGridRef.value?.gridRef
  if (!grid) return

  const gridWidth = grid.offsetWidth
  const gridHeight = grid.offsetHeight
  const centerX = (window.innerWidth - gridWidth) / 2
  const centerY = (window.innerHeight - gridHeight) / 2

  gsap.set(grid, { x: centerX, y: centerY })
}

const intro = async () => {
  await nextTick()

  const grid = mainGridRef.value?.gridRef
  const paintings = document.querySelectorAll('.painting')
  const details = document.querySelector('.details')

  if (!grid) return


  isAnimating.value = true
  document.body.style.overflow = 'hidden'
  document.body.style.position = 'fixed'
  document.body.style.width = '100%'

  const timeline = gsap.timeline()


  timeline.set(containerRef.value, { scale: 0.5 })
  timeline.set(paintings, { scale: 0.5, opacity: 0 })
  if (details) timeline.set(details, { opacity: 0 })


  timeline.to(paintings, {
    scale: 1,
    opacity: 1,
    duration: 0.6,
    ease: 'power3.out',
    stagger: { amount: 1.2, from: 'random' },
  })


  timeline.to(
    containerRef.value,
    {
      scale: 1,
      duration: 1.2,
      ease: 'power3.inOut',
      onComplete: () => {
        if (details) gsap.set(details, { opacity: 1 })


        if (homeButtonRef.value) {
          gsap.fromTo(
            homeButtonRef.value,
            { opacity: 0, y: -20 },
            { opacity: 1, y: 0, duration: 0.6, ease: 'power2.out' },
          )
        }

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
const setupDraggable = () => {
  const grid = mainGridRef.value?.gridRef
  if (!grid) return

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
const initSmoothScroll = () => {
  if (isMobile()) return

  const grid = mainGridRef.value?.gridRef
  if (!grid) return

  const getBounds = () => ({
    minX: -(grid.offsetWidth - window.innerWidth) - 200,
    maxX: 200,
    minY: -(grid.offsetHeight - window.innerHeight) - 200,
    maxY: 100,
  })

  const clamp = (value, min, max) => Math.max(min, Math.min(max, value))

  currentX = gsap.getProperty(grid, 'x') || 0
  currentY = gsap.getProperty(grid, 'y') || 0
  targetX = currentX
  targetY = currentY

  const handleWheel = (e) => {
    e.preventDefault()

    if (showDetails.value || isClosing.value || isAnimating.value || isDragging.value) return

    const bounds = getBounds()

    const multiplier = 2.5
    targetX = clamp(targetX - e.deltaX * multiplier, bounds.minX, bounds.maxX)
    targetY = clamp(targetY - e.deltaY * multiplier, bounds.minY, bounds.maxY)
  }

  const animate = () => {
    if (isDragging.value) {
      currentX = gsap.getProperty(grid, 'x') || 0
      currentY = gsap.getProperty(grid, 'y') || 0
      targetX = currentX
      targetY = currentY
      rafId = requestAnimationFrame(animate)
      return
    }

    currentX += (targetX - currentX) * LERP_FACTOR
    currentY += (targetY - currentY) * LERP_FACTOR

    gsap.set(grid, { x: currentX, y: currentY })

    rafId = requestAnimationFrame(animate)
  }

  window.addEventListener('wheel', handleWheel, { passive: false })

  rafId = requestAnimationFrame(animate)
}

// ===== TOUCH SCROLL (MOBILE) =====
const addTouchScrollListener = () => {
  if (!isMobile()) return

  const grid = mainGridRef.value?.gridRef
  if (!grid) return

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

    if (isScrolling && finalDistance > 10) {
      const velocity = finalDistance / Math.max(touchDuration, 1)
      const inertiaX = finalDeltaX * velocity * 0.2
      const inertiaY = finalDeltaY * velocity * 0.2
      const bounds = getBounds()

      const currentGridX = gsap.getProperty(grid, 'x')
      const currentGridY = gsap.getProperty(grid, 'y')
      const newX = clamp(currentGridX - inertiaX, bounds.minX, bounds.maxX)
      const newY = clamp(currentGridY - inertiaY, bounds.minY, bounds.maxY)

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
        { x: getDetailsPanelWidth() },
        { x: 0, duration: 1.6, ease: 'power3.out', delay: 0.2 },
      )
    }
  })

  flipProduct(clickData.element)

  setTimeout(() => animateTexts(), 1200)

  window.addEventListener('mousemove', handleMouseMove)
}

const closeDetails = () => {
  isClosing.value = true

  const cross = document.querySelector('.cross')
  const grid = mainGridRef.value?.gridRef
  const details = document.querySelector('.details')

  const timeline = gsap.timeline()

  if (cross) {
    timeline.to(cross, { scale: 0, opacity: 0, duration: 0.4, ease: 'power2.in' }, 0)
  }

  if (titleSplit?.chars) {
    timeline.to(
      titleSplit.chars,
      { y: '-100%', opacity: 0, duration: 0.5, ease: 'power3.in', stagger: 0.008 },
      0,
    )
  }

  if (descSplit?.lines) {
    timeline.to(
      descSplit.lines,
      { y: '-100%', opacity: 0, duration: 0.4, ease: 'power3.in', stagger: 0.02 },
      0.1,
    )
  }

  if (grid) {
    const currentX = gsap.getProperty(grid, 'x')
    const shiftMultiplier = isMobile() ? 0.2 : isTablet() ? 0.4 : 0.5
    const targetX = currentX + window.innerWidth * shiftMultiplier

    timeline.to(grid, { x: targetX, duration: 1.2, ease: 'power3.inOut' }, 0.3)
  }

  if (details) {
    timeline.to(details, { x: getDetailsPanelWidth(), duration: 1.2, ease: 'power3.inOut' }, 0.3)
  }

  timeline.call(() => unFlipProduct(), null, 0.6)

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
const flipProduct = (clickedElement) => {
  const detailsThumb = document.querySelector('.details__thumb')
  if (!detailsThumb || !clickedElement) return

  currentPaintingElement.value = clickedElement
  originalParent.value = clickedElement.parentNode

  const state = Flip.getState(clickedElement)

  detailsThumb.appendChild(clickedElement)
  gsap.set(clickedElement, { width: '100%', height: '100%' })

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
      gsap.set(currentPaintingElement.value, { clearProps: 'all' })
      currentPaintingElement.value = null
      originalParent.value = null
    },
  })
}

// ===== ANIMAZIONE TESTI DETTAGLI =====
const animateTexts = () => {
  const title = document.querySelector('.details__title p')
  const description = document.querySelector('.details__body p')

  if (titleSplit) titleSplit.revert()
  if (descSplit) descSplit.revert()

  if (title) {
    gsap.set(title, { opacity: 1 })
    titleSplit = new SplitText(title, { type: 'chars', charsClass: 'char' })

    gsap.fromTo(
      titleSplit.chars,
      { y: '120%', opacity: 0 },
      { y: 0, opacity: 1, duration: 1.4, delay: 0.8, ease: 'power3.out', stagger: 0.015 },
    )
  }
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
const handleContainerClick = (e) => {
  const clientX = e.clientX || (e.touches && e.touches[0]?.clientX) || 0

  if (e.target.closest('.painting')) return

  if (showDetails.value && clientX < getPanelBoundary()) {
    closeDetails()
  }
}

// ===== INTERSECTION OBSERVER =====
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

  if (!isMobile()) {
    initSmoothScroll()
  } else {
    addTouchScrollListener()
  }

  observeProducts()

  if (homeButtonRef.value) {
    homeButtonRef.value.addEventListener('click', goToHero)
  }
})

onUnmounted(() => {
  if (rafId) {
    cancelAnimationFrame(rafId)
    rafId = null
  }
  window.removeEventListener('wheel', () => {})

  if (homeButtonRef.value) {
    homeButtonRef.value.removeEventListener('click', goToHero)
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
  touch-action: pan-x pan-y;
}

.home-button {
  position: fixed;
  top: 2rem;
  left: 2rem;
  z-index: 10001;
  cursor: pointer;
  pointer-events: auto;
  user-select: none;
  opacity: 0;
}

.home-button h2 {
  margin: 0;
  font-family: 'EB Garamond', serif;
  font-size: clamp(1rem, 1.5vw, 1.5rem);
  font-weight: 400;
  color: #c49852;
}

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

.line,
.char {
  transform: translate3d(0, 100%, 0);
}
</style>
