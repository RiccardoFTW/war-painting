<script setup>
import { ref, onMounted } from 'vue'
import MainGrid from './MainGrid.vue'
import { Draggable } from 'gsap/all'
import gsap from 'gsap'
gsap.registerPlugin(Draggable)

const containerRef = ref(null)
const mainGridRef = ref(null)
const isDragging = ref(false)

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

onMounted(() => {
  centerGrid()
  setupDraggable()
})
</script>

<template>
  <div class="container" ref="containerRef">
    <MainGrid ref="mainGridRef" />
  </div>
</template>

<style scoped>
.container {
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
}
</style>
