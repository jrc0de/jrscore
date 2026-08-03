<template>
  <div class="slider-component">
    <label :for="inputId">{{ title }}</label>
    <input :id="inputId" type="range" v-model.number="value" :min :max :step />
    <span>{{ value }}</span>
  </div>
</template>

<script setup lang="js">
import { useId } from 'vue'

const value = defineModel('value', { type: Number, default: 5 })

defineProps({
  min: { type: Number, default: 0 },
  max: { type: Number, default: 10 },
  step: { type: Number, default: 1 },
  title: { type: String, required: true },
})

const inputId = useId()
</script>

<style scoped lang="css">
.slider-component {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px;
  margin: 10px;
}

input[type='range'] {
  accent-color: #35a77c;
  width: 100%;
}
</style>
