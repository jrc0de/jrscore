<template>
  <div class="slider-component">
    <input type="range" v-model.number="value" :min :max :step />
    <span>{{ value }}</span>
  </div>
</template>

<script setup lang="js">
const value = defineModel('value', { type: Number, default: 5 })

defineProps({
  min: { type: Number, default: 0 },
  max: { type: Number, default: 10 },
  step: { type: Number, default: 1 },
})
</script>

<style scoped lang="css">
.slider-component {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px;
  margin: 10px;
  background-color: darkgrey;
}

input {
  width: 100%;
}
</style>
