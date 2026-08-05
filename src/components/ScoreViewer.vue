<template>
  <div class="viewer-container">
    <!-- Contrôles de navigation (uniquement en mode multipage) -->
    <div v-if="totalPages > 1" class="page-controls">
      <button @click="previousPage" :disabled="currentPage === 1" class="icon-btn">
        <i>chevron_left</i>
      </button>

      <span class="page-indicator">{{ currentPage }} / {{ totalPages }}</span>

      <button @click="nextPage" :disabled="currentPage === totalPages" class="icon-btn">
        <i>chevron_right</i>
      </button>
    </div>

    <!-- Zone d'affichage de la partition -->
    <div ref="wrapperRef" class="notation-wrapper">
      <div
        ref="notationRef"
        class="notation"
        :class="{ 'has-score': hasScore, 'fixed-page': format !== 'adjusted' }"
        :style="pageStyle"
      ></div>
    </div>
  </div>
</template>

<script setup lang="js">
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'
import createVerovioModule from 'verovio/wasm'
import { VerovioToolkit } from 'verovio/esm'

const wrapperRef = ref(null)
const notationRef = ref(null)
const hasScore = ref(false)
const boxSize = ref(null)
const currentPage = ref(1)
const totalPages = ref(1)

let toolkit = null
let currentMei = null
let resizeObserver = null

const props = defineProps({
  meiContent: { type: String, default: null },
  format: { type: String, default: 'adjusted' },
})

const FORMAT_OPTIONS = {
  adjusted: {
    breaks: 'none',
    adjustPageHeight: true,
    adjustPageWidth: true,
    pageMarginTop: 10,
    pageMarginBottom: 10,
    pageMarginLeft: 10,
    pageMarginRight: 10,
    footer: 'none',
    header: 'none',
  },
  'a4-portrait': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 2100,
    pageHeight: 2970,
    pageMarginTop: 50,
    pageMarginBottom: 50,
    pageMarginLeft: 50,
    pageMarginRight: 50,
    footer: 'none',
    header: 'none',
  },
  'a4-landscape': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 2970,
    pageHeight: 2100,
    pageMarginTop: 50,
    pageMarginBottom: 50,
    pageMarginLeft: 50,
    pageMarginRight: 50,
    footer: 'none',
    header: 'none',
  },
  'a5-portrait': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 1480,
    pageHeight: 2100,
    pageMarginTop: 40,
    pageMarginBottom: 40,
    pageMarginLeft: 40,
    pageMarginRight: 40,
    footer: 'none',
    header: 'none',
  },
  'a5-landscape': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 2100,
    pageHeight: 1480,
    pageMarginTop: 40,
    pageMarginBottom: 40,
    pageMarginLeft: 40,
    pageMarginRight: 40,
    footer: 'none',
    header: 'none',
  },
}

const pageStyle = computed(() => {
  if (!boxSize.value) return {}
  return {
    width: `${boxSize.value.width}px`,
    height: `${boxSize.value.height}px`,
  }
})

function updateBoxSize() {
  if (!wrapperRef.value) return

  if (props.format === 'adjusted') {
    boxSize.value = null
    return
  }

  const opts = FORMAT_OPTIONS[props.format]
  const ratio = opts.pageWidth / opts.pageHeight

  const cw = wrapperRef.value.clientWidth
  const ch = wrapperRef.value.clientHeight

  let w = cw
  let h = w / ratio

  if (h > ch) {
    h = ch
    w = h * ratio
  }

  boxSize.value = { width: w, height: h }
}

onMounted(async () => {
  const VerovioModule = await createVerovioModule()
  toolkit = new VerovioToolkit(VerovioModule)
  toolkit.setOptions({ footer: 'none', ...FORMAT_OPTIONS[props.format] })

  updateBoxSize()
  resizeObserver = new ResizeObserver(() => updateBoxSize())
  resizeObserver.observe(wrapperRef.value)

  if (props.meiContent) renderMei(props.meiContent)
})

onUnmounted(() => {
  if (resizeObserver) resizeObserver.disconnect()
})

function applyOptionsAndRender() {
  if (!toolkit || !currentMei) return
  toolkit.setOptions({ footer: 'none', ...FORMAT_OPTIONS[props.format] })
  toolkit.loadData(currentMei)

  // Obtenir le nombre total de pages
  totalPages.value = toolkit.getPageCount()

  // S'assurer que currentPage ne dépasse pas le nombre de pages
  if (currentPage.value > totalPages.value) {
    currentPage.value = totalPages.value
  }
  if (currentPage.value < 1) {
    currentPage.value = 1
  }

  // Rendre la page actuelle
  const svg = toolkit.renderToSVG(currentPage.value)
  notationRef.value.innerHTML = svg

  const svgEl = notationRef.value.querySelector('svg')
  if (svgEl) {
    svgEl.removeAttribute('width')
    svgEl.removeAttribute('height')
  }

  hasScore.value = true
}

function renderMei(meiString) {
  currentMei = meiString
  currentPage.value = 1
  applyOptionsAndRender()
}

function nextPage() {
  if (currentPage.value < totalPages.value) {
    currentPage.value++
  }
}

function previousPage() {
  if (currentPage.value > 1) {
    currentPage.value--
  }
}

function clampPage() {
  if (currentPage.value < 1) currentPage.value = 1
  if (currentPage.value > totalPages.value) currentPage.value = totalPages.value
}

// Watcher pour les changements de page
watch(currentPage, () => {
  if (currentMei && toolkit) {
    const svg = toolkit.renderToSVG(currentPage.value)
    notationRef.value.innerHTML = svg

    const svgEl = notationRef.value.querySelector('svg')
    if (svgEl) {
      svgEl.removeAttribute('width')
      svgEl.removeAttribute('height')
    }
  }
})

// Watcher pour les changements de format
watch(
  () => props.format,
  async () => {
    updateBoxSize()
    await nextTick()
    if (currentMei) applyOptionsAndRender()
  },
)

defineExpose({ renderMei, currentPage, totalPages })
</script>

<style scoped>
.viewer-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.page-controls {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  padding: 0.5rem;
  border-bottom: 1px solid var(--border-color);
}

.icon-btn {
  padding: 0.4rem !important;
  min-width: auto !important;
}

.icon-btn:disabled {
  opacity: 0.4;
}

.page-indicator {
  font-size: 0.9rem;
  font-weight: 500;
  min-width: 70px;
  text-align: center;
}

.notation-wrapper {
  flex: 1;
  min-width: 0;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.notation.has-score {
  background: white;
  padding: 0.75rem;
  max-width: 100%;
  max-height: 100%;
  min-width: 0;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  box-sizing: border-box;
}

.notation.has-score :deep(svg) {
  max-width: 100%;
  max-height: 100%;
  width: auto;
  height: auto;
  display: block;
}

.notation.fixed-page {
  overflow: hidden;
  padding: 0;
}

.notation.fixed-page :deep(svg) {
  width: 100%;
  height: 100%;
  max-width: none;
  max-height: none;
}

.notation :deep(svg) {
  color: black;
}

.notation :deep(svg *) {
  fill: black !important;
  stroke: black !important;
}
</style>
