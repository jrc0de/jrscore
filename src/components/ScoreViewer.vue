<template>
  <div class="viewer-container">
    <div v-if="totalPages > 1" class="page-controls">
      <button @click="previousPage" :disabled="currentPage === 1" class="icon-btn">
        <i>chevron_left</i>
      </button>

      <span class="page-indicator"> {{ currentPage }} / {{ totalPages }} </span>

      <button @click="nextPage" :disabled="currentPage === totalPages" class="icon-btn">
        <i>chevron_right</i>
      </button>
    </div>

    <div class="notation-wrapper">
      <div ref="notationRef" class="notation" :class="{ 'has-score': hasScore }" />
    </div>
  </div>
</template>

<script setup lang="js">
import { ref, onMounted, watch } from 'vue'
import createVerovioModule from 'verovio/wasm'
import { VerovioToolkit } from 'verovio/esm'
const notationRef = ref(null)
const hasScore = ref(false)
const currentPage = ref(1)
const totalPages = ref(1)
let toolkit = null
let currentMei = null
const props = defineProps({
  format: {
    type: String,
    default: 'adjusted',
  },

  layoutOptions: {
    type: Object,

    default: () => ({
      pageMarginTop: 50,
      pageMarginBottom: 50,
      pageMarginLeft: 50,
      pageMarginRight: 50,

      scale: 30,

      staffSpacing: 8,
    }),
  },
})

const FORMAT_OPTIONS = {
  adjusted: {
    breaks: 'none',
    adjustPageHeight: true,
    adjustPageWidth: true,
    footer: 'none',
    header: 'none',
  },

  'a4-portrait': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 2100,
    pageHeight: 2970,
    footer: 'none',
    header: 'none',
  },

  'a4-landscape': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 2970,
    pageHeight: 2100,
    footer: 'none',
    header: 'none',
  },

  'a5-portrait': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 1480,
    pageHeight: 2100,
    footer: 'none',
    header: 'none',
  },

  'a5-landscape': {
    breaks: 'auto',
    adjustPageHeight: false,
    adjustPageWidth: false,
    pageWidth: 2100,
    pageHeight: 1480,
    footer: 'none',
    header: 'none',
  },
}

onMounted(async () => {
  const VerovioModule = await createVerovioModule()
  toolkit = new VerovioToolkit(VerovioModule)
})

function getVerovioOptions() {
  return {
    footer: 'none',
    header: 'none',
    ...FORMAT_OPTIONS[props.format],
    pageMarginTop: props.layoutOptions.pageMarginTop,
    pageMarginBottom: props.layoutOptions.pageMarginBottom,
    pageMarginLeft: props.layoutOptions.pageMarginLeft,
    pageMarginRight: props.layoutOptions.pageMarginRight,
    scale: props.layoutOptions.scale,
    spacingStaff: props.layoutOptions.staffSpacing,
  }
}

function applyOptionsAndRender() {
  if (!toolkit || !currentMei) return
  toolkit.setOptions(getVerovioOptions())
  toolkit.loadData(currentMei)
  totalPages.value = toolkit.getPageCount()
  if (currentPage.value > totalPages.value) {
    currentPage.value = totalPages.value
  }
  renderPage()
}

function renderPage() {
  const svg = toolkit.renderToSVG(currentPage.value)
  notationRef.value.innerHTML = svg
  hasScore.value = true
}

function renderMei(mei) {
  currentMei = mei
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

watch(currentPage, () => {
  if (toolkit && currentMei) renderPage()
})

watch(
  () => props.format,

  () => {
    applyOptionsAndRender()
  },
)

watch(
  () => props.layoutOptions,
  () => {
    applyOptionsAndRender()
  },
  {
    deep: true,
  },
)

defineExpose({
  renderMei,
  currentPage,
  totalPages,
})
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
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  padding: 0.5rem;
}

.icon-btn {
  padding: 0.4rem;
}

.notation-wrapper {
  flex: 1;
  min-height: 0;
  display: flex;
  justify-content: center;
  justify-content: safe center;
  align-items: center;
  align-items: safe center;
  overflow: auto;
}

.notation.has-score {
  background: white;
  padding: 0.75rem;
  box-sizing: border-box;
  flex-shrink: 0;
}

.notation :deep(svg) {
  display: block;
}

.notation :deep(svg *) {
  fill: black !important;
  stroke: black !important;
}
</style>
