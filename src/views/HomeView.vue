<template>
  <main class="responsive">
    <div class="grid">
      <div class="s12 center-align">
        <h1 class="primary">JRScore</h1>
      </div>
      <div class="s2">
        <div class="large-space center-align">
          <button>
            <i>attach_file</i>
            <span>Fichier .mei</span>
          </button>
          <input type="file" accept=".mei" @change="onFileChange" />
        </div>
        <RangeSlider title="toto" />
        <RangeSlider title="toto" />
        <RangeSlider title="toto" />
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
