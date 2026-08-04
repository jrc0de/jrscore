<template>
  <div ref="notationRef" class="notation"></div>
</template>

<script setup lang="js">
import { ref, onMounted } from 'vue'
import createVerovioModule from 'verovio/wasm'
import { VerovioToolkit } from 'verovio/esm'

const notationRef = ref(null)
let toolkit = null

const props = defineProps({
  meiContent: { type: String, default: null },
})

onMounted(async () => {
  const VerovioModule = await createVerovioModule()
  toolkit = new VerovioToolkit(VerovioModule)
  if (props.meiContent) renderMei(props.meiContent)
})

function renderMei(meiString) {
  toolkit.loadData(meiString)
  const svg = toolkit.renderToSVG(1)
  notationRef.value.innerHTML = svg
}

defineExpose({ renderMei })
</script>
