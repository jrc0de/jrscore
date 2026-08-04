<template>
  <div ref="notationRef" class="notation" :class="{ 'has-score': hasScore }"></div>
</template>

<script setup lang="js">
import { ref, onMounted } from 'vue'
import createVerovioModule from 'verovio/wasm'
import { VerovioToolkit } from 'verovio/esm'

const notationRef = ref(null)
const hasScore = ref(false)
let toolkit = null

const props = defineProps({
  meiContent: { type: String, default: null },
})

onMounted(async () => {
  const VerovioModule = await createVerovioModule()
  toolkit = new VerovioToolkit(VerovioModule)
  toolkit.setOptions({ footer: 'none' })
  if (props.meiContent) renderMei(props.meiContent)
})

function renderMei(meiString) {
  toolkit.loadData(meiString)
  const svg = toolkit.renderToSVG(1)
  notationRef.value.innerHTML = svg
  hasScore.value = true
}

defineExpose({ renderMei })
</script>

<style scoped>
.notation.has-score {
  background: white;
  padding: 1rem;
  border-radius: 8px;
}

.notation :deep(svg) {
  color: black;
}

.notation :deep(svg *) {
  fill: black !important;
  stroke: black !important;
}
</style>
