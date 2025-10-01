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

const intro = () => {
  const grid = mainGridRef.value.gridRef
  const paintings = document.querySelectorAll('.painting')

  if (!grid) return

  const timeline = gsap.timeline()

  timeline.set(containerRef.value, {
    scale: 0.5,
  })

  timeline.set(paintings, {
    scale: 0.5,
    opacity: 0,
  })

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
  currentPainting.value = clickData.data
  showDetails.value = true

  flipProduct(clickData.element)

  setTimeout(() => {
    animateTexts(clickData.data.id)
  }, 800)

  window.addEventListener('mousemove', handleMouseMove)

  nextTick(() => {
    const cross = paintingDetailsRef.value?.crossRef
    if (cross) {
      gsap.set(cross, { scale: 1, opacity: 0 })
    }
  })
}

const closeDetails = () => {
  unFlipProduct()
  window.removeEventListener('mousemove', handleMouseMove)

  const cross = paintingDetailsRef.value?.crossRef
  if (cross) {
    gsap.to(cross, {
      scale: 0,
      duration: 0.4,
      ease: 'power2.out',
    })
  }
  showDetails.value = false
  currentPainting.value = null

  setTimeout(() => {
    showDetails.value = false
    currentPainting.value = null
  }, 300)
}

const flipProduct = (clickedElement) => {
  const detailsThumb = document.querySelector('.details__thumb')

  if (!detailsThumb || !clickedElement) return

  currentPaintingElement.value = clickedElement
  originalParent.value = clickedElement.parentNode

  const state = Flip.getState(clickedElement)

  detailsThumb.appendChild(clickedElement)

  Flip.from(state, {
    absolute: true,
    duration: 1.2,
    ease: 'power3.inOut',
  })
}

const unFlipProduct = () => {
  if (!currentPaintingElement.value || !originalParent.value) return

  const state = Flip.getState(currentPaintingElement.value)

  originalParent.value.appendChild(currentPaintingElement.value)

  Flip.from(state, {
    absolute: true,
    duration: 1.2,
    ease: 'power3.inOut',
    onComplete: () => {
      currentPaintingElement.value = null
      originalParent.value = null
    },
  })
}

// Animations functions
const animateTexts = (paintingId) => {
  const title = document.querySelector(`[data-title="${paintingId}"]`)
  const description = document.querySelector(`[data-desc="${paintingId}"]`)

  if (title) {
    const splitTitle = new SplitText(title, {
      type: 'lines, chars',
      charsClass: 'char',
    })

    gsap.fromTo(
      title.querySelectorAll('.char'),
      { y: '100%' },
      {
        y: 0,
        duration: 1.1,
        delay: 0.4,
        ease: 'power3.inOut',
        stagger: 0.025,
      },
    )
  }

  if (description) {
    const splitDesc = new SplitText(description, {
      type: 'lines',
      linesClass: 'line',
    })
    gsap.fromTo(
      description.querySelectorAll('.line'),
      { y: '100%' },
      {
        y: 0,
        duration: 1.1,
        delay: 0.6,
        ease: 'power3.inOut',
        stagger: 0.05,
      },
    )
  }
}

const handleMouseMove = (e) => {
  if (!showDetails.value) return

  const cross = paintingDetailsRef.value?.crossRef
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
}
</style>
