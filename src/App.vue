<template>
  <main class="app-layout">
    <nav class="primary header">
      <img src="@/assets/feather.svg" alt="JRScore" class="logo" />

      <div class="title-area">
        <h5>JRScore</h5>
        <small>Convertisseur MEI / SVG</small>
      </div>
    </nav>

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
              <input type="radio" name="format" value="adjusted" v-model="selectedFormat" />
              <span>Ajusté</span>
            </label>
            <label class="radio">
              <input type="radio" name="format" value="a4-portrait" v-model="selectedFormat" />
              <span>A4 portrait</span>
            </label>
            <label class="radio">
              <input type="radio" name="format" value="a4-landscape" v-model="selectedFormat" />
              <span>A4 paysage</span>
            </label>
            <label class="radio">
              <input type="radio" name="format" value="a5-portrait" v-model="selectedFormat" />
              <span>A5 portrait</span>
            </label>
            <label class="radio">
              <input type="radio" name="format" value="a5-landscape" v-model="selectedFormat" />
              <span>A5 paysage</span>
            </label>
          </div>
        </fieldset>

        <fieldset>
          <legend>Réglages</legend>
          <RangeSlider title="Param. 1" />
          <RangeSlider title="Param. 2" />
          <RangeSlider title="Param. 3" />
        </fieldset>

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
        <ScoreViewer ref="scoreViewerRef" :format="selectedFormat" />
      </section>
    </div>
  </main>
</template>

<script setup lang="js">
import { ref } from 'vue'
import JSZip from 'jszip'
import RangeSlider from '@/components/RangeSlider.vue'
import ScoreViewer from '@/components/ScoreViewer.vue'

const scoreViewerRef = ref(null)
const fileInput = ref(null)
const selectedFormat = ref('adjusted')
const hasScore = ref(false)
const currentFileName = ref('score')
const exporting = ref(false)

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

// ===== EXPORT SVG (AUTO-DÉTECTION MONO/MULTI) =====
async function exportSVGMono() {
  const viewer = scoreViewerRef.value

  // Si multipage, exporter tous les SVG en ZIP
  if (viewer.totalPages > 1) {
    exportSVGMulti()
    return
  }

  // Sinon, exporter le SVG simple
  const svg = document.querySelector('.notation svg')
  if (!svg) return

  const svgString = new XMLSerializer().serializeToString(svg)
  const blob = new Blob([svgString], { type: 'image/svg+xml' })
  downloadBlob(blob, `${currentFileName.value}.svg`)
}

// ===== EXPORT SVG MULTIPAGE =====
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

      const svg = document.querySelector('.notation svg')
      if (svg) {
        const svgString = new XMLSerializer().serializeToString(svg)
        const pageNum = String(page).padStart(3, '0')
        zip.file(`page-${pageNum}.svg`, svgString)
      }
    }

    const blob = await zip.generateAsync({ type: 'blob' })
    downloadBlob(blob, `${currentFileName.value}-all-pages.zip`)
  } finally {
    viewer.currentPage = originalPage
    await new Promise((resolve) => setTimeout(resolve, 200))
    exporting.value = false
  }
}

// ===== UTILITAIRE =====
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
.header {
  padding: 0 1.5rem;
  min-height: 72px;
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo {
  width: 42px;
  height: 42px;
  object-fit: contain;
}

.title-area {
  display: flex;
  flex-direction: column;
  justify-content: center;
  line-height: 1.1;
}

.title-area h5 {
  margin: 0;
  font-weight: 800;
  letter-spacing: -0.03em;
  font-size: 1.35rem;
}

.title-area small {
  margin-top: 0.25rem;
  opacity: 0.65;
  font-size: 0.85rem;
  font-style: italic;
}

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
