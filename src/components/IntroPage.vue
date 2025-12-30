<template>
  <div class="intro__container">
    <div class="element__container">
      <div class="columns__container">
        <div class="column">
          <p class="text-black">Diego Velàzquez</p>
          <p class="text-gold">Museo del Prado</p>
          <p class="text-black">Hendrick Vroom</p>
          <p class="text-gold">Rijks Museum</p>
          <p class="text-black">John Singleton Copley</p>
        </div>
        <div class="column">
          <p class="text-gold">Jersey Museum and Art Gallery</p>
          <p class="text-black">Théodore Géricault</p>
          <p class="text-gold">Musèe du Louvre</p>
          <p class="text-black">Jan Matejko</p>
          <p class="text-gold">National Museum</p>
        </div>
        <div class="column">
          <p class="text-black">Ivan Aivazovsky</p>
          <p class="text-gold">Aivazovsky Art Gallery</p>
          <p class="text-black">Otto Dix</p>
          <p class="text-gold">Otto Dix Haus</p>
          <p class="text-black">Jean-Louis-Ernest Meissonier</p>
        </div>
        <div class="column">
          <p class="text-gold">Meissonier's studio</p>
          <p class="text-black">Jean-Louis-Ernest Meissonier</p>
          <p class="text-gold">Musée d'Orsay, Paris</p>
          <p class="text-black">Elizabeth Thompson</p>
          <p class="text-gold">Leeds Art Gallery, Leeds</p>
        </div>
        <div class="column">
          <p class="text-black">Arturo Michelena</p>
          <p class="text-gold">Museo de Bellas Artes</p>
          <p class="text-black">Augusto Ferrer-Dalmau</p>
          <p class="text-gold">Private Collection</p>
          <p class="text-black">William Barnes Wollen</p>
        </div>
        <div class="column">
          <p class="text-gold">Chelmsford Museum</p>
          <p class="text-black">Albrecht Altdorfer</p>
          <p class="text-gold">Alte Pinakothek, Munich</p>
          <p class="text-black">Girodet de Roussy-Trioson</p>
          <p class="text-gold">Palace of Versailles</p>
        </div>
        <div class="column">
          <p class="text-black">Emanuel Leutze</p>
          <p class="text-gold">Metropolitan Museum of Art</p>
          <p class="text-black">Umberto Boccioni</p>
          <p class="text-gold">Museo del Novecento</p>
          <p class="text-black">Benjamin West</p>
        </div>
        <div class="column">
          <p class="text-gold">National Gallery of Canada</p>
          <p class="text-black">Benjamin West</p>
          <p class="text-gold">National Gallery of Art</p>
          <p class="text-black">Dominic Serres</p>
          <p class="text-gold">National Maritime Museum</p>
        </div>
      </div>
    </div>
    <div class="blum-overlay"></div>
    <div class="blum-overlay"></div>
    <div class="blum-overlay"></div>
  </div>
</template>

<script setup>
import { onMounted, nextTick } from 'vue'
import gsap from 'gsap'
import { SplitText } from 'gsap/all'

gsap.registerPlugin(SplitText)

const emit = defineEmits(['navigate'])

// ===== TEXT ANIMATIONS =====
const animateColumns = async () => {
  await nextTick()

  const isMobile = window.innerWidth <= 768

  const timeline = gsap.timeline({
    onComplete: () => {
      // Dopo che tutte le animazioni sono completate, transizione verso HeroPage
      setTimeout(() => {
        emit('navigate', 'hero')
      }, 600) // Piccolo delay prima della transizione
    },
  })

  const splits = []
  const columns = document.querySelectorAll('.column')

  columns.forEach((column, columnIndex) => {
    // Su mobile, salta le colonne nascoste (dalla 5 in poi)
    if (isMobile && columnIndex >= 4) {
      return
    }

    const texts = column.querySelectorAll('p')

    texts.forEach((text, textIndex) => {
      gsap.set(text, { opacity: 1 })

      const split = new SplitText(text, {
        type: 'chars',
        charsClass: 'char',
      })
      splits.push(split)

      // Calcola il timing: ogni colonna inizia dopo la precedente, ogni testo nella colonna è leggermente sfalsato
      // Su mobile, riduciamo il delay tra colonne per velocizzare l'animazione
      const columnDelay = isMobile ? 0.3 : 0.5
      const startTime =
        columnIndex * columnDelay + // Delay tra colonne
        textIndex * 0.1 // Delay tra testi nella stessa colonna

      timeline.fromTo(
        split.chars,
        {
          y: '120%',
          opacity: 0,
        },
        {
          y: 0,
          opacity: 1,
          duration: 1,
          ease: 'power3.in',
          stagger: 0.06,
        },
        startTime,
      )
    })
  })
}

onMounted(async () => {
  await nextTick()
  animateColumns()
})
</script>

<style scoped>
.intro__container {
  position: fixed;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.element__container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 0.3rem 0;
  gap: 2rem;
  width: 100%;
  pointer-events: auto;
}

.columns__container {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: center;
  align-items: flex-start;
  gap: 2rem;
  width: 100%;
  padding: 0 0.8rem;
}

.column {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex: 1;
}

.column p {
  margin: 0;
  font-family: 'EB Garamond', 'Serif';
  font-style: italic;
  font-weight: 300;
  font-size: clamp(0.75rem, 0.8vw, 0.875rem);
  line-height: 1.6;
  margin-bottom: 0.5rem;
  opacity: 0;
  white-space: nowrap;
}

.text-black {
  color: #000;
}

.text-gold {
  color: #c49852;
}

.char,
.line {
  transform: translate3d(0, 100%, 0);
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

@media (max-width: 768px) {
  .columns__container {
    flex-wrap: wrap;
    gap: 1.5rem;
    padding: 0 1rem;
    justify-content: center;
  }

  .column {
    flex: 0 0 calc(50% - 0.75rem);
    min-width: 0;
  }

  .column:nth-child(n + 5) {
    display: none;
  }

  .column p {
    font-size: clamp(0.7rem, 2vw, 0.85rem);
    white-space: normal;
    word-break: break-word;
    line-height: 1.4;
  }
}
</style>
