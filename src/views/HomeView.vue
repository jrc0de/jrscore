<template>
  <h1>JRScore</h1>

  <div><input type="file" id="monFichier" name="monFichier" accept=".mei" /></div>

  <RangeSlider />
</template>

<script setup lang="js">
import RangeSlider from '@/components/RangeSlider.vue'
</script>
