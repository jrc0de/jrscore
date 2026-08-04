<template>
  <div ref="wrapperRef" class="notation-wrapper">
    <div
      ref="notationRef"
      class="notation"
      :class="{ 'has-score': hasScore, 'fixed-page': format !== 'adjusted' }"
      :style="pageStyle"
    ></div>
  </div>
</template>

<script setup lang="js">
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'
import createVerovioModule from 'verovio/wasm'
import { VerovioToolkit } from 'verovio/esm'

const wrapperRef = ref(null)
const notationRef = ref(null)
const hasScore = ref(false)
const boxSize = ref(null) // { width, height } en px, null = taille naturelle (mode "adjusté")
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
    boxSize.value = null // la boîte s'adapte librement au contenu
    return
  }

  const opts = FORMAT_OPTIONS[props.format]
  const ratio = opts.pageWidth / opts.pageHeight // largeur / hauteur

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
  const svg = toolkit.renderToSVG(1)
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
  applyOptionsAndRender()
}

watch(
  () => props.format,
  async () => {
    updateBoxSize()
    await nextTick()
    if (currentMei) applyOptionsAndRender()
  },
)

defineExpose({ renderMei })
</script>

<style scoped>
.notation-wrapper {
  width: 100%;
  height: 100%;
  min-width: 0;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

/* Mode "Ajusté" : la boîte s'adapte au contenu, comme avant */
.notation.has-score {
  background: white;
  padding: 1rem;
  border-radius: 8px;
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

/* Formats fixes (A4/A5) : la boîte a une taille imposée (calculée en JS)
   qui respecte exactement le ratio largeur/hauteur de la page */
.notation.fixed-page {
  overflow: hidden;
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
