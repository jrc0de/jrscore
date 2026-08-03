<template>
  <div class="page">
    <header class="banner">
      <h1>JRScore</h1>
      <a href="#" class="about-link">À propos</a>
    </header>

    <div class="layout">
      <aside class="controls">
        <fieldset class="import-fieldset">
          <legend>Importer</legend>
          <label for="monFichier" class="file-box" title="Importer un fichier MEI (.mei)">
            <svg
              class="icon"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <path
                d="M3 7a2 2 0 0 1 2-2h4l2 2h8a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V7z"
              />
            </svg>
          </label>
          <input
            type="file"
            id="monFichier"
            name="monFichier"
            accept=".mei"
            class="visually-hidden"
          />
        </fieldset>

        <fieldset>
          <legend>Réglages</legend>
          <RangeSlider title="toto" />
          <RangeSlider title="tata" />
          <RangeSlider title="titi" />
          <RangeSlider title="tictac" min="0" max="1" step="0.01" value="0.5" />
        </fieldset>

        <fieldset>
          <legend>Exporter</legend>
          <h4>SVG</h4>
          <h4>PDF</h4>
        </fieldset>
      </aside>

      <main class="content"></main>
    </div>
  </div>
</template>

<script setup lang="js">
import RangeSlider from '@/components/RangeSlider.vue'
</script>

<style lang="css" scoped>
.page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  color: #e5e5e5;
  background: #111;
}

.banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 24px;
  border-bottom: 1px solid #555;
}

.banner h1 {
  margin: 0;
  line-height: 1;
}

.about-link {
  color: #e5e5e5;
  text-decoration: none;
  font-size: 0.9rem;
}

.about-link:hover {
  color: #35a77c;
}

.layout {
  display: flex;
  flex: 1;
  min-height: 0;
}

.controls {
  width: 220px;
  flex-shrink: 0;
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.content {
  flex: 1;
  padding: 10px;
}

fieldset {
  border: 1px solid #555;
  border-radius: 6px;
  color: #e5e5e5;
}

.import-fieldset {
  display: flex;
  justify-content: center;
}

.file-box {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 6px;
  cursor: pointer;
}

.icon {
  width: 20px;
  height: 20px;
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}

h1 {
  color: #35a77c;
}
</style>
