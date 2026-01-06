<!--
  PaintingImg.vue - Componente singolo dipinto

  Mostra un'immagine del dipinto con:
  - Effetto hover scale
  - Click handler per aprire i dettagli
  - Supporto touch per mobile
-->
<template>
  <div class="painting" ref="paintingRef" :data-id="paintingId" @click="handleClick">
    <img :src="src" :alt="alt" />
  </div>
</template>

<script setup>
// ===== IMPORTS =====
import { ref } from 'vue'

// ===== PROPS =====
const props = defineProps({
  src: {
    type: String,
    required: true,
  },
  alt: {
    type: String,
    required: true,
  },
  paintingId: {
    type: Number,
    required: true,
  },
  paintingData: {
    type: Object,
    required: true,
  },
})

// ===== EVENTI =====
const emit = defineEmits(['click'])

// ===== STATO =====
const paintingRef = ref(null)

// ===== METODI =====
/**
 * Gestisce il click/tap sul dipinto
 * Emette l'evento con i dati del dipinto e l'elemento per FLIP
 */
const handleClick = (e) => {
  e.stopPropagation()
  e.preventDefault()

  const imgElement = paintingRef.value?.querySelector('img')
  emit('click', {
    data: props.paintingData,
    element: imgElement || paintingRef.value,
  })
}
</script>

<style scoped>
/* Container dipinto */
.painting {
  position: relative;
  width: 18.5vw;
  aspect-ratio: 1 / 1;
  touch-action: manipulation; /* Migliora risposta touch */
  cursor: pointer;

  @media (max-width: 1024px) {
    width: 25vw;
  }

  @media (max-width: 768px) {
    width: 45vw;
  }
}

/* Immagine del dipinto */
.painting img {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: contain;
  will-change: transform;
  transition: transform 300ms ease-in;
}

/* Effetto hover */
.painting:hover img {
  transform: scale(1.05);
}
</style>
