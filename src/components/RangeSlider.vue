<template>
  <div class="slider-component">
    <label :id="labelId">{{ title }}</label>

    <div class="slider small">
      <input
        type="range"
        v-model.number="value"
        :min="min"
        :max="max"
        :step="step"
        :aria-labelledby="labelId"
      />
      <span></span>
      <span class="tooltip"></span>
    </div>
  </div>
</template>

<script setup lang="js">
import { useId } from 'vue'

const value = defineModel('value', { type: Number, default: 5 })

defineProps({
  min: { type: Number, default: 0 },
  max: { type: Number, default: 10 },
  step: { type: Number, default: 0.01 },
  title: { type: String, required: true },
})

const labelId = useId()
</script>
