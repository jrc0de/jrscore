<template>
  <main class="app-layout">
    <nav class="primary">
      <h1>JRScore</h1>
      <div class="max"></div>
      <a href="/about" class="button transparent small-round">
        <span>À propos</span>
      </a>
    </nav>

    <div class="app-body">
      <aside class="sidebar">
        <fieldset>
          <legend>Importer</legend>
          <div class="center-align">
            <button>
              <i>attach_file</i>
              <span>Fichier .mei</span>
            </button>
            <input type="file" accept=".mei" @change="onFileChange" />
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
            <button class="responsive">
              <i>image</i>
              <span>Format SVG</span>
            </button>
          </div>

          <div class="field">
            <button class="responsive">
              <i>picture_as_pdf</i>
              <span>Format PDF</span>
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
import RangeSlider from '@/components/RangeSlider.vue'
import ScoreViewer from '@/components/ScoreViewer.vue'

const scoreViewerRef = ref(null)
const selectedFormat = ref('adjusted')

function onFileChange(event) {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    scoreViewerRef.value.renderMei(e.target.result)
  }
  reader.readAsText(file)
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
