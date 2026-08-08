<template>
  <main class="app-layout">
    <Header />

    <div class="app-body">
      <aside class="sidebar">
        <fieldset>
          <legend>Importer</legend>

          <div class="center-align">
            <button @click="triggerFileInput">
              <i>attach_file</i>
              <span>Fichier MEI</span>
            </button>

            <input
              ref="fileInput"
              type="file"
              accept=".mei"
              @change="onFileChange"
              style="display: none"
            />
          </div>
        </fieldset>

        <div class="small-space"></div>

        <fieldset>
          <legend>Format</legend>

          <div class="radio-group">
            <label class="radio">
              <input type="radio" value="adjusted" v-model="selectedFormat" />
              <span>Ajusté</span>
            </label>

            <label class="radio">
              <input type="radio" value="a4-portrait" v-model="selectedFormat" />
              <span>A4 portrait</span>
            </label>

            <label class="radio">
              <input type="radio" value="a4-landscape" v-model="selectedFormat" />
              <span>A4 paysage</span>
            </label>

            <label class="radio">
              <input type="radio" value="a5-portrait" v-model="selectedFormat" />
              <span>A5 portrait</span>
            </label>

            <label class="radio">
              <input type="radio" value="a5-landscape" v-model="selectedFormat" />
              <span>A5 paysage</span>
            </label>
          </div>
        </fieldset>

        <PageSettings v-model="layoutOptions" />

        <div class="small-space"></div>

        <fieldset>
          <legend>Exporter</legend>

          <div class="field">
            <button class="responsive" @click="exportSVGMono" :disabled="!hasScore">
              <i>image</i>
              <span>Format SVG</span>
            </button>
          </div>
        </fieldset>
      </aside>

      <section class="viewer-panel">
        <ScoreViewer
          ref="scoreViewerRef"
          :format="selectedFormat"
          :layout-options="layoutOptions"
        />
      </section>
    </div>
  </main>
</template>

<script setup lang="js">
import { ref, watch } from 'vue'
import JSZip from 'jszip'

import Header from '@/components/Header.vue'
import PageSettings from '@/components/PageSettings.vue'
import ScoreViewer from '@/components/ScoreViewer.vue'

const scoreViewerRef = ref(null)

const fileInput = ref(null)

const selectedFormat = ref('adjusted')

const hasScore = ref(false)

const currentFileName = ref('score')

const exporting = ref(false)

/*
  Tous les paramètres Verovio
  pilotés par les sliders
*/
const layoutOptions = ref({
  pageMarginTop: 10,

  pageMarginBottom: 10,

  pageMarginLeft: 10,

  pageMarginRight: 10,

  scale: 30,

  staffSpacing: 8,

  showMeasureNumbers: true,

  useEncodedBreaks: false,
})

/*
  Le mode "Ajusté" bascule automatiquement les marges à 10
  (comportement du script --crop). On mémorise les marges
  précédentes pour les restaurer en quittant ce mode.
  Le format par défaut étant "adjusted", on pré-charge ici
  les marges générales (50) à restaurer plus tard.
*/
let savedMargins = {
  pageMarginTop: 50,
  pageMarginBottom: 50,
  pageMarginLeft: 50,
  pageMarginRight: 50,
}

watch(selectedFormat, (newFormat, oldFormat) => {
  if (newFormat === 'adjusted' && oldFormat !== 'adjusted') {
    savedMargins = {
      pageMarginTop: layoutOptions.value.pageMarginTop,
      pageMarginBottom: layoutOptions.value.pageMarginBottom,
      pageMarginLeft: layoutOptions.value.pageMarginLeft,
      pageMarginRight: layoutOptions.value.pageMarginRight,
    }

    layoutOptions.value.pageMarginTop = 10
    layoutOptions.value.pageMarginBottom = 10
    layoutOptions.value.pageMarginLeft = 10
    layoutOptions.value.pageMarginRight = 10
  } else if (oldFormat === 'adjusted' && newFormat !== 'adjusted' && savedMargins) {
    layoutOptions.value.pageMarginTop = savedMargins.pageMarginTop
    layoutOptions.value.pageMarginBottom = savedMargins.pageMarginBottom
    layoutOptions.value.pageMarginLeft = savedMargins.pageMarginLeft
    layoutOptions.value.pageMarginRight = savedMargins.pageMarginRight

    savedMargins = null
  }
})

function triggerFileInput() {
  fileInput.value?.click()
}

function onFileChange(event) {
  const file = event.target.files[0]

  if (!file) return

  currentFileName.value = file.name.replace('.mei', '')

  const reader = new FileReader()

  reader.onload = (e) => {
    scoreViewerRef.value.renderMei(e.target.result)

    hasScore.value = true
  }

  reader.readAsText(file)
}

// ================================
// EXPORT SVG SIMPLE / MULTIPAGE
// ================================

function serializeNotationSvg() {
  const svg = document.querySelector('.notation svg')

  if (!svg) return null

  // On clone pour ne jamais toucher au SVG affiché à l'écran
  const clone = svg.cloneNode(true)

  if (!layoutOptions.value.showMeasureNumbers) {
    clone.querySelectorAll('.mNum.autogenerated').forEach((el) => el.remove())
  }

  return new XMLSerializer().serializeToString(clone)
}

async function exportSVGMono() {
  const viewer = scoreViewerRef.value

  if (viewer.totalPages > 1) {
    exportSVGMulti()

    return
  }

  const svgString = serializeNotationSvg()

  if (!svgString) return

  const blob = new Blob([svgString], {
    type: 'image/svg+xml',
  })

  downloadBlob(blob, `${currentFileName.value}.svg`)
}

async function exportSVGMulti() {
  exporting.value = true

  const zip = new JSZip()

  const viewer = scoreViewerRef.value

  const totalPages = viewer.totalPages

  const originalPage = viewer.currentPage

  try {
    for (let page = 1; page <= totalPages; page++) {
      viewer.currentPage = page

      await new Promise((resolve) => setTimeout(resolve, 200))

      const svgString = serializeNotationSvg()

      if (svgString) {
        const pageNum = String(page).padStart(3, '0')

        zip.file(`page-${pageNum}.svg`, svgString)
      }
    }

    const blob = await zip.generateAsync({
      type: 'blob',
    })

    downloadBlob(blob, `${currentFileName.value}-all-pages.zip`)
  } finally {
    viewer.currentPage = originalPage

    exporting.value = false
  }
}

function downloadBlob(blob, filename) {
  const url = URL.createObjectURL(blob)

  const link = document.createElement('a')

  link.href = url

  link.download = filename

  document.body.appendChild(link)

  link.click()

  document.body.removeChild(link)

  URL.revokeObjectURL(url)
}
</script>

<style scoped>
.app-layout {
  height: 100vh;

  display: flex;

  flex-direction: column;

  overflow: hidden;
}

.app-body {
  flex: 1;

  min-height: 0;

  display: flex;

  overflow: hidden;
}

.sidebar {
  width: 260px;

  flex-shrink: 0;

  overflow-y: auto;

  padding: 1rem;
}

.viewer-panel {
  flex: 1;

  min-width: 0;

  min-height: 0;

  overflow: hidden;
}

.radio-group {
  display: flex;

  flex-direction: column;

  gap: 0.5rem;
}
</style>
