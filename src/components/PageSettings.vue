<template>
  <fieldset>
    <legend>Réglages</legend>

    <RangeSlider
      title="Marge haut"
      :min="0"
      :max="300"
      :step="10"
      v-model:value="options.pageMarginTop"
    />

    <RangeSlider
      title="Marge bas"
      :min="0"
      :max="300"
      :step="10"
      v-model:value="options.pageMarginBottom"
    />

    <RangeSlider
      title="Marge gauche"
      :min="0"
      :max="300"
      :step="10"
      v-model:value="options.pageMarginLeft"
    />

    <RangeSlider
      title="Marge droite"
      :min="0"
      :max="300"
      :step="10"
      v-model:value="options.pageMarginRight"
    />

    <RangeSlider title="Échelle" :min="20" :max="80" :step="1" v-model:value="options.scale" />

    <RangeSlider
      title="Espacement des portées"
      :min="0"
      :max="20"
      :step="1"
      v-model:value="options.staffSpacing"
    />
  </fieldset>
</template>

<script setup>
import RangeSlider from '@/components/RangeSlider.vue'

const options = defineModel({
  type: Object,
  required: true,
})
</script>
