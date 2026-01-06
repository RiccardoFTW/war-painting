<!--
  PaintingDetails.vue - Pannello dettagli dipinto

  Mostra:
  - Thumbnail del dipinto (animata con FLIP)
  - Titolo del dipinto
  - Metadati: artista, anno, location
  - Descrizione

  Include anche il cursore "cross" per chiudere
-->
<template>
  <!-- Pannello dettagli -->
  <div class="details" :class="{ '--is-showing': isVisible }">
    <!-- Container per l'immagine (target del FLIP) -->
    <div class="details__thumb"></div>

    <div class="details__content">
      <!-- Titolo -->
      <div class="details__title">
        <p :data-title="currentPainting?.id">{{ currentPainting?.title }}</p>
      </div>

      <!-- Metadati -->
      <div class="details__meta">
        <p class="meta__artist">{{ currentPainting?.artist }}</p>
        <p class="meta__year">{{ currentPainting?.year }}</p>
        <p class="meta__location">{{ currentPainting?.Location }}</p>
      </div>

      <!-- Descrizione -->
      <div class="details__body">
        <p :data-desc="currentPainting?.id">{{ currentPainting?.description }}</p>
      </div>
    </div>
  </div>

  <!-- Cursore cross per chiudere (segue il mouse) -->
  <div class="cross" @click="closeDetails">
    <span></span>
    <span></span>
  </div>
</template>

<script setup>
// ===== PROPS =====
defineProps({
  isVisible: {
    type: Boolean,
    default: false,
  },
  currentPainting: {
    type: Object,
    default: null,
  },
})

// ===== EVENTI =====
const emit = defineEmits(['close'])

// ===== METODI =====
/** Emette evento per chiudere il pannello */
const closeDetails = () => {
  emit('close')
}
</script>

<style scoped>
/* ===== PANNELLO DETTAGLI ===== */
.details {
  position: fixed;
  top: 0;
  right: 0;
  width: 50vw;
  height: 100vh;
  padding: 3vw 4vw;
  background-color: #f1f1f1;
  opacity: 1;
  transform: translate3d(100%, 0, 0); /* Nascosto fuori schermo */
  z-index: 9999;
  will-change: transform;

  @media (max-width: 1024px) {
    width: 60vw;
    padding: 4vw 5vw;
  }

  @media (max-width: 768px) {
    width: 90%;
    padding: 6vw 5vw;
  }
}

/* Container thumbnail - qui viene spostata l'immagine con FLIP */
.details__thumb {
  position: relative;
  width: 100%;
  height: 50vh;
  margin-bottom: 2rem;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;

  @media (max-width: 1024px) {
    height: 45vh;
  }

  @media (max-width: 768px) {
    height: 35vh;
    margin-bottom: 1.5rem;
  }
}

.details__thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* ===== TITOLO ===== */
.details__title {
  font-family: 'EB Garamond', serif;
  font-style: italic;
  margin-bottom: 2vw;
  position: relative;
  display: grid;
  color: #c49852;

  @media (max-width: 768px) {
    margin-bottom: 3vw;
  }
}

.details__title p {
  grid-area: 1 / -1;
  overflow: hidden;
  font-size: 3vw;

  @media (max-width: 1024px) {
    font-size: clamp(1.5rem, 3.5vw, 2.5rem);
  }

  @media (max-width: 768px) {
    font-size: clamp(1.3rem, 5vw, 2rem);
  }
}

/* ===== METADATI ===== */
.details__meta {
  font-family: 'EB Garamond', serif;
  margin-bottom: 2vw;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.details__meta p {
  font-size: clamp(0.9rem, 1.2vw, 1.1rem);
}

.meta__artist {
  font-weight: 600;
}

.meta__year {
  font-style: italic;
  opacity: 0.8;
}

.meta__location {
  opacity: 0.7;
  font-size: clamp(0.8rem, 1vw, 1rem);
}

/* ===== DESCRIZIONE ===== */
.details__body {
  display: flex;
  flex-wrap: wrap;
  align-items: end;
  gap: 2vw;

  @media (max-width: 768px) {
    gap: 1rem;
  }
}

.details__body p {
  overflow: hidden;
  font-family: 'EB Garamond', serif;
  font-size: clamp(0.95rem, 1.1vw, 1.05rem);
  line-height: 1.6;
  color: #000;

  @media (max-width: 768px) {
    font-size: clamp(0.9rem, 3.5vw, 1rem);
    line-height: 1.5;
  }
}

/* ===== CURSORE CROSS ===== */
.cross {
  position: fixed;
  top: 0;
  left: 0;
  width: 40px;
  height: 40px;
  cursor: pointer;
  z-index: 10000;
  transform: scale(0); /* Nascosto inizialmente */
  pointer-events: none;

  @media (max-width: 768px) {
    width: 50px;
    height: 50px;
  }
}

/* Linee del cross */
.cross span {
  position: absolute;
  width: 100%;
  height: 2px;
  background: #c49852;
  top: 50%;
  left: 0;
}

.cross span:first-child {
  transform: rotate(45deg);
}

.cross span:last-child {
  transform: rotate(-45deg);
}
</style>
