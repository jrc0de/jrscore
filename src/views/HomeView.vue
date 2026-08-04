<template>
  <main>
    <div class="grid">
      <div class="s12">
        <nav class="primary">
          <h1>JRScore</h1>
          <div class="max"></div>
          <a href="/about" class="button transparent small-round">
            <span>À propos</span>
          </a>
        </nav>
      </div>
      <div class="s2">
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
      </div>
      <div class="s10">
        <ScoreViewer ref="scoreViewerRef" />
      </div>
    </div>
  </main>
</template>

<script setup lang="js">
import { ref } from 'vue'
import RangeSlider from '@/components/RangeSlider.vue'
import ScoreViewer from '@/components/ScoreViewer.vue'

const scoreViewerRef = ref(null)

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
