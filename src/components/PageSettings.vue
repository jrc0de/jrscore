<template>
  <fieldset>
    <legend>Options</legend>

    <label class="checkbox">
      <input type="checkbox" v-model="options.showMeasureNumbers" />
      <span>Numéros de mesure</span>
    </label>

    <label class="checkbox">
      <input type="checkbox" v-model="options.useEncodedBreaks" />
      <span>Sauts fixes</span>
    </label>
  </fieldset>

  <fieldset>
    <legend>Affichage</legend>
    <RangeSlider title="Zoom" :min="10" :max="120" :step="5" v-model:value="options.scale" />
  </fieldset>

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

    <RangeSlider
      title="Espacement des portées"
      :min="0"
      :max="20"
      :step="1"
      v-model:value="options.staffSpacing"
    />

    <RangeSlider
      title="Espacement linéaire"
      :min="0"
      :max="1"
      :step="0.05"
      v-model:value="options.spacingLinear"
    />

    <RangeSlider
      title="Espacement non linéaire"
      :min="0"
      :max="1"
      :step="0.05"
      v-model:value="options.spacingNonLinear"
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
