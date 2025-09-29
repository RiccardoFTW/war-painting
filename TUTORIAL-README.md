# 🎨 Tutorial: Da JavaScript Vanilla a Vue.js con GSAP

## 📋 Indice

1. [Introduzione](#introduzione)
2. [Concetti Fondamentali](#concetti-fondamentali)
3. [Struttura del Progetto](#struttura-del-progetto)
4. [Step by Step Implementation](#step-by-step-implementation)
5. [Codice Completo](#codice-completo)
6. [Concetti Avanzati](#concetti-avanzati)
7. [Debugging e Problemi Comuni](#debugging-e-problemi-comuni)

---

## 🎯 Introduzione

Questo tutorial documenta la migrazione di una classe JavaScript vanilla che gestiva una galleria interattiva con drag & drop verso un'implementazione Vue.js moderna.

### Obiettivo Originale

Convertire una classe Grid JavaScript che gestiva:

- Centraggio automatico della griglia
- Drag & drop con GSAP Draggable
- Animazioni fluide
- Controllo con scroll wheel
- Sistema di dettagli prodotti

---

## 🧠 Concetti Fondamentali

### 1. **Template Refs in Vue**

I **template refs** sono il modo di Vue per accedere agli elementi DOM, sostituendo `document.querySelector()`.

#### JavaScript Vanilla (PRIMA):

```javascript
const grid = document.querySelector('.grid') // Cerca nel DOM
```

#### Vue con Template Refs (DOPO):

```javascript
// Nel <script setup>
const gridRef = ref(null)  // Crea una "scatola" vuota

// Nel template
<div class="grid" ref="gridRef">  // Collega l'elemento alla "scatola"

// Uso
gridRef.value  // Contiene l'elemento HTML
```

**Analogia**: I refs sono come **cassette postali**. Prima crei la cassetta (`ref(null)`), poi Vue ci "consegna" l'elemento quando è pronto.

### 2. **Lifecycle Hooks: onMounted**

`onMounted` è il momento in cui Vue ha finito di creare tutti gli elementi HTML.

```javascript
import { onMounted, nextTick } from 'vue'

onMounted(async () => {
  await nextTick() // Aspetta che tutto il DOM sia completamente pronto

  // Ora possiamo accedere agli elementi
  console.log(gridRef.value) // <div class="grid">...</div>
})
```

**Timing importante**:

1. Vue esegue `<script setup>`
2. Vue crea il template HTML
3. `onMounted` viene chiamato
4. `nextTick` garantisce che tutto sia renderizzato

### 3. **defineExpose: Comunicazione tra Componenti**

`defineExpose` permette a un componente figlio di "esporre" i suoi refs al componente genitore.

#### Componente Figlio (MainGrid.vue):

```javascript
const gridRef = ref(null)

defineExpose({
  gridRef, // "Espone" il ref al genitore
})
```

#### Componente Genitore (MainContainer.vue):

```javascript
const mainGridRef = ref(null) // Ref per il componente figlio

// Accesso al ref esposto
const grid = mainGridRef.value.gridRef // L'elemento DOM della griglia
```

### 4. **GSAP Integration in Vue**

GSAP funziona perfettamente con Vue, ma bisogna:

1. Installare GSAP: `npm install gsap`
2. Importare i plugin necessari
3. Registrare i plugin
4. Usare i refs invece di querySelector

```javascript
import gsap from 'gsap'
import { Draggable } from 'gsap/all'

gsap.registerPlugin(Draggable)

// Uso con refs
gsap.set(gridRef.value, { x: 100, y: 50 })
```

---

## 🏗️ Struttura del Progetto

```
src/
├── components/
│   ├── MainContainer.vue     # Container principale + logica drag
│   ├── MainGrid.vue         # Layout griglia + ref esposto
│   ├── MainColumn.vue       # Colonna singola
│   └── PaintingImg.vue      # Immagine singola
├── composables/             # (Futuro: logica riutilizzabile)
└── utils/                   # (Futuro: funzioni helper)
```

### Separazione delle Responsabilità

| Componente        | Responsabilità                                   |
| ----------------- | ------------------------------------------------ |
| **MainContainer** | Stato globale, drag logic, animazioni principali |
| **MainGrid**      | Layout griglia, esposizione refs                 |
| **MainColumn**    | Layout colonna                                   |
| **PaintingImg**   | Rendering immagine                               |

---

## 🚀 Step by Step Implementation

### Step 1: Preparazione Base

#### 1.1 Installazione GSAP

```bash
npm install gsap
```

#### 1.2 Aggiunta Template Refs

**MainGrid.vue:**

```vue
<script setup>
import { ref, defineExpose } from 'vue'

const gridRef = ref(null)

defineExpose({
  gridRef,
})
</script>

<template>
  <div class="grid" ref="gridRef">
    <!-- contenuto -->
  </div>
</template>
```

**MainContainer.vue:**

```vue
<script setup>
import { ref } from 'vue'
import MainGrid from './MainGrid.vue'

const containerRef = ref(null)
const mainGridRef = ref(null)
</script>

<template>
  <div class="container" ref="containerRef">
    <MainGrid ref="mainGridRef" />
  </div>
</template>
```

### Step 2: Implementazione Centraggio

#### Logica di Centraggio

```javascript
const centerGrid = () => {
  const grid = mainGridRef.value.gridRef // Accesso al DOM element

  if (!grid) return // Controllo di sicurezza

  // Calcolo dimensioni
  const gridWidth = grid.offsetWidth
  const gridHeight = grid.offsetHeight
  const windowWidth = window.innerWidth
  const windowHeight = window.innerHeight

  // Calcolo centro
  const centerX = (windowWidth - gridWidth) / 2
  const centerY = (windowHeight - gridHeight) / 2

  // Animazione GSAP
  gsap.set(grid, { x: centerX, y: centerY })
}
```

#### Chiamata nel Lifecycle

```javascript
import { onMounted, nextTick } from 'vue'

onMounted(async () => {
  await nextTick() // Aspetta che il DOM sia pronto
  centerGrid()
})
```

### Step 3: Implementazione Drag & Drop

#### Import e Setup GSAP

```javascript
import gsap from 'gsap'
import { Draggable } from 'gsap/all'

gsap.registerPlugin(Draggable)
```

#### Stato di Dragging

```javascript
const isDragging = ref(false)
```

#### Logica Draggable

```javascript
const setupDraggable = () => {
  const grid = mainGridRef.value.gridRef

  if (!grid) return

  Draggable.create(grid, {
    type: 'x,y', // Movimento in entrambe le direzioni
    bounds: {
      // Limiti di movimento
      minX: -(grid.offsetWidth - window.innerWidth) - 200,
      maxX: 200,
      minY: -(grid.offsetHeight - window.innerHeight) - 200,
      maxY: 100,
    },
    inertia: true, // Inerzia quando rilasci
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
```

#### Chiamata Completa

```javascript
onMounted(async () => {
  await nextTick()
  centerGrid() // Prima centra
  setupDraggable() // Poi abilita il drag
})
```

---

## 💻 Codice Completo

### MainContainer.vue

```vue
<script setup>
import { ref, onMounted, nextTick } from 'vue'
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

onMounted(async () => {
  await nextTick()
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
  overflow: hidden; /* Nasconde overflow */
}
</style>
```

### MainGrid.vue

```vue
<script setup>
import { ref, defineExpose } from 'vue'
import MainColumn from './MainColumn.vue'

const gridRef = ref(null)

defineExpose({
  gridRef,
})
</script>

<template>
  <div class="grid" ref="gridRef">
    <MainColumn />
    <MainColumn />
    <MainColumn />
    <MainColumn />
    <MainColumn />
  </div>
</template>

<style scoped>
.grid {
  position: absolute;
  width: max-content;
  height: max-content;
  display: flex;
  gap: 5vw;
}
</style>
```

---

## 🎓 Concetti Avanzati

### 1. **Reactive State Management**

```javascript
// Stato reattivo
const isDragging = ref(false)

// Computed per stati derivati
const gridClasses = computed(() => ({
  grid: true,
  '--is-dragging': isDragging.value,
}))

// Watcher per reagire ai cambiamenti
watch(isDragging, (newValue) => {
  console.log('Dragging changed:', newValue)
})
```

### 2. **Composables (Futuro)**

Per logica riutilizzabile:

```javascript
// composables/useGridDrag.js
export function useGridDrag(gridRef) {
  const isDragging = ref(false)

  const setupDraggable = () => {
    // ... logica drag
  }

  return {
    isDragging,
    setupDraggable,
  }
}
```

### 3. **Performance Optimization**

```javascript
// Debounce per resize
const debouncedResize = debounce(() => {
  updateBounds()
}, 100)

window.addEventListener('resize', debouncedResize)
```

---

## 🐛 Debugging e Problemi Comuni

### 1. **Refs Non Trovati**

**Problema**: `gridRef.value` è `undefined`

**Soluzioni**:

```javascript
// ✅ Usa nextTick
onMounted(async () => {
  await nextTick()
  // ora i refs sono disponibili
})

// ✅ Controlli di sicurezza
const grid = mainGridRef.value?.gridRef
if (!grid) {
  console.log('Grid non trovato!')
  return
}

// ✅ Debug step by step
console.log('mainGridRef:', mainGridRef.value)
console.log('gridRef:', mainGridRef.value?.gridRef)
```

### 2. **defineExpose Non Funziona**

**Problema**: `mainGridRef.value.gridRef.value` è `undefined`

**Soluzione**: Vue "unwrappa" automaticamente i refs esposti

```javascript
// ❌ SBAGLIATO
const grid = mainGridRef.value.gridRef.value

// ✅ CORRETTO
const grid = mainGridRef.value.gridRef
```

### 3. **GSAP Plugin Non Trovato**

**Problema**: `Draggable is not defined`

**Soluzioni**:

```javascript
// ✅ Import corretto
import { Draggable } from 'gsap/all'
// oppure
import { Draggable } from 'gsap/Draggable'

// ✅ Registrazione plugin
gsap.registerPlugin(Draggable)
```

### 4. **CSS Overflow Issues**

**Problema**: La griglia va fuori dal viewport

**Soluzione**:

```css
.container {
  overflow: hidden; /* Nasconde tutto ciò che esce */
}

.grid {
  position: relative; /* Non absolute per il drag */
}
```

---

## 🚀 Prossimi Passi

### Funzionalità da Implementare:

1. **Scroll Wheel Control** - Controllo con rotellina mouse
2. **Image Animations** - Animazioni hover e intersection observer
3. **Details System** - Pannello dettagli prodotti
4. **Responsive Design** - Adattamento mobile
5. **Performance Optimization** - Lazy loading e debouncing

### Refactoring Suggeriti:

1. **Composables** - Estrarre logica riutilizzabile
2. **Store Management** - Pinia per stato globale complesso
3. **Component Props** - Rendere componenti più flessibili
4. **TypeScript** - Aggiungere type safety

---

## 🎯 Conclusioni

Abbiamo imparato:

✅ **Template Refs** - Accesso agli elementi DOM in Vue  
✅ **Component Communication** - defineExpose per refs condivisi  
✅ **GSAP Integration** - Animazioni professionali in Vue  
✅ **Lifecycle Management** - onMounted e nextTick  
✅ **Debugging Techniques** - Come risolvere problemi comuni  
✅ **Architecture Patterns** - Separazione responsabilità

**Risultato**: Una griglia interattiva completamente funzionale con drag & drop, centraggio automatico e architettura Vue moderna! 🎨

---

_Tutorial creato durante la migrazione da JavaScript vanilla a Vue.js_
