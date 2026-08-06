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

    <div ref="wrapperRef" class="notation-wrapper">
      <div
        ref="notationRef"
        class="notation"
        :class="{
          'has-score': hasScore,
          'fixed-page': format !== 'adjusted',
        }"
        :style="pageStyle"
      />
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

      scale: 40,

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

  boxSize.value = {
    width: w,
    height: h,
  }
}

onMounted(async () => {
  const VerovioModule = await createVerovioModule()

  toolkit = new VerovioToolkit(VerovioModule)

  updateBoxSize()

  resizeObserver = new ResizeObserver(updateBoxSize)

  resizeObserver.observe(wrapperRef.value)
})

onUnmounted(() => {
  if (resizeObserver) resizeObserver.disconnect()
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

  const svgEl = notationRef.value.querySelector('svg')

  if (svgEl) {
    svgEl.removeAttribute('width')

    svgEl.removeAttribute('height')
  }

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

  async () => {
    updateBoxSize()

    await nextTick()

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

  align-items: center;

  overflow: hidden;
}

.notation.has-score {
  background: white;

  padding: 0.75rem;

  display: flex;

  justify-content: center;

  align-items: center;

  box-sizing: border-box;
}

.notation :deep(svg) {
  max-width: 100%;

  max-height: 100%;

  display: block;
}

.notation.fixed-page :deep(svg) {
  width: 100%;

  height: 100%;
}

.notation :deep(svg *) {
  fill: black !important;

  stroke: black !important;
}
</style>
